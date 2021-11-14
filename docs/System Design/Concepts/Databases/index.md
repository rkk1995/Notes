# Databases

## Row vs Column Databases

- Row
  - Optimal for read/writes
  - OLTP (Transactions)
  - Compression isn't efficient
  - Aggregation isn't efficient
  - Efficient queries w/ multiple columns
- Column
  - Slower writes
  - OLAP
  - Compress greatly
  - Aggregation is great
  - Inefficient queries w/ multi-columns
  
## MySQL BinLog

The binary log is a set of log files that contain information about data modifications made to a MySQL server instance. The log is enabled by starting the server with the --log-bin option. Who will use this log? 
**PubSub system sending updates to subscribers**

- Cache invalidation in other regions
- ETL to data warehouse