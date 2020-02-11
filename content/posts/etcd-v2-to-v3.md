---
title: "From Etcd v2 to v3"
date: 2020-02-10T16:37:31+08:00
lastmod: 2020-02-10T16:37:31+08:00
draft: false
show_in_homepage: true
description_as_summary: true
license: ""

tags: [
    "etcd",
    "etcd v2",
    "etcd v3",
]

comment: true
toc: true
auto_collapse_toc: true
---

In etcd3, the base server interface uses **gRPC** instead of **JSON** for increased efficiency. Support for JSON endpoints is maintained through a gRPC gateway. The new API revisits the design of key expiry **TTLs**, replacing them with lightweight streaming **lease** keepalive model. **Watchers** are redesigned as well, replacing the older event model with one that **streams and multiplexes events over key intervals**. The v3 data model does away with explicit key hierarchies and unreliable watch windows, replacing them with **a flat binary key space  with transactional, multiversion concurrency control semantics**.

## Efficiency and scaling
### RPCs
Native etcd3 clients communicate over a **gRPC** protocol. The protocol messages are defined using **protobuf**, which simplifies the **generation of efficient RPC stub code** and makes protocol extensions **easier to manage**. For comparison, even after optimizing etcd2’s client-side JSON parsing, etcd3 gRPC still holds a 2x message processing performance edge over etcd2. Likewise, gRPC is better at handling connections. Where **gRPC uses HTTP2 to multiplex multiple streams of RPCs over a single TCP connection**, a JSON client must establish a new connection for every request.

### Leases
Keys expire in etcd2 through a **time-to-live (TTL)** mechanism. For every key with a TTL, a **client** must **periodically refresh the key** to keep it from being automatically deleted when the TTL expires. Each refresh establishes a new connection and issues a consensus proposal to etcd to update the key. To keep all TTL keys alive, an **idling** cluster must have a **minimum request throughput** of the number of TTL keys divided by the average TTL.

Leases in etcd3 replace the earlier notion of key TTLs. Leases reduce keep-alive traffic and eliminate steady-state consensus updates. Instead of a key having a TTL, **a lease with a TTL is attached to a key**. When the lease’s TTL expires, it deletes **all attached keys**. This model reduces keep-alive traffic when multiple keys are attached to the same lease. The keep-alive connection thrashing in etcd2 is avoided by **multiplexing keep-alives on a lease’s single gRPC stream**. Likewise, keep-alives are processed by the **leader**, avoiding any consensus overhead when idling.

### Watchers
A watch in etcd waits for changes to keys. Unlike systems such as ZooKeeper or Consul that return one event per watch request, etcd can **continuously watch** from the current revision. In etcd2, these streaming watches use **long polling over HTTP**, forcing the etcd2 server to wastefully **hold open a TCP connection per watch**. When an application with thousands of clients watches thousands of keys, it can quickly exhaust etcd2 server socket and memory resources.

The etcd3 API **multiplexes watches on a single connection**. Instead of opening a new connection, a client **registers a watcher on a bidirectional gRPC stream**. The stream delivers events tagged with a watcher’s registered ID. Multiple watch streams can even **share the same TCP connection**. Multiplexing and stream connection sharing reduce etcd3’s memory footprint by at least an order of magnitude.

## Concurrency control
When **multiple clients concurrently** read and modify a key or a set of keys, it is important to have **synchronization primitives** to prevent data races from corrupting application state. For this purpose, etcd2 offers both load-link/store-conditional and compare-and-swap operations; a client specifies a previous revision index or previous value to match before updating a key. Although these operations suffice for simple semaphores and limited atomic updates, they are inadequate for describing more sophisticated approaches to serializing data access, such as distributed locks and transactional memory.

etcd3 can **serialize multiple operations into a single conditional mini-transaction**. Each transaction includes a conjunction of **conditional guards** (e.g., checks on key version, value, modified revision, and creation revision), a list of the **operations** to apply when all conditions evaluate to **true** (i.e., Get, Put, Delete), and a list of operations to apply if any of the conditions evaluate to **false**. Transactions make distributed locks safe in etcd3 because accesses can be conditional based on whether the client still holds its lock. This means that even if a client loses its claim on a lock, whether due to clock skew or missing expiration events, etcd3 will refuse to honor the stale request.