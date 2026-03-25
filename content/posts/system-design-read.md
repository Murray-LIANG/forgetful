---
title: "System Design Todo"
date: 2020-09-30T21:24:44+08:00
lastmod: 2020-09-30T21:24:55+08:00
draft: false
show_in_homepage: true
description_as_summary: true
license: ""

tags: [
    "system design",
    "design",
    "todo",
]

comment: true
toc: true
auto_collapse_toc: true
---

# SQL and NoSQL
## SQL
1. Do we need ACID transactions?
2. Do we need to run complex dynamic queries?
3. Do we need to use the storage for analytics or data warehousing?

## NoSQL
1. Do we need a database that can be easily scaled for both writes and reads?
2. Do we need a database with highly available and network partition tolerant?
3. Which types of NoSQL should be choose?
    1. Document - the size of the record we are storing.
    2. Graph - any relationship between what we are storing.
    3. Column
    4. Key-value

# Message Queue vs. Stream processing