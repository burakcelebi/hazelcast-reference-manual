
<font color="##153F75">**Hazelcast IMDG Enterprise HD**</font>
<br></br>

If you have <font color="##153F75">**Hazelcast IMDG Enterprise HD**</font>, your Hazelcast Java client's Near Cache can benefit from the High-Density Memory Store. 

Let's recall the Java client's Near Cache configuration (please see the [Configuring Client Near Cache section](1600_Hazelcast_Clients/100_Java_Client/02_Configuration/50_Overview.md)) **without** High-Density Memory Store:

```xml
</hazelcast-client>
    ...
    ...
    <near-cache name="MENU">
        <max-size>2000</max-size>
        <time-to-live-seconds>0</time-to-live-seconds>
        <max-idle-seconds>0</max-idle-seconds>
        <eviction-policy>LFU</eviction-policy>
        <invalidate-on-change>true</invalidate-on-change>
        <in-memory-format>OBJECT</in-memory-format>
    </near-cache>
    ...
</hazelcast-client>
```

You can configure this Near Cache to use Hazelcast's High-Density Memory Store by setting the in-memory format to NATIVE. Please see the following configuration example:

```xml
</hazelcast-client>
    ...
    ...
    <near-cache>
       ...
       <time-to-live-seconds>0</time-to-live-seconds>
       <max-idle-seconds>0</max-idle-seconds>
       <invalidate-on-change>true</invalidate-on-change>
       <in-memory-format>NATIVE</in-memory-format>
       <eviction size="1000" max-size-policy="ENTRY_COUNT" eviction-policy="LFU"/>
       ...
    </near-cache>
</hazelcast-client>
``` 

Please notice that when the in-memory format is NATIVE, i.e. High-Density Memory Store is enabled, the configuration element `<eviction>` is used to specify the eviction behavior of your client's Near Cache. In this case, the elements `<max-size>` and `<eviction-policy>` used in the configuration of a Near Cache without High-Density Memory Store do not have any impact. 

The element `<eviction>` has the following attributes:

- `size`: Maximum size (entry count) of the Near Cache.
- `max-size-policy`: Maximum size policy for eviction of the Near Cache. Available values are as follows:
	* ENTRY_COUNT: Maximum entry count per member.
	* USED_NATIVE_MEMORY_SIZE: Maximum used native memory size in megabytes.
	* USED_NATIVE_MEMORY_PERCENTAGE: Maximum used native memory percentage.
	* FREE_NATIVE_MEMORY_SIZE: Minimum free native memory size to trigger cleanup.
	* FREE_NATIVE_MEMORY_PERCENTAGE: Minimum free native memory percentage to trigger cleanup.
- `eviction-policy`: Eviction policy configuration. Its default values is NONE. Available values are as follows:
	- NONE: No items will be evicted and the property max-size will be ignored. You still can combine it with time-to-live-seconds.
	- LRU: 	Least Recently Used.
	- LFU: 	Least Frequently Used.

Keep in mind that you should have already enabled the High-Density Memory Store usage for your client, using the `<native-memory>` element in the client's configuration.

Please see the [High-Density Memory Store section](/13_Storage/00_High-Density_Memory_Store) for more information on Hazelcast's High-Density Memory Store feature.







