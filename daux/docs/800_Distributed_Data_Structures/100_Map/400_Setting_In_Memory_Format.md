
IMap (and a few other Hazelcast data structures, such as ICache) has an `in-memory-format` configuration option. By default, Hazelcast stores data into memory in binary (serialized) format. Sometimes it can be efficient to store the entries in their object form, especially in cases of local processing, such as entry processor and queries.

To set how the data will be stored in memory, set `in-memory-format` in the configuration. You have the following format options:

- `BINARY` (default). The data (both the key and value) will be stored in serialized binary format. You can use this option if you mostly perform regular map operations, such as `put` and `get`.

- `OBJECT`. The data will be stored in deserialized form. This configuration is good for maps where entry processing and queries form the majority of all operations and the objects are complex, making the serialization cost comparatively high. By storing objects, entry processing will not contain the deserialization cost. Note that when you use `OBJECT` as the in-memory format, the key will still be stored in binary format, and the value will be stored in object format.
 
- `NATIVE`: (<font color="##153F75">**Hazelcast IMDG Enterprise HD**</font>) This format behaves the same as BINARY, however, instead of heap memory, key and value will be stored in the off-heap memory.


Regular operations like `get` rely on the object instance. When the `OBJECT` format is used and a `get` is performed, the map does not return the stored instance, but creates a clone. Therefore, this whole `get` operation first includes a serialization on the member owning the instance, and then a deserialization on the member calling the instance. When the `BINARY` format is used, only a deserialization is required; `BINARY` is faster.

Similarly, a `put` operation is faster when the `BINARY` format is used. If the format was `OBJECT`, the map would create a clone of the instance, and there would first be a serialization and then a deserialization. When BINARY is used, only a deserialization is needed.


![image](../../images/NoteSmall.jpg) ***NOTE:*** *If a value is stored in `OBJECT` format, a change on a returned value does not affect the stored instance. In this case, the returned instance is not the actual one but a clone. Therefore, changes made on an object after it is returned will not reflect on the actual stored data. Similarly, when a value is written to a map and the value is stored in `OBJECT` format, it will be a copy of the `put` value. Therefore, changes made on the object after it is stored will not reflect on the stored data.*

