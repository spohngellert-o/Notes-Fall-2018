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

Use case - many operations acro1 many different keys

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
* Hashes (String &rarr; String)
* Bit arrays

### Redis at Scale

* As a cache, scaling up is easy. As a datastore, nodes must be in fixed places as you need key &rarr; node store.
* Sharding is popular for scaling.

### Redis Security

* Basically none
* User sends auth command with password, but password is stored in plaintext

### Redis vs Relational

* In Redis, there aren't schemas, so data retrieval is controled at application layer.
* It is very important to think up front about how data will be stored.
* Targeted at explicit lookups/reads

### Persistence Process

1. Client sends write command to DB
2. DB receives
3. DB calls sys call to write
4. OS transfers write buffer to disk controller (disk cache)
5. Disc controller writes

### Pools

Multiple Redis servers running on the same machine using different ports

Efficient memory usage, one cpu per server

### Replication

* Non blocking on slave side
* Slaves are read-only 

**Process**

1. Master starts buffering new commands and saving db in background
2. After saving, master transfers db to slave
3. Slave saves files to disk and loads into memory
4. Master sends all buffered commands to slave

### Persistence

RDB - redis database file, point in time snapshot. Good for backup and recovery, easy to use and compact. Limited to save points, not good if you want to minimize chance of data loss if Redis stops. Can be time consuming and make performance worse.

AOF (Append Only File) - Main persistence option. Every operation is logged, log can be piped to another instance and DB can be reconstructed. More durable, single file with no corruptions. Automatically rewrites in bg if it gets too big. Takes longer to load into memory on server restart, bigger file, can be slower than rdb.


### Levels of consistency

Strict - Changes to data are atomic
Sequential - Clients see changes in the order they were applied

Causal - All causally related changes are observed in the same order by all clients

Eventual - All updates will eventually prpogate through the system, and all replicas will be consistent

Weak - no guarantees, changes may appear out of order.

Read After Write - Users always see data that they submitted

Monotonic Reads - After users have seen data, they shouldn't see earlier data

Consistent Prefix Reads - Users see data in an order that makes sense

### Tunable consistency

If W + R > N, we get strict consistency where:
* R = number of servers read
* W = number of servers written to
* N = number of servers

If W + R <= N, we get eventual consistency

### Replication methods


Statement based - Send all statements to clients. Simple, but error prone due to functions with side effects like now()

WAL - Send a log to the replicas. Leader and all followers must implement the same storage engine, makes upgrades difficult.

Logical (row-based) Log - For RDBs, inserted rows, deleted rows, etc. Logical logs are decoupled from storage engine (any SQL will work)

Trigger based - Changes are logged to a separate table whenever a trigger fires in response to an insert, update etc. Can be application specific but error prone.

### Synchronous vs Async

Sync - Leader waits for response from follower before giving response to client

Async - Leader doesn't wait for confirmation 

These two can be mixed

### One leader based replication (master/slave)

* Write to leader only
* Leader sends replication stream to followers
* clients read from anywhere


### Multi Leader Replication

* Each datacenter has a leader
* Leaders communicate with each other
* Pro: Greater tolerance of intra-datacenter networking problems
* Con: Need to resolve conflicts. Ex - 2 devices edit the same thing connected to 2 data centers. How do we resolve if they do conflicting things?

### Conflict handling strategies

Avoidance - Require all writes for a particular user go through the sme datacenter.

Convergence - Allow conflict, but converge on a common outcome. For example, use write timestamps and use the most recent write.

### Leaderless replication

* No leader
* Writes to one or more replicas in any order
* Essentially use W + R < N model