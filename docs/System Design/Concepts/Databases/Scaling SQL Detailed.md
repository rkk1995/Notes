# Scaling SQL Detailed

## Replication and Sharding

There are two basic strategies. Replication copies the entire dataset to another machine, so that the number of read queries can be doubled and so we have a hot standby of our data. Sharding (or partitioning) splits the dataset into multiple shards, and assigns the shards to different servers, so that the data writes and data size can be scaled beyond what a single server can handle.

Replication allows more reads, but not more writes, since every write must be copied to every machine, a phenomenon called write amplification.
Sharding allows more writes as well as reads, but risks losing data since the data is not stored redundantly and also makes it hard to do cross-shard queries and transactions.
Often replication is combined with sharding. In this case we can handle more reads and writes, but also have the reliability advantage. 3 replicas per shard are usually enough. In this case we still have the downsides of write amplification and of difficulties with cross-shard queries.

## Leader Follower

The simplest and most common way of replication is leader/follower. Here we have one database server which is the leader, and the others are the followers. All writes go to the leader, then get replicated out to the followers. This can be easily implemented by taking the WAL that the leader is already keeping and transmitting it to the other servers, where it is replayed. Oftentimes databases will keep a separate log for replication, because the WAL format is often overoptimized for disk writes. We can read from the followers to reduce load on the leader. The followers also are hot standbys and can replace a leader if it crashes.

Problem #1: how do we avoid stale reads when it takes a short while for changes to replicate over? For example, are we sure we will read our own writes? Or, are we sure reads are monotonic (if we read from an older follower, might we see a value disappear that we just read from a more recent follower?) The only way to be sure everything is fresh is to read everything from the leader, but this nullifies the performance advantage. This problem is often punted over to the developer, who is responsible for making sure stale reads are not a risk to their code.

Problem #2: when do we acknowledge the write back to the client? If we wait until the write is replicated (synchronous mode), we block and slow down, and risk timeouts if the other server suddenly becomes unavailable. But if we don’t wait (asynchronous mode), we might not be able to read our own writes, and we risk losing unreplicated data when the leader crashes. Which brings us to…

Problem #3: how to do failover without downtime? All clients send their writes to the leader, but what happens if the leader fails? We must appoint a new leader from the followers. However, how can we detect leader failure? We might use a heartbeat: if the leader stops sending a heartbeat signal, the followers hold an election and choose a new leader from the followers. However, the network may be partitioned, and the leader is still accepting writes, blissfully unaware it’s no longer a leader, and now those writes will be discarded after it rejoins, while they were acknowledged to the client. Also, when the new leader was elected, some changes from the old leader might not have replicated over yet, those are now lost. In any case, while the new leader is being appointed there is no leader, so no writes can occur. In other words, the database is down (briefly).

This is the approach most traditional RDBMS’s use. We cannot scale writes using this approach. Going leaderless has its disadvantages, so is there a way to make leaders work and still scaling writes? One way of doing this is to shard the dataset across multiple replica sets. Many distributed databases work this way: MongoDB, HBase, Elasticsearch and Spanner (more on that later).

## Sharding

Now the problem becomes one of how to shard the data across servers. If we want the database to do it automatically, there are two major strategies:

- Sharding by a range of the key. E.g. split the A-Z key space into A-L, M-Z. This can suffer from “hotspotting”, where all writes end up on the same shard because they have a similar key (e.g. using an ISO timestamp as a key), which nullifies the benefits of sharding.
- Sharding by a range of the hash of the key. This spreads keys evenly across the shards, but comes with the downside of no longer being able to do range queries efficiently.

In other words, as a developer you have to be aware of the sharding strategy and its consequences.
