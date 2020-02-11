# System Design Topics


## Some high-level trade-offs

- Performance vs scalability
- Latency vs throughput
- Availability vs consistency

Keep in mind that everything is a trade-off.

## Performance vs scalability

A service is scalable if it results in increased performance in a manner proportional to resources added. Generally, increasing performance means serving more units of work, but it can also be to handle larger units of work, such as when datasets grow.

Many algorithms that perform reasonably well under low load and small datasets can explode in cost if either requests rates increase, the dataset grows or the number of nodes in the distributed system increases.

## Latency vs throughput

Generally, you should aim for maximal throughput with acceptable latency.

## Availability vs consistency

**CAP** theorem

- Consistency
- Availability
- Partition Tolerance

You can only support **two** of the above guarantees.

**Networks aren't reliable, so you'll need to support partition tolerance. You'll need to make a software tradeoff between consistency and availability.**

## Consistency patterns

### Weak consistency
After a write, reads may or may not see it. A best effort approach is taken.

This approach is seen in systems such as **memcached** . Other **real time** use cases such as **VoIP**, **Video chat**, and **realtime multiplayer games**.

For example, if you are on a phone call and lose reception for a few seconds, when you regain connection you do not hear what was spoken during connection loss.

### Eventual consistency

After a write, reads will eventually see it (typically within milliseconds). Data is replicated asynchronously. Eventual consistency works well in highly available systems.

**DNS** and **email** are using eventual consistency.

### Strong consistency

After a write, reads will see it. Data is replicated synchronously. This approach is seen in **file systems** and **RDBMSes**. Strong consistency works well in systems that need transactions.

## Availability patterns

### Fail-over

- Active-passive or master-slave

  Only the active server handles traffic.
  Heartbeats are sent between the active and passive server on standby. If the heartbeat is interrupted, the passive server takes over the active's IP address and resumes service.
  The length of downtime is determined by whether the passive server is already running in 'hot' standby or whether it needs to start up from 'cold' standby.

- Active-active or master-master

  Both servers are managing traffic, spreading the load between them.
  If the servers are public-facing, the DNS would need to know about the public IPs of both servers. If the servers are internal-facing, application logic would need to know about both servers.

### Replication

- Master-slave replication

- Master-master replication


## DNS

Domain name to IP.

## CDN
A globally distributed network of proxy servers, serving content from locations closer to the user.

Amazon's CloudFront is one of the CDNs. It's DNS resolution will tell clients which server to contact.

### Push CDNs

### Pull CDNs

## Load balancer

![sdf](https://camo.githubusercontent.com/21caea3d7f67f451630012f657ae59a56709365c/687474703a2f2f692e696d6775722e636f6d2f6838316e39694b2e706e67)

Load balancers are effective at:
- Preventing requests from going to unhealthy servers
- Preventing overloading resources
- Helping eliminate single points of failure

### Horizontal scaling
Load balancer can also help with horizontal scaling, improving performance and availability.

## Application Layer

Aka. platform layer, added between the web application and database. Adding application layer can be a way to reuse your infrastructure for multiple products or interfaces (a web application, an API, an iPhone app, etc) without writing too much redundant bolerplate code for dealing with caches, databases, etc.

## Database

### RDBMS
A relational database like SQL is a collection of data items orgnized in tables.
ACID is a set of properties of relational database transactions.
- Atomicity - Each transaction is all or nothing
- Consistency
- Isolation
- Durability

#### Scale a relational database: master-slave replication, master-master replication, federation, sharding, denormalization, and SQL tuning.

**Master-slave replication**

The master serves reads and writes, replicating writes to one or more slaves, which serve only reads. Slaves can also replicate to additional slaves in a tree-like fashion. If the master goes offline, the system can continue to operate in read-only mode until a slave is promoted to a master or a new master is provisioned.
![](https://camo.githubusercontent.com/6a097809b9690236258747d969b1d3e0d93bb8ca/687474703a2f2f692e696d6775722e636f6d2f4339696f47746e2e706e67)

Disadvantage(s):
- Additional logic is needed to promote a slave to a master.

**Master-master replication**
![](https://camo.githubusercontent.com/5862604b102ee97d85f86f89edda44bde85a5b7f/687474703a2f2f692e696d6775722e636f6d2f6b7241484c47672e706e67)

Disadvantage(s):
- You'll need a load balancer or you'll need to make changes to your application logic to determine where to write.
- Most master-master systems are either loosely consistent (violating ACID) or have increased write latency due to synchronization.
- Conflict resolution comes more into play as more write nodes are added and as latency increases.

**Disadvantages of replication**
- There is a potential for loss of data if the master fails before any newly written data can be replicated to other nodes.
- Writes are replayed to the read replicas. If there are a lot of writes, the read replicas can get bogged down with replaying writes and can't do as many reads.
- The more read slaves, the more you have to replicate, which leads to greater replication lag.
- On some systems, writing to the master can spawn multiple threads to write in parallel, whereas read replicas only support writing sequentially with a single thread.
- Replication adds more hardware and additional complexity.

**Sharding**

Sharding distributes data across different databases such that each database can only manage a subset of the data. Taking a users database as an example, as the number of users increases, more shards are added to the cluster.

Similar to the advantages of federation, sharding results in less read and write traffic, less replication, and more cache hits. Index size is also reduced, which generally improves performance with faster queries. If one shard goes down, the other shards are still operational, although you'll want to add some form of replication to avoid data loss. Like federation, there is no single central master serializing writes, allowing you to write in parallel with increased throughput.

Common ways to shard a table of users is either through the user's last name initial or the user's geographic location.

**Disadvantages of sharding**
- You'll need to update your application logic to work with shards, which could result in complex SQL queries.
- Data distribution can become lopsided in a shard. For example, a set of power users on a shard could result in increased load to that shard compared to others.
  - Rebalancing adds additional complexity. A sharding function based on consistent hashing can reduce the amount of transferred data.
- Joining data from multiple shards is more complex.
Sharding adds more hardware and additional complexity.

