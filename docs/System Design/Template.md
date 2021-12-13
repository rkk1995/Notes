# Template

## (1) Gather Requirements [1-2 min]

Ask questions to help define what parts of the system we will focus on.

1. Users/Customers
   1. Who will use? How will system be used?
   2. Do we need very low latency? (Usually serve from Memory then)
2. Usage patterns
   1. What data do we store? Does the data change alot?
   2. Real time or delayed data ok? Stale data okay?
   3. Read your own writes usually a must otherwise confusing
   4. Do we need to push notifications?
3. Scale (use answers for capacity estimation)
   1. How many users?
   2. How many data points?
4. Open source technologies ok? Cost of development?

## (2) Functional + Non Functional Requirements [5 min]

1. Functional requirements
   1. Actions the systems will make. Expand to make more extensible.
   2. countViewEvent -> countEvent -> processEvent -> processEvents
   3. How to prevent abuse? API Key for greater limits
2. Non-Functional requirements
   1. Availability vs Consistency (discuss)
   2. Scalability
   3. Performance (Low latency)
   4. Durable (if required)
   5. What features should we ensure (payment always processed, data not lost, etc)

## (3) Capacity Estimations [3 min]

1. Num users per day & Num of events per user = Num events per day
2. QPS = Num events per day / 1E5
3. R&W Ratio applied to QPS to figure out reads/writes per second.
4. Storage size per 1 day and 5 yr = Num events per day * 5Y
5. Bandwidth read / write = R/W per second * size of each
  
