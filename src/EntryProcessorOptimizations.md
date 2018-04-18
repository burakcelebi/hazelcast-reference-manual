
### Entry Processor Performance Optimizations

By default the entry processor executes on a partition thread. A partition thread is responsible for handling
one or more partitions. The design of entry processor assumes users have fast user code execution of the `process()` method.
In the pathological case where the code that is very heavy and executes in multi-milliseconds, this may create a bottleneck.

We have a slow user code detector which can be used to log a warning controlled by the following system properties:

- `hazelcast.slow.operation.detector.enabled` (default: true)
- `hazelcast.slow.operation.detector.threshold.millis` (default: 10000)

The defaults catch extremely slow operations but you should set this much lower, say to 1ms, at development time to catch entry processors that could be problematic in production. These are good candidates for our optimizations.

We have two optimizations:

- `Offloadable` which moves execution off the partition thread to an executor thread
- `ReadOnly` which means we can avoid taking a lock on the key

These are enabled very simply by implementing these interfaces in your `EntryProcessor`.

As of 3.9, these optimizations apply to the following IMap methods only:

- `executeOnKey(Object, EntryProcessor)`
- `submitToKey(Object, EntryProcessor)`
- `submitToKey(Object, EntryProcessor, ExecutionCallback)`

#### Offloadable Entry Processor

If an entry processor implements the `Offloadable` interface, the `process()` method will be executed in the executor
specified by the `Offloadable`'s `getExecutorName()` method.

Offloading will unblock the partition thread allowing the user to profit from much higher throughput.
The key will be locked for the time span of the processing in order to not generate a write conflict.

In this case the threading looks as follows:

1. partition thread (fetch entry & lock key)
2. execution thread (process(entry) method)
3. partition thread (set new value & unlock key, or just unlock key if the entry has not been modified)


The method `getExecutorName()` method may also return two constants defined in the [`Offloadable` interface](http://docs.hazelcast.org/docs/latest/javadoc/com/hazelcast/core/Offloadable.html):

- NO_OFFLOADING: Processing will not be offloaded if the method `getExecutorName()` returns this constant; it will be executed as if it does not implement the `Offloadable` interface.
- OFFLOADABLE_EXECUTOR: Processing will be offloaded to the default `ExecutionService.OFFLOADABLE_EXECUTOR`.

Note that if the method `getExecutorName()` cannot find an executor whose name matches the one called by this method, then the default executor service is used. Here is the configuration for the "default" executor:

```xml
<executor-service name="default">
    <pool-size>16</pool-size>
    <queue-capacity>0</queue-capacity>
</executor-service>
```

An example of an Offloadable called "OffloadedInventoryEntryProcessor" would be as follows:

```xml
<executor-service name="OffloadedInventoryEntryProcessor”>
    <pool-size>30</pool-size>
    <queue-capacity>0</queue-capacity>
</executor-service>
```

Remember to set the `pool-size` (count of executor threads per member) according to your execution needs. Please refer to the [Configuring Executor Service section](#configuring-executor-service) for the configuration details.


#### ReadOnly Entry Processor

By default, an entry processor will not run if the key is locked.
It will wait until the key has been unlocked (it applies to the `executeOnKey`, `submitToKey` methods, that were mentioned before).

If the entry processor implements the `ReadOnly` interface without implementing the `Offloadable` interface, the processing will not
be offloaded to an external executor. However, the entry processor will not observe if the key of the processed entry is
locked, nor will try to acquire the lock since the entry processor will not do any modifications.

If the entry processor implements `ReadOnly` and modifies the entry, an `UnsupportedOperationException` will be thrown.


#### ReadOnly and Offloadable Entry Processor

If the entry processor implements both `ReadOnly` and `Offloadable` interfaces, we will observe the combination of both
optimizations described above.

The `process()` method will be executed in the executor specified by the `Offloadable`'s `getExecutorName()` method.
Also, the entry processor will not observe if the key of the processed entry is locked, nor will try to acquire the
lock since the entry processor will not do any modifications.

In this case the threading looks as follows:

1. partition thread (fetch entry)
2. execution thread (process(entry))

In this case the `EntryProcessor.getBackupProcessor()` has to return null; otherwise an `IllegalArgumentException`
exception is thrown.

If the entry processor implements `ReadOnly` and modifies the entry, an `UnsupportedOperationException` will be thrown.

Putting it all together:

```
public class OffloadableReadOnlyEntryProcessor implements EntryProcessor<String, Employee>, 
        Offloadable, ReadOnly {

    @Override
    public Object process(Map.Entry<String, Employee> entry) {
        // heavy logic
        return null;
    }

    @Override
    public EntryBackupProcessor<String, Employee> getBackupProcessor() {
        // ReadOnly EntryProcessor has to return null, since it's just a read-only operation that will not be
        // executed on the backup
        return null;
    }

    @Override
    public String getExecutorName() {
        return OFFLOADABLE_EXECUTOR;
    }
}
```
