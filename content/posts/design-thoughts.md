---
title: "Some Design Thoughts"
date: 2020-02-10T16:37:31+08:00
lastmod: 2020-02-10T16:37:31+08:00
draft: false
show_in_homepage: true
description_as_summary: true
license: ""

tags: [
    "design",
    "system design",
    "sql",
    "nosql",
    "object storage",
    "file storage",
]

comment: true
toc: true
auto_collapse_toc: true
---

# SQL or NoSQL

## SQL Tables vs NoSQL Documents

SQL databases store data records in tables.

NoSQL databases store JSON-like field-value pair documents. Similar documents can be stored in a collection, which is analogous to an SQL table.

## SQL Schema vs NoSQL Schemaless

## SQL Normalization vs NoSQL Denormalization

SQL databases use foreign key to relate one table to another, which minimizes data redundancy.

We can use normalization in NoSQL, but this is not always practical. We may opt to denormalize our document and repeat, for example, publisher information for every book.

## SQL Relational JOIN vs NoSQL

NoSQL has no equivalent of JOIN. If we used normalized collections, we would need to fetch all `book` documents, retrieve all associated `publisher` documents, and manually link the two in our program logic. **This is one reason denormalization is often essential.**

## SQL vs NoSQL Data Integrity

## SQL vs NoSQL Transactions

In SQL databases, two or more updates can be executed in a transaction - an all-or-nothing wrapper that guarantees success or failure.

In a NoSQL database, modification of a single document is atomic. However, there's no transaction equivalent for updates to multiple documents.

## SQL vs NoSQL Performance

NoSQL's simpler denormalized store allows you to retrieve all information about a specific item in a single request.

## SQL vs NoSQL Scaling

Scaling can be tricky for SQL databases. Lots of things need to consider. How do you allocate related data?

Many NoSQL databases have been built with scaling functionality from the start.

## Summary

Projects where SQL is ideal:
- Logical related discrete data requirements which can be identified up-front
- Data integrity is essential

Projects where NoSQL is ideal:
- Unrelated, indeterminate or evolving data requirements
- Simpler or looser project objectives, able to start coding immediately
- Speed and scalability is imperative


# Object stores or file servers

File storage presents itself as a file system hierarchy with directories, sub-directories and files. It is great and works beautifully when the number of files is not very large. It also works well when you know exactly where your files are stored.

Object storage, on the other hand, typically presents itself via a RESTful API. There is no concept of a file system. Instead, an application would save a object (files + additional metadata) to the object store via the PUT API and the object storage would save the object somewhere in the system. The object storage platform would give the application a unique key (analogous to a valet ticket) for that object which the application would store in the application database. If an application wanted to fetch that object, all they would need to do is give the key as part of the GET API and the object would be fetched by the object storage.
