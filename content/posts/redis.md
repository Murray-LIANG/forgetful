---
title: "Redis"
date: 2020-02-10T16:37:31+08:00
lastmod: 2020-02-10T16:37:31+08:00
draft: false
show_in_homepage: true
description_as_summary: true
license: ""

tags: [
    "redis",
]

comment: true
toc: true
auto_collapse_toc: true
---

## Data structures
- Binary-safe strings
- Lists
- Sets
- Sorted sets
- Hashes
- Bit arrays (bitmaps)
- HyperLogLogs

## LRU cache
Used as a caching system.

## Master slave replication
- async replication is used.

## Persistence

Several options:
- RDB: Redis Database Backup, hot snapshot that is taken periodically and is meant for point-in-time recovery. An RDB file is literally a dump of all user data stored in an internal, compressed serialization format.
- AOF: Append Only File, persist the dataset by taking a snapshot and appending it with changes as they arrive. Redis can be set up to rewrite the AOF file based on configurable thresholds or perform this operation on demand.

## High availability

Provided by Redis Sentinel:
- monitoring
- notification
- automatic failover
- configuration provider: clients can use Sentinel for service discovery in order to get the current Redis **master** server. Refer to: https://redis.io/topics/sentinel-clients

## Redis cluster

Provides the abilities to:
- automatically split your dateset among multiple nodes.
- continue operations when a subset of the nodes are experiencing failures or are unable to communicate with the rest of the cluster.

The topology could be like the one below. The cluster uses three nodes running two redis instances per each.
![](/forgetful/images/redis-cluster-3-nodes.png)

Refer to https://www.linode.com/docs/applications/big-data/how-to-install-and-configure-a-redis-cluster-on-ubuntu-1604/