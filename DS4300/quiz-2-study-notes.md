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
	
	[3] &rarr; [1 2][4]

Now, insert 0 and it becomes the following:
	
	[1 3] &rarr; [0][2][4]

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

Tree data structure for indexing multi-dimensional data.

### Data warehousing

* Provides separate resource for data analytics
* Data can be manipulated without impacting users

Data Marts - Warehouse subdivided into small marts that can be used by the appropriate people.


### OLTP vs OLAP

**OLTP**

* Focus is fast querying speed, maintaining data integrity
* Primary user - enduser/customer
* Data is latest state

**OLAP**

* Focus is on getting large amounts of data/records
* Bulk write as well
* Primary user - internal anlayst
* Data is the history of events
* much larger in scale

### Columnar Storage

* Data is stored in column order on disk
* Good for data aggregates on columns (makes it faster)


## Section 2: KV Stores/Redis

KV stores are designed to be simple, fast, and scalable

#### Features

* Consistency - Consistency only guaranteed on a single key. Distributed stores are ventually consistent.
* Transactions - varies
* Querying - By key, and usually only by key.
* Data - Blob, text, json, xml, etc.
* Scaling - Shard by key

#### Use Cases

* Storing session info - Everything about session stored with **one** put, retrieved with get. Makes it fast.
* User Profiles, Preferences
* Shopping cart data
* Caching layer

### Redis

* In memory data structure store
* Can be used as DB, cache, message broker
* Super fassed as it is in-memory, unlike MongoDB
* Supports clustering via Master-Slave
* Doesn't handle complex data very well
* Has support for snapshotting

#### Master-Slave replication

* Performance critical and some data loss is acceptable
* Replicas do backups, Masters service requests

### Redis vs Memcached

* Memchached is another NoSQL kv store, stores in memory
* Memcached uses a volatile cache: data is not persisted
* Redis persists data in background

### Datatypes in Redis

#### Keys

Usually strings, but can be any binary

#### Values

* Strings
* Lists
* Sets
* Sorted Sets
* Hashes
* Bit arrays

### Redis at Scale

* As a cache, scaling up is easy. As a datastore, nodes must be in fixed places as you need key &rarr; node store.