---
title: "Service Discovery and Registration"
date: 2020-09-21T20:17:45+08:00
lastmod: 2020-09-21T20:17:55+08:00
draft: false
show_in_homepage: true
description_as_summary: true
license: ""

tags: [
    "service discovery",
    "service registration",
]

comment: true
toc: true
auto_collapse_toc: true
---

# Difficult of Service Discovery

Service instances have dynamically assigned network locations. Moreover, the set of service instances changes dynamically because of autoscaling, failures, and upgrades. Consequently, your client code needs to use a more elaborate service discovery mechanism.

![](/forgetful/images/service-discovery-and-registration-difficult-service-discovery.png)

# The Client-Side Discovery Pattern

When using client‑side discovery, **the client is responsible for determining the network locations of available service instances and load balancing requests across them**.

The client queries a **service registry**, which is a database of available service instances. The client then uses a **load‑balancing algorithm to select one** of the available service instances and makes a request.

![](/forgetful/images/service-discovery-and-registration-client-side-pattern.png) 

The network location of a service instance is registered with the service registry **when it starts up**. It is removed from the service registry **when the instance terminates**. The service instance’s registration is typically **refreshed periodically using a heartbeat** mechanism.

## Benefits and Drawbacks

The client knows about the available services instances, it can make intelligent, application‑specific load‑balancing decisions such as using hashing consistently.

One significant drawback of this pattern is that it couples the client with the service registry. You must implement client‑side service discovery logic for each programming language and framework used by your service clients.

# The Server-Side Discovery Pattern

The client makes a request to a service via a load balancer. The **load balancer queries the service registry** and routes each request to an available service instance. As with client‑side discovery, **service instances are registered and deregistered with the service registry**.

![](/forgetful/images/service-discovery-and-registration-server-side-pattern.png)

## Benefits and Drawbacks

One great benefit of this pattern is that details of discovery are abstracted away from the client. Clients simply make requests to the load balancer. This eliminates the need to implement discovery logic for each programming language and framework used by your service clients. Also, as mentioned above, some deployment environments provide this functionality for free.

This pattern also has some drawbacks, however. Unless the **load balancer** is provided by the deployment environment, it is yet **another highly available system component that you need to set up and manage**.

# The Service Registry

The service registry is a key part of service discovery. It is a **database containing the network locations of service instances**. A service registry needs to be highly available and up to date. Clients can cache network locations obtained from the service registry. However, that information eventually becomes out of date and clients become unable to discover service instances. Consequently, a service registry consists of a cluster of servers that use a replication protocol to maintain consistency.

Common service registries include:

1. **etcd** – A highly available, distributed, consistent, key‑value store that is used for shared configuration and service discovery. Two notable projects that use etcd are Kubernetes and Cloud Foundry.
2. **consul** – A tool for discovering and configuring services. It provides an API that allows clients to register and discover services. Consul can perform health checks to determine service availability.
3. **Apache Zookeeper** – A widely used, high‑performance coordination service for distributed applications. Apache Zookeeper was originally a subproject of Hadoop but is now a top‑level project.

# Service Registration Options

## The Self‑Registration Pattern

When using the self‑registration pattern, a service instance is responsible for registering and deregistering **itself** with the service registry. Also, if required, a service instance sends **heartbeat** requests to **prevent its registration from expiring**. The following diagram shows the structure of this pattern.

![](/forgetful/images/service-discovery-and-registration-self-registration-pattern.png)

### Benefits and Drawbacks

One benefit is that it is relatively simple and doesn’t require any other system components. 

However, a major drawback is that it couples the service instances to the service registry. You must implement the registration code in each programming language and framework used by your services.

## The Third-party Registration Pattern
When using the third-party registration pattern, service instances aren’t responsible for registering themselves with the service registry. Instead, another system component known as the **service registrar handles the registration**. The service registrar tracks changes to the set of running instances by either **polling the deployment environment or subscribing to events**. When it notices a newly available service instance it registers the instance with the service registry. The service registrar also deregisters terminated service instances. The following diagram shows the structure of this pattern.
![](/forgetful/images/service-discovery-and-registration-third-party-pattern.png)

### Benefits and Drawbacks

A major benefit is that services are **decoupled** from the service registry. You don’t need to implement service‑registration logic for each programming language and framework used by your developers. Instead, service instance registration is handled in a centralized manner within a dedicated service.

One drawback of this pattern is that unless it’s built into the deployment environment, it is yet **another highly available system component that you need to set up and manage**.

# Example of Server-Side Discovery and Registration

```console
                                          Requests       +---------+
                                     +------------------>+ Node #1 +--------+
                                     |                   +---------+        |
                                     |                                      |
                 +-----------+       |                                      |
                 |           |       |    Requests       +---------+        |
     Requests    |  API /    +-------------------------->+ Node #2 +--------+
+--------------->+  Gateway  |       |                   +---------+        |
                 |           |       |                                      |
                 |           |       |                                      |
                 +-----+-----+       |    Requests       +---------+        |
                       |             +------------------>+ Node #3 +--------+
                       |                                 +---------+        |
                       | Subscribe for changes                              |
                       | (New node added or                                 |
                       |  node removed)                                     |
                       |                                                    |
                       |           +------------+                           |
                       |           |            |  Periodic Heartbeat       |
                       +---------->+    etcd    +<--------------------------+
                                   |            |
                                   +------------+
```

## API / Gateway

1. Acts as the load balancer: Load-balancing incoming requests to available nodes.
2. Subscribes for changes of nodes: adding new node or removing exist node.

## etcd
1. Acts as Service Registry: storing a key-value pair with TTL for each node.

## Node
1. Services the requests.
2. Sends periodic heartbeat which refreshes the TTL key to etcd store.

## More detail
**Node** represents whatever service in the system that can scale horizontally and to whom requests are load-balanced.

On startup node is taking whatever port is available starting from 8080, then it’s putting lease in etcd, which is basically setting key-value pair **with 10 seconds TTL** (time to live), once it is set, it starts **refreshing TTL every 5 seconds**.

If **Node** dies, hangs, key-value pair will be removed, at the same time making this node unavailable and undiscoverable for the gateway.

Key is designed in the following way `heartbeat|{NODE_TYPE}|{url}`, so the gateway can make use of **etcd range subscribe to watch only for keys that starts with heartbeat**, also from that key, information about node type and url that it is hosted at, can be retrieved.

**Gateway** manages dictionary structure that holds information about available nodes in the system. It subscribes to heartbeats in etcd using **WatchRange** and updates dictionary state accordingly.

Using that dictionary incoming requests are load-balanced to random  available or round-robin node.