1. (Optional) Bandwidth estimate (#of bytes/sec system should handle for incoming and outgoing traffic)
2. (Optional) Cache estimate = 20% * Traffic
   1. 80-20 rule - 20% of objects are used 80% of the time.
   2. If we are using a cache, what is the kind of data we want to store in cache
   3. Redis. Do we need persistence?

## (4) High Level Design [5-10 min]

- "Hey I'm going to solve problem for small scale. 1 server is going to handle it. Then once I've fixed the solution I will work on scaling it up."
- Explain high level data flow for **each operation in functional requirements**.
- Stateful vs Stateless web tier: stateful has to deal with sticky session and other compleixty, stateless is more robust and scalable too.

## (5) Data Model [5 min]

Think about the nature of our data

1. How many records do we store? (Capacity Estimations)
2. Size of each object? (Capacity Estimations)
3. . Raw data or aggregated data?
4. Relationship between records?
5. Read vs Write heavy?

Defining the data model in the early part of the interview will clarify how data will flow between different system components. Later, it will guide for data partitioning and management. The candidate should identify various system entities, how they will interact with each other, and different aspects of data management like storage, transportation, encryption, etc.

Which database system should we use? Will Non Relational like [Cassandra](https://en.wikipedia.org/wiki/Apache_Cassandra) best fit our needs, or should we use a relational-like solution? What kind of block storage should we use to store photos and videos? SQL consider if we have lots of tables, joins, need for consistency and transactions.

- Relational is harder to scale since it offers more guarantees. Does not mean its not as scalable, but more work and may lose guarantees.
  - Availability, Consistency, Txn, Schema, Scalability, joins, ACID properties
  - Writes scaled by sharding / Reads scaled by replication
  - MySQL auto-incrementing columns for primary keys (unique IDs)
- Non Relational
  - Easily scaled for writes / reads by simply adding more servers and using Consistent hashing.

See [SQL or NoSQL](Concepts/Databases/SQL%20or%20NoSQL.md)

## (6) Deep Dive [15-20 min]

- Scaling individual components:
  - Use Non Functional Requirements to drive this.
  - Availability, Consistency and Scale story for each component
  - Consistency and availability patterns
  
### Database scaling

- Data access patterns. Do we need seperate hot/cold storage?
  - DB cleanup processor?
- Vertical vs Horizontal. Usually Vertical not as cost efficient, reliable, or scalable.
- When sharding in H, most important factor is sharding key.
  - Resharding, Celebrity, Join/Denormalization.
  - Consider Proximity Server. We store data in DB but create Quad Tree in memory.
    - Maybe we cannot fit entire Quad Tree on server. Shard so each quad tree contains 1/10th of original quad tree. How to shard?
    - If we shard on regionid, we may have hot regions. NY machines will have alot more queries/data than SA machines
      - Although not applicable to Quad Tree, one approach to deal with hot partitions is to include event time into the partition key. All events for one minute go to one host and over time the load is distributed.
    - If we shard on locationId, we have to fan out to all hosts containing each quad tree to aggregate results. Each quad tree will contain random elemsensts
    - **Tradeoff**
  - Consistent Hashing

- MySQL
  - Cluster proxy that routes traffic to correct shard. To know the shards, can use a configuration service which maintains a connection to all shards using ZooKeeper ephemeral nodes.
  - Shard proxy can sit infront of a database and cache queries, monitor health, and publish metrics.
  - Replication with async, semisync, sync
  - Complex, FB uses many optimizations and extensive cache layer. Also moved to LSM which has less write amplification w/ RocksDB.
- NoSQL
  - Utilizes gossip protocol instead of a cluster proxy w/ configuration service.
  - Uses consistent hashing to pick the node to write the data. Data is replicated to next 2-3 nodes in the ring.
  - Can increase read/write quorum.
- Mention utilizing multiple data centers per shard.

### Message Queues

- Asynchronous communication between web servers. Servse as a buffer and distributes requests.
- If consumer is unavailable, message will not be lost. Fire and forget.
- There are many benefits of batching: it increases throughput, it helps to save on cost, request compression is more effective. But there are drawbacks as well. It introduces some complexity both on the client and the server side. For example think of a scenario when partitioner service processes a batch request and several events from the batch fail, while other succeed. Should we re-send the whole batch?
  
### Caches

- For data that is read heavy (data access patterns!)
- Seperate cache tier? Yes, better performance, ability to scale independently
- Persistence?
- Eviction policies:
  - LRU
  - LFU
  - FIFO
- Cache Loading Policies
  - Cache aside
  - Write through
  - Write behind
  - Refresh ahead
- Consistency? ReadYourOwnWrites:
  - Cache in your own region always updated on writes. Tie user to a specific cache cluster.
  - SQL Binlog used to replicate cache invalidations to other servers.
- Client caching, CDN caching, Web server caching, Database caching, Application caching, Cache @Query level, Cache @Object level

### Load Balancer

- Between clients and application servers, application servers and database/cache servers.
- Layer 4 vs Layer 7 - transport vs application. Layer 7 has more info at cost of time and computing resources.
- Pros
  - Prevents requests going to unhealthy servers. Eliminates single point of failure
  - SSL termination - decrypt incoming requests and encrypt server responses
  - Sticky sessions
- Cons
  - Can become performance bottleneck, increased complexity, need for multiple to prevent SPoF
- Primary/Seconday load balancer to address SpoF

### Communication

- TCP: ordered, more overhead, has seq numbers and checksum fields for each packet
- UDP: unordered, less latency, less overhead
- Short Poll vs Long Poll vs Websocket
- JSON vs Thrift. Saving space on the wire. 50% reductions Field tags vs field names.

### Error cases

- Write Ahead Log for services that keep in memory data structures
- Timeouts on service calls set to speed of 1% slowest requests. Maybe hit a bad server.
- Retry mechanism. Exponential backoff with jitter.

### CDN

- Lightens DB load by serving static content (images, videso, CSS, JS)
- Push vs Pull?
  - Push better for small traffic. Pull allows user demanded data to be populated with TTL.
- Generally costly. Charged for data transfer in/out so should not cache infrequently used assets.
  
### Logging/Metrics

- Host level metrics: CPU/Memory/Disk I/O
- Aggregated level metrics: performance of entire database/cache tier
- Business metrics: DAU, retention, revenue
