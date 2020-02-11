---
title: "System Design"
date: 2020-02-10T16:37:31+08:00
lastmod: 2020-02-10T16:37:31+08:00
draft: false
show_in_homepage: true
description_as_summary: true
license: ""

tags: [
    "system design",
]

comment: true
toc: true
auto_collapse_toc: true
---


No design is correct or wrong. There are just good designs and bad designs which **heavily depend on the use case**.

Hence, it is extremely important to **clarify the requirements** for the problem asked.

## Basic Terminologies

### Replication
Frequently copying the data across multiple machines.

### Consistency
The data is same across the cluster with multiple machines. You can read or write to/from any node and get the same data.

### Eventual Consistency
All machines will have the same data eventually. It's possible that at a given instance, those machines have different versions of the same data (temporarily inconsistent) but they will eventually reach a state where they have the same data.

### Availability
The ability to always respond to queries (read/write) irrespective of nodes going down.

### Partition Tolerance
The cluster continues to function even if there is a "partition" (communication break) between two nodes (both nodes are up, but can't communicate).

### Vertical Scaling and Horizontal Scaling
To scale vertically is to increase the resources of the server (RAM, CPU, storage, etc.)

To scale horizontally is adding more servers.

### Sharding
When data does not fit on a single machine, sharding refers to splitting the very large database into smaller, faster and more manageable parts called data shards.

## CAP Theorem
In a distributed system, it is impossible to simultaneously guarantee all of the following:
- Consistency
- Availability
- Partition Tolerance

http://ksat.me/a-plain-english-introduction-to-cap-theorem/

## Steps to Approach a Problem

### Feature expectations
It is extremely important to get a very clear understanding of whats the **requirement** for the question.

### Estimations
Estimate the **scale** required for the system.

The goal of this step is to understand the level of sharding required (if any) and to zero down on the design goals for the system.

For example, if the total data required for the system **fits on a single machine**, we might **not** need to go into sharding and the complications that go with a distributed system design. Or if the most **frequently used data** fits on a single machine, in which case **caching** could be done on a single machine.

### Design goals
Figure out what are the most important goals for the system.

It is possible that there are systems which are latency systems in which case a solution that does not account for it, might lead to bad design.

### Skeleton of the design
30-40 mins is not enough time to discuss every single component in detail. As such, a good strategy is to **discuss a very high level** with the interviewer and go into **a deep dive of components as enquired** by the interviewer.

### Deep dive
