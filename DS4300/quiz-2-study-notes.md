MTTF - Amount of time it takes for a device to fail
MTBF - Amount of time between failures, until repair is done.

### Indexing

Advantage: Faster read access
Disadvantage: Slower inserts/updates, storage overhead.

Indexing basically subdivides data so that it can be read more quickly.

### B-Trees

Keys are sorted, enabling efficient range queries
Organizes DB into fixed size blocks
Each child (block) is responsible for a range of keys
Can "sub branch": for example, to find 257 in 0-500 you may go - 0-500 &rarr; 200-299 &rarr; 240-259 etc.

If inserting into a full node, the smaller half and bigger half become subnodes, and the middle number is promoted as a key to the 2 nodes.

Pros: Consistent performance over a wide range of read/wirte workloads, range queries
Cons: Page fragmentation, WAL required, I/O overhead for blocks, requires locking mechanism for concurrency support

##### Example

Max size: 2, starting tree:
	[3] -> [1 2][4]
Now, insert 0 and it becomes the following:
[1 3]->[0][2][4]

### Write ahead logging

Write to log first before writing to disk

Allows program to determine if operation completed, adjust accordingly

### Hash Indexing

In  memory hash  map, maps to the byte location in a file where the data is stored.

### Heap Storage

Clustered index - Keys and values are both stored in the index

Non-Clustered index - Values are on disk in a Heap file. Keys give pointers to where to go on disk.

Covering index - Some columns are in the index, others are in the heap

### Secondary Indexes

Can have duplicates, unlike the primary index, which must be a unique ID. Can be useful if want to be reading on this value often.

### Compaction and Merging

Compaction - Compact multiple writes so that you only get latest update.
Merge - Merge together writes from multiple data segments, compact together.

### Riak Key-Value Store/Bitcask Storage Engine

Keys in memory, values on disk
High performance reads and writes
Use case - many operations across many different keys
All keys must fit in RAM

### Limitations of Hash indexes

* Must fit in RAM
* Can't do range queries

### LSM Trees

* Active segment is in-memory balanced tree called a **memtable**
* Periodically, data from memtable is written to disk in sorted order.
* Compaction and merging are efficient because the segments are sorted
* No longer need every key in ram, go to the segment on disk.


Pros - Faster writes, efficient storage, simple
Cons - Newer, slower reads, performance may depend on state of compaction.

### More Riak and Bitcask

Early KV Store

**Features**:
* Low latency per item read or written
* High throughput for handling incoming stream of random itmes
* Size(Datasets) >> Size(RAM)
* Crash Recovery
* Backup and Restore support
* Simple and maintainable

**How it was done**:
* Each DB *instance* stored in its own directory
* Each instance has an active log file, once it gets too big, it becomes immutable and a new log is created
* Appends are only allowed to the active log file
* Deletes use a special *tombstone value* indicating the item has been deleted
* KeyDir - On insert/update, key is updated with value location of the new value. Old data remains on disk but later is removed.
	* To get data you look up the key, which gives you the file, position of the value in the file, and the size of the value. Now simply read from file.

### R-Trees



