
Hazelcast's durable executor service is a data structure which is able to store an execution task both on the executing Hazelcast member and its backup member(s), if configured. By this way, you do not lose any tasks if a member goes down or any results if the submitter (member or client) goes down while executing the task. When using the durable executor service you can either submit or execute a task randomly or on the owner of a provided key. Note that in [executor service](/00_Executor_Service), you can submit or execute tasks to/on the selected member(s).

Processing of the tasks when using durable executor service involves two invocations:

1. Sending the task to primary Hazelcast member (primary partition) and to its backups, if configured, and executing the task.
2. Retrieving the result of the task.

As you may already know, Hazelcast's executor service returns a `future` representing the task to the user. With the above two-invocations approach, it is guaranteed that the task is executed before the `future` returns and you can track the response of a submitted task with a unique ID. Hazelcast stores the task on both primary and backup members, and starts the execution also. 


With the first invocation, a [Ringbuffer](/06_Distributed_Data_Structures/06_Ringbuffer) stores the task and a generated sequence for the task is returned to the caller as a result. In addition to the storing, the task is executed on the local execution service for the primary member. By this way, the task is now resilient to member failures and you are able to track the task with its ID.

After the first invocation has completed and the sequence of task is returned, second invocation starts to retrieve the result of task with that sequence. This retrieval waits in the waiting operations queue until notified, or it runs immediately if the result is already available.

When task execution is completed, Ringbuffer replaces the task with the result for the given task sequence. This replacement notifies the waiting operations queue.

### Configuring Durable Executor Service

This section presents example configurations for durable executor service along with the descriptions of its configuration elements and attributes.

**Declarative:**

```xml
<durable-executor-service name="myDurableExecSvc">
	<pool-size>8</pool-size>
	<durability>1</durability>
	<capacity>1</capacity>
</durable-executor-service>
```

**Programmatic:**

```java
Config config = new Config();
config.getDurableExecutorConfig( "myDurableExecSvc" ).
      .setPoolSize ( 8 )
      .setDurability( 1 )
      .setCapacity( 1 );

HazelcastInstance hazelcast = Hazelcast.newHazelcastInstance();
DurableExecutorService durableExecSvc = hazelcast.getDurableExecutorService("myDurableExecSvc");
```
 
Following are the descriptions of each configuration element and attribute:

* `name`: Name of the executor task.
* `pool-size`: Number of executor threads per member for the executor.
* `durability`: Count of the backup members on which the execution task will be stored. Its default value is 1.
* `capacity`: Executor's task queue capacity; the number of tasks this queue can hold.

### Split-Brain Protection for DurableExecutorService

DurableExecutorService can be configured to check for a minimum number of available members before applying queue operations (see [Split-Brain Protection](/2600_Network_Partitioning/100_Split-Brain_Protection.md)). This is a check to avoid performing successful queue operations on all parts of a cluster during a network partition.

Following is a list of methods that now support Split-Brain Protection checks. The list is grouped by quorum type.

- WRITE, READ_WRITE:
    - `disposeResult`
    - `execute`
    - `executeOnKeyOwner`
    - `retrieveAndDisposeResult`
    - `shutdown`
    - `shutdownNow`
    - `submit`
    - `submitToKeyOwner`
- READ, READ_WRITE:
    - `retrieveResult`