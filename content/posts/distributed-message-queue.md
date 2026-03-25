---
title: "Distributed Message Queue"
date: 2020-09-30T20:26:20+08:00
lastmod: 2020-09-30T20:26:28+08:00
draft: false
show_in_homepage: true
description_as_summary: true
license: ""

tags: [
    "message queue",
    "queue",
    "distributed",
    "asynchronous processing",
    "rabbitmq",
    "kafka",
]

comment: true
toc: true
auto_collapse_toc: true
---

# Asynchronous processing

Asynchronous processing allows a task to call a service and move on the next task while the service processes the request at its own pace.

# Why not use queue everywhere

Extremely tricky and complicated.

# Typical use cases of queue

## Real-time streaming
Queuing model allows data to be continuously streamed and processed in real time.

## Event driven architecture
Pub-sub (topic) model sends notifications to the subscriber when things change.

## Distributed, fault-tolerant, scalable infrastructure
Multiple services (or microservices) communicate with each other through queues as shared interfaces.

When a microservice fails unexpectedly, a queue is still able to accept messages. This essentially provides a **buffer** for our microservice to recover.

# RabbitMQ vs Kafka
## Traditional message processing vs. Stream processing
Traditional message processing - you apply simple computations on the messages, in most cases **individually** per message.

Stream processing - you apply complex operations on **multiple input streams and multiple records** (messages) at the same time (like aggregations and joins).

Traditional messaging system cannot go `back in time`. It **deletes the messages automatically** after they got delivered to all subscribed consumers. In contract, Kafka keeps the messages as it uses a pull based model (consumers pull data out of Kafka) for a configurable amount of time. This allows consumers to rewind and consume messages multiple times or if you add a new consumer, it can read the complete history. This makes stream processing possible, because it allows for more complex applications.

Stream processing is more about processing infinite input stream, in contract to batch processing that is applied to finite inputs.

## Kafka and RabbitMQ details
1. Generally, use Kafka if you have a fire hose of events (20k+/sec per producer) you need delivered in partitioned order **at least once** with a mix of online and batch consumers, but most importantly you're OK with your consumers managing the state of your "cursor" on the Kafka topic.

    Kafka's main superpower is that it is less like a queue system and more like a **circular buffer** that scales as much as your disk on your cluster, and thus allows you to **be able to re-read messages**.

2. Use RabbitMQ if you have messages (20k+/sec per queue) that need to routed in complex ways to consumers, you want **per-message delivery guarantees**, you need one or more features of protocols like AMQP, and you want the **broker to manage that state** of which consumer has been delivered which message.

    RabbitMQ's main superpower is that it's scalable, high performance queue system with well-defined consistency rules, and ability to create interesting exchange toplogies.

### Different design philosophies

- **RabbitMQ is broker-centric**, focused around **delivery guarantees** between producers and consumers, with transient preferred over durable messages.
- **Kafka is producer-centric**, based around **partitioning** a fire hose of event data into durable message **brokers with cursors**, supporting batch consumers that may be offline, or online consumers that want message at low latency.


- **RabbitMQ** uses the broker itself to maintain state of what's consumed via **message acknowledgements**.
- **Kafka** doesn't have message acknowledgements, it assumes **the consumer tracks of what's been consumed so far**. Kafka use **Zookeeper** to reliably maintain their state across a cluster.

- **RabbitMQ** presumes that consumers are mostly **online**, and any messages "in wait" are held opaquely.
- **Kafka** was based from the beginning around both **online and batch consumers**, and also has producer message batching - it's designed for holding and distributing large volumes of messages.

- **RabbitMQ** provides **rich routing** capabilities with AMQP exchange, binding and queuing model.
- **Kafka** has a very simple routing approach - in AMQP parlance it uses **topic** exchanges only.

- **RabbitMQ**'s philosophy is to make the cluster transparent, as if it were a virtual broker.
- **Kafka** makes partitions explicit, by **forcing the producer to know it is partitioning a topic's messages across several nodes**, this has the benefit of preserving ordered delivery within a partition.

- **RabbitMQ** ensures queued messages are stored **in published order** even in the face of requeues or channel closure. One can setup a similar topology & order delivery to Kafka using the **consistent hash exchange** or **sharding plugin**, or even more interesting topologies.
- **Kafka** presumes that producers generate a massive stream of events on their own timetable - there's no room for throttling producers because consumers are slow, since the data is too massive. The whole job of Kafka is to provide the **"shock absorber" between the flood of events and those who want to consume them** in their own way - some online, others offline, only batch consuming on an hourly or even daily basis.