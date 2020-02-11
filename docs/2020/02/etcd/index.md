# Etcd


`etcd` is a highly available key-value store. 

`redis`, `memcached` are popular key-value stores. These are general-purpose distributed memory caching system often used to speed up dynamic database-driven websites by caching data and objects in memory.

`etcd` cannot be stored in memory, they can only be persisted in disk storage, whereas `redis` can be cached in ram and can also be persisted in disk.

`etcd` does not have various data types. But `redis` and other key-value stores have data-type flexibility.

`etcd` guarantees only high availability, but does not give you the fast querying and indexing. All the nosql key-value stores are built with the goal of fast querying and searching.
