
Starting with Hazelcast 3.6, Hazelcast introduces cluster and member states in addition to the default `ACTIVE` state. This section explains these states of Hazelcast clusters and members which you can use to allow or restrict the designated cluster/member operations.

#### Cluster States

By changing the state of your cluster, you can allow/restrict several cluster operations or change the behavior of those operations. You can use the methods `changeClusterState()` and `shutdown()` which are in the <a href="https://github.com/hazelcast/hazelcast/blob/master/hazelcast/src/main/java/com/hazelcast/core/Cluster.java" target="_blank">Cluster interface</a> to change your cluster's state.

 Hazelcast clusters have the following states:

- **`ACTIVE`**: This is the default cluster state. Cluster continues to operate without restrictions.
<br></br>
- **`NO_MIGRATION`**:
    - In this state, migrations (partition rebalancing) and backup replications are not allowed. In other words, there will be no data movement between Hazelcast members. However, in case of member failures, backup replicas can be still promoted to the primaries to maintain availability, and migration listeners can be notified for these promotion migrations. Please note that promoting a backup replica to the primary replica is a local operation and does not transfer partition data between Hazelcast members.
    - The cluster accepts new members.
    - All other operations are allowed.
    - You cannot change the state of a cluster to `NO_MIGRATION` when migration/replication tasks are being performed.
    - When you want to add multiple new members to the cluster, you can first change the cluster state to `NO_MIGRATION`, then start the new members. Once all of them join to the cluster, you can change the cluster state back to `ACTIVE`. Then, the cluster will rebalance partition replica distribution at once.
<br></br>
- **`FROZEN`**:
	- In this state, the partition table is frozen and partition assignments are not performed.
	- The cluster does not accept new members.
	- If a member leaves, it can join back. Its partition assignments (both primary and backup replicas) remain the same until either it joins back or the cluster state is changed to `ACTIVE`. When it joins back to the cluster, it will own all previous partition assignments as it was. On the other hand, when the cluster state changes to `ACTIVE`, re-partitioning starts and unassigned partition replicas are assigned to the active members.
	- All other operations in the cluster, except migration, continue without restrictions.
	- You cannot change the state of a cluster to `FROZEN` when migration/replication tasks are being performed.
	- You can make use of `FROZEN` state along with the [Hot Restart Persistence](/1500_Storage/300_Hot_Restart_Persistence/100_Hot_Restart_Types.md) feature. You can change the cluster state to `FROZEN`, then restart some of your members using the Hot Restart feature. The data on the restarting members will not be accessible but you will be able to access to the data that is stored in other members. Basically, `FROZEN` cluster state allows you do perform maintenance on your members with degrading availability partially.
<br></br>
- **`PASSIVE`**:
	- In this state, the partition table is frozen and partition assignments are not performed. 
	- The cluster does not accept new members.
	- If a member leaves while the cluster is in this state, the member will be removed from the partition table if cluster state moves back to `ACTIVE`. 
	- This state rejects ALL operations immediately EXCEPT the read-only operations like `map.get()` and `cache.get()`, replication and cluster heartbeat tasks. 
	- You cannot change the state of a cluster to `PASSIVE` when migration/replication tasks are being performed.
	- You can make use of `PASSIVE` state along with the [Hot Restart Persistence](/1500_Storage/300_Hot_Restart_Persistence/100_Hot_Restart_Types.md) feature. Please see <a href="https://github.com/hazelcast/hazelcast/blob/master/hazelcast/src/main/java/com/hazelcast/core/Cluster.java#L245" target="_blank">Cluster Shutdown API</a> for more info.
<br></br>
- **`IN_TRANSITION`**: 
	- This state shows that the state of the cluster is in transition. 
	- You cannot set your cluster's state as `IN_TRANSITION` explicitly. 
	- It is a temporary and intermediate state. 
	- During this state, your cluster does not accept new members and migration/replication tasks are paused.



![image](../../images/NoteSmall.jpg) ***NOTE:*** *All in-cluster methods are fail-fast, i.e. when a method fails in the cluster, it throws an exception immediately (it will not be retried).*


The following snippet is from the `Cluster` interface showing the new methods used to manage your cluster's states.


```java
public interface Cluster {
...
...
    ClusterState getClusterState();
    void changeClusterState(ClusterState newState);
    void changeClusterState(ClusterState newState, TransactionOptions transactionOptions);
    void shutdown();
    void shutdown(TransactionOptions transactionOptions);
```

Please refer to the <a href="https://github.com/hazelcast/hazelcast/blob/master/hazelcast/src/main/java/com/hazelcast/core/Cluster.java" target="_blank">Cluster interface</a> for information on these methods.

#### Cluster Member States

Hazelcast cluster members have the following states:

- **`ACTIVE`**: This is the initial member state. The member can execute and process all operations. When the state of the cluster is `ACTIVE` or `FROZEN`, the members are in the `ACTIVE` state. 
<br></br>
- **`PASSIVE`**: In this state, member rejects all operations EXCEPT the read-only ones, replication and migration operations, heartbeat operations, and the join operations as explained in the Cluster States section above. A member can go into this state when either of the following happens:
	1. Until the member's shutdown process is completed after the method `Node.shutdown(boolean)` is called. Note that, when the shutdown process is completed, member's state changes to `SHUT_DOWN`. 
	2. Cluster's state is changed to `PASSIVE` using the method `changeClusterState()`. 
<br></br>
- **`SHUT_DOWN`**: A member goes into this state when the member's shutdown process is completed. The member in this state rejects all operations and invocations. A member in this state cannot be restarted.
<br></br>
