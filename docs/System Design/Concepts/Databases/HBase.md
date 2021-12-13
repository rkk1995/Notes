# HBase

Facebook created Cassandra and it was purpose built for an inbox type application, but they found Cassandra's eventual consistency model wasn't a good match for their new real-time Messages product. Facebook also has an extensive [MySQL infrastructure](http://highscalability.com/blog/2010/11/4/facebook-at-13-million-queries-per-second-recommends-minimiz.html), but they found performance suffered as data set and indexes grew larger. And they could have built their own, but they chose HBase.

HBase is a *[scaleout table store supporting very high rates of row-level updates over massive amounts of data](http://www.cloudera.com/blog/2010/06/integrating-hive-and-hbase/)*. Exactly what is needed for a Messaging system. HBase is also a column based key-value store built on the [BigTable](http://en.wikipedia.org/wiki/HBase) model. It's good at fetching rows by key or scanning ranges of rows and filtering. Also what is needed for a Messaging system. Complex queries are not supported however.

Facebook chose HBase because they **monitored their usage and figured out what the really needed**. What they needed was a system that could handle two types of data patterns:

1. A short set of temporal data that tends to be volatile
2. An ever-growing set of data that rarely gets accessed

- Has a simpler consistency model than Cassandra.
- Very good scalability and performance for their data patterns.
- Most feature rich for their requirements: auto load balancing and failover, compression support, multiple shards per server, etc.
- HDFS, the filesystem used by HBase, supports replication, end-to-end checksums, and automatic rebalancing.

That's what HBase has achieved. Given how HBase covers a nice spot in the persistence spectrum--real-time, distributed, linearly scalable, robust, BigData, open-source, key-value, column-oriented--we should see it become even more popular, especially with its anointment by Facebook.

## Facebook move to MyRocks on MySQL

To help improve Messenger even more, we now have overhauled and modernized the storage service to make it faster, more efficient, more reliable, and easier to upgrade with new features. This evolution involved several major changes:

- We redesigned and simplified the data schema, created a new source-of-truth index from existing data, and made consistent invariants to ensure that all data is formatted correctly.
- We moved from HBase, an open source distributed key-value store based on HDFS, to [MyRocks](https://code.facebook.com/posts/190251048047090/myrocks-a-space-and-write-optimized-mysql-database/), Facebook's open source database project that integrates RocksDB as a MySQL storage engine.
- We moved from storing the database on spinning disks to flash on our new [Lightning Server SKU](https://code.facebook.com/posts/989638804458007/introducing-lightning-a-flexible-nvme-jbof/).
