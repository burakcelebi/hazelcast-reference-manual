
The following example code illustrates a distributed queue that connects a producer and consumer.

#### Putting Items on the Queue

Let's `put` one integer on the queue every second, 100 integers total.

```java
import com.hazelcast.core.Hazelcast;
import com.hazelcast.core.HazelcastInstance;
import com.hazelcast.core.IQueue;

public class ProducerMember {
  public static void main( String[] args ) throws Exception {
    HazelcastInstance hz = Hazelcast.newHazelcastInstance();
    IQueue<Integer> queue = hz.getQueue( "queue" );
    for ( int k = 1; k < 100; k++ ) {
      queue.put( k );
      System.out.println( "Producing: " + k );
      Thread.sleep(1000);
    }
    queue.put( -1 );
    System.out.println( "Producer Finished!" );
  }
}
``` 

`Producer` puts a **-1** on the queue to show that the `put`s are finished. 

#### Taking Items off the Queue

Now, let's create a `Consumer` class to `take` a message from this queue, as shown below.


```java
import com.hazelcast.core.Hazelcast;
import com.hazelcast.core.HazelcastInstance;
import com.hazelcast.core.IQueue;

public class ConsumerMember {
  public static void main( String[] args ) throws Exception {
    HazelcastInstance hz = Hazelcast.newHazelcastInstance();
    IQueue<Integer> queue = hz.getQueue( "queue" );
    while ( true ) {
      int item = queue.take();
      System.out.println( "Consumed: " + item );
      if ( item == -1 ) {
        queue.put( -1 );
        break;
      }
      Thread.sleep( 5000 );
    }
    System.out.println( "Consumer Finished!" );
  }
}
```

As seen in the above example code, `Consumer` waits five seconds before it consumes the next message. It stops once it receives **-1**. Also note that `Consumer` puts **-1** back on the queue before the loop is ended. 

When you first start `Producer` and then start `Consumer`, items produced on the queue will be consumed from the same queue.

#### Balancing the Queue Operations

From the above example code, you can see that an item is produced every second and consumed every five seconds. Therefore, the consumer keeps growing. To balance the produce/consume operation, let's start another consumer. This way, consumption is distributed to these two consumers, as seen in the sample outputs below. 

The second consumer is started. After a while, here is the first consumer output:

```plain
...
Consumed 13 
Consumed 15
Consumer 17
...
```

Here is the second consumer output:

```plain
...
Consumed 14 
Consumed 16
Consumer 18
...
```

In the case of a lot of producers and consumers for the queue, using a list of queues may solve the queue bottlenecks. In this case, be aware that the order of the messages sent to different queues is not guaranteed. Since in most cases strict ordering is not important, a list of queues is a good solution.

![image](../../images/NoteSmall.jpg) ***NOTE:*** *The items are taken from the queue in the same order they were put on the queue. However, if there is more than one consumer, this order is not guaranteed.*

#### ItemIDs When Offering Items

Hazelcast gives an `itemId` for each item you offer, which is an incrementing sequence identification for the queue items. You should consider the following to understand the `itemId` assignment behavior:

- When a Hazelcast member has a queue, and that queue is configured to have at least one backup, and that member is restarted, the `itemId` assignment resumes from the last known highest `itemId` before the restart; `itemId` assignment does not start from the beginning for the new items.
- When the whole cluster is restarted, the same behavior explained in the above consideration applies if your queue has a persistent data store (`QueueStore`). If the queue has `QueueStore`, the `itemId` for the new items are given, starting from the highest `itemId` found in the IDs returned by the method `loadAllKeys`. If the method `loadAllKeys` does not return anything, the `itemId`s will started from the beginning after a cluster restart.
- The above two considerations mean there will be no duplicated `itemId`s in the memory or in the persistent data store.
