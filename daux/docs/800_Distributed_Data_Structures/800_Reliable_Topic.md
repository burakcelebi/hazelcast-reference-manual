
The Reliable Topic data structure was introduced in Hazelcast 3.5. The Reliable Topic uses the same `ITopic` interface
as a regular topic. The main difference is that Reliable Topic is backed up by the Ringbuffer (also introduced with Hazelcast 
3.5) data structure. The following are the advantages of this approach:

* Events are not lost since the Ringbuffer is configured with one synchronous backup by default.
* Each Reliable `ITopic` gets its own Ringbuffer; if a topic has a very fast producer, it will not lead to problems at topics that run at a slower pace.
* Since the event system behind a regular `ITopic` is shared with other data structures, e.g., collection listeners, 
  you can run into isolation problems. This does not happen with the Reliable `ITopic`.

### Sample Reliable ITopic Code

```java
import com.hazelcast.core.Topic;
import com.hazelcast.core.Hazelcast;
import com.hazelcast.core.MessageListener;

public class Sample implements MessageListener<MyEvent> {

  public static void main( String[] args ) {
    Sample sample = new Sample();
    HazelcastInstance hazelcastInstance = Hazelcast.newHazelcastInstance();
    ITopic topic = hazelcastInstance.getReliableTopic( "default" );
    topic.addMessageListener( sample );
    topic.publish( new MyEvent() );
  }

  public void onMessage( Message<MyEvent> message ) {
    MyEvent myEvent = message.getMessageObject();
    System.out.println( "Message received = " + myEvent.toString() );
  }
}
```

You can configure the Reliable `ITopic` using its Ringbuffer. If a Reliable Topic has the name `Foo`, then you can configure this topic
by adding a `ReliableTopicConfig` for a Ringbuffer with the name `Foo`. By default, a Ringbuffer does not have any TTL (time to live) and
it has a limited capacity; you may want to change that configuration.

By default, the Reliable `ITopic` uses a shared thread pool. If you need better isolation, you can configure a custom executor on the 
`ReliableTopicConfig`. 

Because the reads on a Ringbuffer are not destructive, batching is easy to apply. `ITopic` uses read batching and reads
ten items at a time (if available) by default.

### Slow Consumers

The Reliable `ITopic` provides control and a way to deal with slow consumers. It is unwise to keep events for a slow consumer in memory 
indefinitely since you do not know when the slow consumer is going to catch up. You can control the size of the Ringbuffer by using its capacity. For the cases when a Ringbuffer runs out of its capacity, you can specify the following policies for the `TopicOverloadPolicy` configuration:

* `DISCARD_OLDEST`: Overwrite the oldest item, even if a TTL is set. In this case the fast producer supersedes a slow consumer.
* `DISCARD_NEWEST`: Discard the newest item.
* `BLOCK`: Wait until the items are expired in the Ringbuffer.
* `ERROR`: Immediately throw `TopicOverloadException` if there is no space in the Ringbuffer.

### Configuring Reliable Topic

The following are example Reliable Topic configurations.


**Declarative:**

```xml
<reliable-topic name="default">
    <statistics-enabled>true</statistics-enabled>
    <message-listeners>
        <message-listener>
        ...
        </message-listener>
    </message-listeners>
    <read-batch-size>10</read-batch-size>
    <topic-overload-policy>BLOCK</topic-overload-policy>
</reliable-topic>
```

**Programmatic:**

```java
Config config = new Config();
ReliableTopicConfig rtConfig = config.getReliableTopicConfig( "default" );
rtConfig.setTopicOverloadPolicy( TopicOverloadPolicy.BLOCK )
        .setReadBatchSize( 10 )
        .setStatisticsEnabled( true );
```

Reliable Topic configuration has the following elements.

- `statistics-enabled`: Enables or disables the statistics collection for the Reliable Topic. The default value is `true`.
- `message-listener`: Message listener class that listens to the messages when they are added or removed.
- `read-batch-size`: Minimum number of messages that Reliable Topic will try to read in batches. The default value is 10.
- `topic-overload-policy`: Policy to handle an overloaded topic. Available values are `DISCARD_OLDEST`, `DISCARD_NEWEST`, `BLOCK` and `ERROR`. The default value is `BLOCK. Please see the Slow Consumers section above for definitions of these policies.



