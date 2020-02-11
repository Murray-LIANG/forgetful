# Distributed



## Partitioning / Sharding
github.com/Murray-LIANG/nodering

## Distributed Lock

- Could use etcd as backend. Some implementation: https://github.com/etcd-io/etcd/blob/master/clientv3/concurrency/mutex.go
- If two clients request to lock at the same time, the later client would change the value of same key in etcd3 but with new revision. The lock logic needs to wait for old revision deletion which means the older lock is released.
- To make sure locks released even when clients crash, use expiration on the key.
- If clients crash then start or some blocking action executed longer than lock expiration, there could be two clients holding the lock at the same time. Use tech like fencing to keep the data right. Refer to https://martin.kleppmann.com/2016/02/08/how-to-do-distributed-locking.html
- Could start another thread as the heartbeat which will be called periodically at a given rate to avoid locks, sessions and other from being automatically closed/discarded by etc3. During each beat, the lock lease will be refreshed. This could make sure the lock is always acquired as long as the client is alive even some blocking action executed long time.

## Distributed Queue

- Set up two queues: a Pending Queue and a Running Queue. All items are in the Pending Queue initially.
- These two queues could be stored in etcd3. Then a worker retrieves an item from Pending Queue, transfers it to Running Queue, and creates a lock on the item. These three actions could be put into a transaction of etcd3. The lock here is guarded by a lease, which is automatically removed if the worker dies.
- Three could be another process to scan the items in Running Queue and transfers the ones which are not locked (expired due to worker dies) to Pending Queue.

## Heartbeat

- One server listens on url `/localhost/heartbeat`, several clients send request to server, then server would know which clients are alive.
- Could be based on rpc or http directly.
- The info of which nodes are alive could be stored in etcd shared by a stand-by heartbeat server.
- The active-passive heartbeat server could be monitored by keepalived and expose one floating IP.


