# Scaling SQL and NoSQL

## Vertical v.s. Horizontal Scaling

### Vertical scaling

Vertical scaling is the process of improving the processing power of the server, by increasing its hardware capacity (e.g. CPU, RAM). While this could be a plausible option, there’s a threshold to the hardware improvements — it’s capped at what’s currently available. In addition, after a certain point, vertical scaling becomes too expensive to be a viable option.

### Horizontal Scaling

Horizontal scaling, on the other hand, achieves scale by increasing the number of servers. Theoretically, you could scale to have as many servers in parallel as you wish, which is why horizontal scaling is the preferred option when databases have to scale. As servers are distributed, we gain the benefits of being able to store more data, but we also inherit the problems of a distributed system. We’ll look at the underlying limitations of scaling and how we can work around them.

## Database Sharding

### What is sharding?  

The concept of database sharding is key to scaling, and it applies to both SQL and NoSQL databases. As the name suggests, we’re slicing up the database into multiple pieces (shards). Each shard has a unique index that corresponds to the type of data it stores. For example, if we choose to index our database by name, shard A can store the data of users with names that start with A–F, shard B could store data from users ages G–M, and so on. When a search query comes in, the database can quickly refer to the index and quickly pinpoint the shard we should look at instead of scanning the entire database.

### Costs of sharding

However, it’s essential to note that database sharding comes at a cost, especially for SQL databases. Monolithic databases like Oracle, MySQL, PostgreSQL do not support automatic sharding and engineers would have to manually write logic to handle the sharding. Often, because of the high cost of maintainability, changing schemas (e.g. how the databases are sharded) becomes challenging.

## Scaling SQL Databases

### Cluster proxy

Assuming that we’ve already applied the sharding logic to our SQL database, you might be asking, how would a query service know which shard to communicate with to retrieve data? If you’re thinking about a middleman, you’re on the right track. A solution is to place a proxy, like a load balancer, that sits in front of the query service and the database. Underneath the hood, the load balancer relies on a Configuration Service such as ZooKeeper to keep track of the shards and their indices. By processing the query, the load balancer will know exactly which shard to direct the request to.

![](image/SQL or NoSQL/1637967053878.png)

### Shard proxy

Thus far, we’ve done a decent job at speeding up the query by looking at a specific shard instead of scanning the entire database — but we can still do better. Similar to what we’ve done before, we can introduce another proxy that sits between the load balancer and the shard, which we’ll call the shard proxy. If the shard is sufficiently large, we can once again shard that proxy to speed up the queries. This process of repeated sharding is called hierarchical sharding.

Apart from redirecting to the specific index, a shard proxy can also further improve queries through actions like caching frequently used data, monitor database health, store and publish metrics about data, and terminate queries if a request is taking too long.

![](image/SQL or NoSQL/1637967066376.png)

### Availability

Now that we’re no longer dealing with a monolithic database, we need to ensure that the data is still available when one instance goes down. In distributed systems, the solution to ensuring availability is to almost always introduce replication, so that there’s no single bottleneck of failure and there will be backup copies available.

Going back to the shards, we can ensure availability using a master-slave architecture. The idea is to have a master shard which is read/write and slave shards that are read-only, in order to ensure a single origin of truth. Whenever there is a write operation, it will be applied directly to the master shard, which will, in turn, propagate the changes to the slave shards. Read queries can be directed to the slave shards in order to reduce the load on the master shard. In the event that the master shard becomes unavailable, the shard proxy can choose a single slave shard to replace the master shard.

![](image/SQL or NoSQL/1637967077677.png)

At a data center level, we could replicate the database to ensure that we have a backup in the event that a data center fails.

### Consistency

SQL databases were not designed with scalability in mind but with ACID properties (Atomicity, Consistency, Isolation, and Durability). As such, a single instance of a SQL database is guaranteed to be consistent. However, once you explore distributed systems with SQL databases and ensure the availability of data, you’re bound to run into the issues of consistency which are inevitable in distributed systems.

A simple analogy would be a factory owner making an announcement without a loudspeaker. If there are only a few workers, each of them would be able to instantly receive his message. Conversely, when there are thousands of workers, the message might take a while to reach the person at the back since each worker would have to rely on the person in front of them to convey them the message.

Going back to the master-slave architecture, it will take some time for the data to be propagated from the master to its slaves. Therefore, it exists at a window of time that the master and its slaves can have different states. In scaling a SQL database, we sacrifice consistency for eventual consistency.

## Scaling NoSQL Databases

### Cassandra wide column dataBase

Contrary to SQL databases, NoSQL databases were designed with scale in mind. Therefore, the scaling process is largely invisible to the end-user as it is done automatically. We’ll nevertheless explore what’s happening beneath the hood. While there are many different types of NoSQL databases and different ways of scaling, I will be using Cassandra’s wide column databases as a reference.

### Shards/nodes are equal

Similar to how we shard a SQL database, NoSQL also has shards of databases that correspond to different indices. However, one key difference is that instead of a master-slave architecture, every shard is equal. In addition, since there will be graph notions involved, I’ll be referring to shards as nodes moving forward, as that’s a more intuitive label.

Nodes are able to communicate and exchange information about each other — they know where a particular data is stored, if the data is not contained within itself. Usually, there are a fixed number of nodes that each node communicates with. I imagine that this is to prevent a O(V²) time complexity, where V is the number of nodes, and to cap it at O(1) time.

When a query comes in, an initial coordinator node is picked within the cluster to server the request via an algorithm. Such algorithms include round-robin, or shortest distance (from request to node). If the coordinator node does not have the data, it will forward the request to another node who has the data, or knows another node that has it.

![](image/SQL or NoSQL/1637977649291.png)

For example, the data for user F is stored on node A. Node D, the coordinator node, can simply communicate with node A, and knows that the data is stored on node A. Therefore, Node D can simply redirect the request to node A.

Similarly, when a write request goes to a coordinator node, it will use consistent hashing to determine which node should the data be stored in.

### Gossip protocol

The whole idea of sharing information through nodes is often referred to as the gossip protocol, where information is transferred from node to node. The term epidemic protocol is an alternative term, because the information is transmitted similarly to how viruses spread in a community.

### Availability

As mentioned previously, data is replicated to prevent a single point of failure and maintain availability.

In Cassandra, since the nodes are considered equals, a node can simply replicate its data in other nodes instead of employing a master-slave architecture. As we already know, when we have a distributed system, data replication happens asynchronously and takes a long time. Therefore, instead of waiting for all nodes to respond to the coordinator node signaling that a successful replication occurred, the coordinator node only needs to wait for x number of nodes to respond. This approach for waiting for x number of successful write responses for a replication to be successful is called quorum writes.

Besides replication at the cluster level, multiple copies of the entire cluster are also stored across different data centers to prevent a single point of failure at the data center level.

### Consistency

In most NoSQL databases, availability is prioritized over consistency since they have BASE (Basically Available, Soft State, Eventually Consistent) properties. In other words, it’s preferred to show stale data than no data at all. After all, there will be eventual consistency amongst the nodes, as explained in the gossip protocol earlier. Given these properties, it would make sense that financial systems that require high precision of data would use SQL databases, whereas less “significant” data like view counts could rely on NoSQL databases.

When a read query goes to the coordinator nodes, which then initiates parallel read requests to replica nodes, it’s possible that different responses are being returned. This discrepancy could be due to the fact that certain replica nodes were unavailable during the replication process, or that the data has yet to be propagated to some nodes. Just as we have quorum writes, we also have quorum reads, in which the response that x number of nodes agree on is returned.
