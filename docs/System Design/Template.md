# Template

## (1) Gather Requirements [3 min]

1. Use case
   1. Who will use
   2. Usage patterns
2. Real time or delayed data ok? Stale data okay?
   1. Read your own writes usually a must otherwise confusing.
      1. Can use cache layer to address this. Cache in your own region always updated on writes.
      2. Binlog used to replicate cache invalidations to other servers. 

## (2) Functional + Non Functional Requirements [5 min]

1. FunctionaL requirements
   1. Define API
2. Non-Functional requirements
3. Extended requirements

## (3) Capacity Estimations [3 min]

1. Traffic estimation (read request per sec, write requests per sec)
2. Storage estimation (storage needed to store worth 3 yrs of stored 'object')
3. Bandwidth estimate (#of bytes/sec system should handle for incoming and outgoing traffic)
4. Cache estimate (memory needed to cache some of the hot read responses, 80-20 rule)
   1. If we are using a cache, what is the kind of data we want to store in cache
   2. How much RAM and how many machines do we need for us to achieve this?

## (4) Data Model [5 min]

Defining the data model in the early part of the interview will clarify how data will flow between different system components. Later, it will guide for data partitioning and management. The candidate should identify various system entities, how they will interact with each other, and different aspects of data management like storage, transportation, encryption, etc. Here are some entities for our Twitter-like service:

**User**: UserID, Name, Email, DoB, CreationDate, LastLogin, etc.  
**Tweet**: TweetID, Content, TweetLocation, NumberOfLikes, TimeStamp, etc.  
**UserFollow**: UserID1, UserID2  
**FavoriteTweets**: UserID, TweetID, TimeStamp

Which database system should we use? Will NoSQL like [Cassandra](https://en.wikipedia.org/wiki/Apache_Cassandra) best fit our needs, or should we use a MySQL-like solution? What kind of block storage should we use to store photos and videos?

## (5) High Level Design [5-10 min]

Draw a block diagram with 5-6 boxes representing the core components of our system. We should identify enough components that are needed to solve the actual problem from end to end.

## (6) Deep Dive [15-20 min]

1. Scaling the algorithm
2. Scaling individual components:
   - Availability, Consistency and Scale story for each component
   - Consistency and availability patterns
3. Think about the following components, how they would fit in  and how it would help
   - DNS
   - CDN [Push vs Pull]
   - Load Balancers [Active-Passive, Active-Active, Layer 4, Layer 7]
   - Reverse Proxy
   - Application layer scaling [Microservices, Service Discovery]
   - DB [RDBMS, NoSQL]
     - RDBMS 
       - Master-slave, Master-master, Federation, Sharding, Denormalization, SQL Tuning
     - NoSQL
       - Key-Value, Wide-Column, Graph, Document
       - Fast-lookups:        
         - RAM  [Bounded size] => Redis, Memcached
         - AP [Unbounded size] => Cassandra, RIAK,Voldemort
         - CP [Unbounded size] => HBase, MongoDB,Couchbase, DynamoDB
   - Caches
     - Client caching, CDN caching, Web server caching, Database caching, Application caching, Cache @Query level, Cache @Object level
     - Eviction policies:
       - LRU
       - LFU
       - FIFO
     - Cache Loading Policies
       - Cache aside
       - Write through
       - Write behind
       - Refresh ahead
   - Asynchronism
     - Message Queues
     - Task Queues
   - Communication
     - TCP
     - UDP
     - REST
     - RPC
     - Thrift
     - GraphQL

