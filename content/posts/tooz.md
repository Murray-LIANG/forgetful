---
title: "Tooz"
date: 2020-02-10T16:37:31+08:00
lastmod: 2020-02-10T16:37:31+08:00
draft: false
show_in_homepage: true
description_as_summary: true
license: ""

tags: [
    "tooz",
]

comment: true
toc: true
auto_collapse_toc: true
---

Document page: https://docs.openstack.org/tooz/latest/

Tooz is a library to manage the distributed primitives like **group membership** protocol, **lock service** and **leader election**. It provides a common coordination API for different drivers, like zookeeper, redis, memcached, etcd, .etc.

## Drivers

Document page: https://docs.openstack.org/tooz/latest/user/drivers.html

### Zookeeper

The zookeeper is the reference implementation and provides the **most solid** features as it's possible to build a cluster of zookeeper servers that is resilient towards network partitions for example.

#### Considerations
Primitives are based on **sessions** (and typically require careful selection of session heartbeat periodicity and server side configuration of session expiry).

### Redis

The redis driver is a basic implementation and provides reasonable resiliency when used with **redis-sentinel**. A lot of the features provided in tooz are based on **timeout (heartbeats, locks, etc)** so are **less resilient** than other backends.

#### Considerations
- **Less resilient** than other backends such as zookeeper.
- Primitives are often based on **TTL(s)** that may expire before being renewed.

### Memcached
The memcached driver is a basic implementation and provides **little resiliency**, though it’s much simpler to setup. A lot of the features provided in tooz are based on **timeout (heartbeats, locks, etc)** so are **less resilient** than other backends.

#### Considerations
- **Less resilient** than other backends such as zookeeper and redis.
- Primitives are often based on **TTL(s)** that may expire before being renewed.
- **Lacks certain primitives** (compare and delete) so certain functionality is fragile and/or broken due to this.