
### Abstract Entry Processor

You can use the `AbstractEntryProcessor` class when the same processing will be performed both on the primary and backup map entries, i.e., the same logic applies to them. If you use entry processor, you need to apply the same logic to the backup entries separately. The `AbstractEntryProcessor` class makes this primary/backup processing easier.

You can use the [`AbstractEntryProcessor` class](http://docs.hazelcast.org/docs/latest/javadoc/com/hazelcast/map/AbstractEntryProcessor.html) to create your own abstract entry processor. The method `getBackupProcessor` in this class returns an `EntryBackupProcessor` instance. This means the same processing will be applied to both the primary and backup entries. If you want to apply the processing only upon the primary entries, make the `getBackupProcessor` method return null. 

![image](images/NoteSmall.jpg) ***NOTE***: *Beware of the null issue described above. Due to a yet unsent backup from a previous operation, an `EntryBackupProcessor` may temporarily receive `null` from `Map.Entry.getValue()` even though the value actually exists in the map. If you decide to use `AbstractEntryProcessor`, make sure your code logic is not sensitive to null values, or you may encounter `NullPointerException` during runtime.*
