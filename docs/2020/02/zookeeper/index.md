# Zookeeper


https://stackoverflow.com/questions/37293928/zookeeper-vs-in-memory-data-grid-vs-redis

By default, Zookeeper replicates all your data to every node and lets clients watch the data for changes. Changes are sent very quickly (within a bounded amount of time) to clients. You can also create "ephemeral nodes", which are deleted within a specified time if a client disconnects. ZooKeeper is **highly optimized for reads, while writes are very slow** (since they generally are sent to every client as soon as the write takes place). Finally, the maximum size of a "file" (znode) in Zookeeper is 1MB, but typically they'll be single strings.

Taken together, this means that **zookeeper is not meant to store for much data, and definitely not a cache**. Instead, it's for **managing heartbeats/knowing what servers are online, storing/updating configuration, and possibly message passing** (though if you have large #s of messages or high throughput demands, something like RabbitMQ will be much better for this task).

Basically, ZooKeeper (and Curator, which is built on it) helps in handling the mechanics of **clustering -- heartbeats, distributing updates/configuration, distributed locks**, etc.

It's not really comparable to Redis, but for the specific questions...

1. It doesn't support any computation and for most data sets, won't be able to store the data with any performance.

2. It's replicated to all nodes in the cluster (there's nothing like Redis clustering where the data can be distributed). All messages are processed atomically in full and are sequenced, so there's no real transactions. It can be USED to implement cluster-wide locks for your services (it's very good at that in fact), and there are a lot of locking primitives on the znodes themselves to control which nodes access them.

3. Sure, but ZooKeeper fills a niche. It's a tool for making a distributed applications play nice with multiple instances, not for storing/sharing large amounts of data. Compared to using an IMDG for this purpose, Zookeeper will be faster, manages heartbeats and synchronization in a predictable way (with a lot of APIs for making this part easy), and has a "push" paradigm instead of "pull" so nodes are notified very quickly of changes.

The quotation from the linked question...

    A canonical example of Zookeeper usage is distributed-memory computation

... is, IMO, a bit misleading. You would use it to **orchestrate the computation, not provide the data**. For example, let's say you had to process rows 1-100 of a table. You might put 10 ZK nodes up, with names like "1-10", "11-20", "21-30", etc. Client applications would be notified of this change automatically by ZK, and the first one would grab "1-10" and set an ephemeral node clients/192.168.77.66/processing/rows_1_10

The next application would see this and go for the next group to process. **The actual data to compute would be stored elsewhere (ie Redis, SQL database, etc)**. If the node failed partway through the computation, another node could see this (after 30-60 seconds) and pick up the job again.

I'd say the canonical example of ZooKeeper is leader election, though. Let's say you have 3 nodes -- one is master and the other 2 are slaves. If the master goes down, a slave node must become the new leader. This type of thing is perfect for ZK.
