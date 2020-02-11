---
title: "Sharding Pinterest"
date: 2020-02-10T16:37:31+08:00
lastmod: 2020-02-10T16:37:31+08:00
draft: false
show_in_homepage: true
description_as_summary: true
license: ""

tags: [
    "sharding",
    "system design",
    "pinterest",
]

comment: true
toc: true
auto_collapse_toc: true
---

Note of https://medium.com/@Pinterest_Engineering/sharding-pinterest-how-we-scaled-our-mysql-fleet-3f341e96ca6f

## Feature Required

1. Create Pins.
2. Like other Pins.
3. Follow other Pinners.
4. View a home feed of all the Pinners he/she follows.
5. Support asking for N number of Pins in a board in a deterministic order (such as reverse creation time or user specified ordering). 

## Scale Required

50M Pins have been saved by Pinners onto 1B boards.

## Design Goals

1. Stable and high available.
2. Low latency.
3. Eventually consistency.

## Skeleton of the Design

### Shard
The data spans (is sharded to) multiple databases. Then we couldn't use the database's joins, foreign keys or indexes to gather all data.

### Load Balancer
Need to support load balancing our data. **The data doesn't move once it is stored into one shard.** If we had to move data, it was better to move an entire virtual node to a different physical node.

### Replication
Could **replica all data to a slave machine for backup (high availability)**. But we only interact with the master in production, **never read/write to a slave in production** because slave lag causes bugs and once you're sharded, there's generally no advantage to interacting with a slave in production.

## Deep Dive

### How We Shard
We started with **8 servers** (MySQL001A ~ MySQL008A) running one MySQL instance each. Counting in the slave machines (MySQL001B ~ MySQL008B), we have **16 servers** totally.

Each server can have multiple databases. In this example, **each server has 512 databases**. Each database is a shard of our data. So we have 512 * 8 = 4096 shards.

MySQL001A has shards db00000 ~ db00511, while MySQL008A has db03584 ~ db04095.

Once a piece of data lands in a shard, it never moves outside that shard. However, we could move whole shard to to other machine to get more capacity for example.

We maintain **a configuration table that says which machines these shards are on**:
```json
[{"range":     (0,511), "master": "MySQL001A", "slave": "MySQL001B"},
 {"range": (512, 1023), "master": "MySQL002A", "slave": "MySQL002B"},
    ...
 {"range": (3584, 4095), "master": "MySQL008A", "slave": "MySQL008B"}]
```

This config only changes when we need to move shards around or replace a host. If a master dies, we can promote the slave and then bring up a new slave. **The config lives in ZooKeeper/etcd and, on update, is sent to services that maintain the MySQL shard.**

Each shard contains the same set of tables: `pins`, `boards`, `users_have_pins`, `users_like_pins`, `pins_liked_by_users`, etc.

### How We Generate UUID
Next, we need to design a way to **generate UUID for all objects**. In our design, UUID is a 64 bit long which contains 16-bits shared ID, 10-bits type ID, 36-bits local ID and 2-bits reserved for future usage.

```python
#rr <   shard id   > < type id> <    local id                      >
 00 0100100101011010 0101010100 010101010101010101010101010101010110
# while `rr` is reserved.
```

Given this Pin: https://www.pinterest.com/pin/241294492511762325/, decompose the Pin ID 241294492511762325:

```python
ShardID = (241294492511762325 >> 46) & 0xFFFF = 3429
TypeID  = (241294492511762325 >> 36) & 0x3FF = 1
LocalID = (241294492511762325 >>  0) & 0xFFFFFFFFF = 7075733
```

So this Pin object lives on shard 3429. It's type is 1 (i.e. `Pin`), and it's in the row 7075733 in the `pins` table.

Shard `3429` is on host `MySQL007A`
```python
conn = MySQLdb.connect(host='MySQL007A')
conn.execute('SELECT data FROM db03429.pins where local_id=7075733')
```

### How We Model Data Objects
There are two types of data: objects and mappings.

#### Object Tables
Such as `pins`, `users`, `boards` and `comments`, would have below columns, ID (local ID, an auto-incrementing primary key) and a blob of data that contains a JSON with all the object's data.

```sql
CREATE TABLE pins (
    local_id INT PRIMARY KEY AUTO_INCREMENT,
    data TEXT,
    ts TIMESTAMP DEFAULT CURRENT_TIMESTAMP
)
```

For example, a Pin object looks like this:
```json
{
    "details": "New Star Wars character", 
    "link": "http://webpage.com/asdf", 
    "user_id": 241294629943640797, 
    "board_id": 241294561224164665, …
}
```

To create a new Pin, we:
1. gather all the data and create a JSON blob.
2. decide on a shard ID (we prefer to choose the same shard ID as the board it's inserted into, but that's not necessary).
3. the type is 1 for Pin.
4. connect to that database, and insert the JSON into the `pins` table.
5. MySQL will give back the auto-incremental local ID.
6. compose the 64-bit UUID with shard, type and new local ID.

To delete a Pin, we can delete its row in MySQL, and add a JSON field called `active` and set it to `false`.

#### Mapping Tables

A mapping table links one object to another, such as a board to the pins on it. The MySQL table for a mapping contains three columns: a 64-bit `from` ID, a 64-bit `to` ID and a sequence ID. There are index keys on the (`from`, `to`, `sequence`) triple, and they live on the shard of the `from` ID.

```sql
CREATE TABLE board_has_pins (
    board_id INT,
    pin_id INT,
    sequence INT,
    INDEX( board_id, pin_id, sequence)
)
```

Mapping tables are unidirectional, such as a `board_has_pins` table. If you need the opposite direction, you'll need a separate `pin_owned_by_board` table. The `sequence` ID gives an ordering (our ID's can't be compared across shards as the new local ID offsets diverge). We usually insert new Pins into a new board with a `sequence` ID = unix timestamp. The sequence can be any numbers, but a unix timestamp is a convenient way to force new stuff always higher since time monotonically increases.

```sql
SELECT pin_id FROM board_has_pins
WHERE board_id=241294561224164665 ORDER BY sequence
LIMIT 50 OFFSET 150
```


