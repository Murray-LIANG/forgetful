---
title: "Redis vs Zookeeper"
date: 2020-02-10T16:37:31+08:00
lastmod: 2020-02-10T16:37:31+08:00
draft: false
show_in_homepage: true
description_as_summary: true
license: ""

tags: [
    "redis",
    "zookeeper",
]

comment: true
toc: true
auto_collapse_toc: true
---

## Compare
*Copied from stackoverflow*

Redis is fast; really, really fast. It is also immediately consistent, so it's good for
fast moving data sets. The downside is that, running on one server, if it fails then you
lose write access until another server takes it's place. Replacing the server is a manual
operation unless you automate it yourself. (You can still get read access to your data if
you configure a slave instance).

Zookeeper also features immediately consistency. It's not half as quick, but it will 
recover automatically (where possible) in the face of failure, so if you need continuous 
write access, even when your servers fail on you then you'll want to use Zookeeper.

My advice is, use zookeeper for **coordination**: tracking which nodes are active, leader 
election amongst a group, etc. Use redis for datasets that need fast writes but where the 
occasional outage isn't a disaster. Hit counters for web pages for example.

## For Distributed Lock

*Copied from Quora*

Yes, it can be used as a very lightweight distributed locking system.  As you suggest Redis servers are each a point of failure (and their slave/replication and the various promotion systems, including the sentinel code in the more recent Redis releases) are only partial mitigation for such failures.

If you're going to use Redis you'll need to rely on quality client libraries and code to provide failover and resilience ... and partitioning will present serious challenges depending on your application.

Zookeeper might be a better choice for a fault tolerant locking system ... though the transaction rates will be considerably slower than naïve code running against a Redis server.  (I don't know how they'd compare if you had robust client code implementing consensus across a cluster of Redis servers ... and I'm not even sure that a truly robust protocol is theoretically possible over the existing server implementation).

