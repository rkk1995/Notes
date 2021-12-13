# SQL vs NoSQL

## Reasons for SQL

- Structured/Relational data
- ACID properties
- Require multiple indexes
- Need for complex dynamic queries + analytics
- Writes scaled by sharding / Reads scaled by replication
- Automation - backup, restore, failover, replication (semi-sync, async, sync)
- MySQL auto-incrementing columns for primary keys (unique IDs)
  - but MySQL can’t guarantee uniqueness across physical and logical databases.

### Facebook

There are many reasons why we use MySQL at Facebook. MySQL is amenable to automation, making it easy for a small team to manage thousands of MySQL servers while providing high-quality service. The automation includes backup, restore, failover, schema changes, and replacing failed hardware. MySQL also has flexible and extensible replication features, including asynchronous replication and lossless semi-synchronous replication, which makes it possible to do failover without losing data.

Semisynchronous replication falls between asynchronous and fully synchronous replication. The source waits until at least one replica has received and logged the events (the required number of replicas is configurable), and then commits the transaction. The source does not wait for all replicas to acknowledge receipt, and it requires only an acknowledgement from the replicas, not that the events have been fully executed and committed on the replica side. Semisynchronous replication therefore guarantees that if the source crashes, all the transactions that it has committed have been transmitted to at least one replica.

### Facebook move to MyRocks on MySQL

In the past, Facebook used InnoDB, a B+Tree based storage engine as the backend. The challenge was to find an index structure using less space and write amplification [1]. LSM-tree [2] has the potential to greatly improve these two bottlenecks. https://vldb.org/pvldb/vol13/p3217-matsunobu.pdf

#### LSM vs B-Tree

LSM-trees are typically faster for writes, whereas B-trees are thought to be faster for reads. Reads are typically slower on LSM-tress as they have to check several different data structures and SSTables at different stages of compaction.

The persistent data structure in InnoDB is the B+Tree on the primary key. In a few words, it means that the data is physically stored as B+Tree. It's well-known when we face a very high volume of inserts or random updates on a B+Tree (a heavy-write workload; for example, an event/tracking recording system), then there might be a trend to system degradation; because there is an overhead on the index maintenance while inserting rows in order to keep the tree as balanced as possible.

Log-structured merge tree (or LSM-tree) is a data structure with performance characteristics very attractive on high-insertion scenarios. It's based on the principle that the most efficient operation on persistent storage (SSD, hard disks) is sequential access, so the strategy to improve the performance on write-heavy workloads is to append data similar to traditional logs, thus eliminating any type of slow random access.

[B Tree vs LSM](../../../Notes/books/DDIA/chapter3.md#b-trees-and-lsm-trees)
## Reasons for NoSQL

- Common types : Key-Value, Document, Wide-Column, Graph
- Dynamic schemas /Non-relational data
- No need for complex joins
- Easily scaled for writes / reads by simply adding more servers. Consistent hashing

## Example

If you need to store a list, you can store it in both k/v and mysql. In k/v, you can have a key and the value is the list. It's great if the list doesn't change often and you often have to read the whole list. If you store as multiple rows in mysql, getting the whole list is more expensive but getting 1 row is easier, adding a row also easier, etc. There are many things to discuss here.

