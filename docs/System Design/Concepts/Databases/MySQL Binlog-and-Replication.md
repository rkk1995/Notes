# MySQL BinLog

The binary log is a set of log files that contain information about data modifications made to a MySQL server instance. The log is enabled by starting the server with the --log-bin option. 

## Facebook Example

We used traditional asynchronous MySQL replication for cross region MySQL replication. However, for in-region fault tolerance, we created a middleware called Binlog Server (Log Backup Unit) which can retrieve and serve the MySQL replication logs known as Binary Logs. Binlog Servers only retain a short period of recent transaction logs and do not maintain a full copy of the database. 

Each MySQL instance replicates its log to two Binlog Servers using MySQL Semi-Synchronous protocol. All three servers are spread across different failure domains within the region. This architecture made it possible to achieve both short (in-region) commit latency and one database copy per region. We use MySQL's Binary Logs not only for MySQL Replication, but also for notifying updates to external applications. We created a pub-sub service called Wormhole [6] for this. One of the use cases of Wormhole is invalidating the TAO cache in remote regions by reading the Binary Log of the region's MySQL instance.