---
title: "How to Approach a System Design Interview Question"
date: 2020-02-10T16:37:31+08:00
lastmod: 2020-02-10T16:37:31+08:00
draft: false
show_in_homepage: true
description_as_summary: true
license: ""

tags: [
    "design",
    "system design",
]

comment: true
toc: true
auto_collapse_toc: true
---

The system design interview is an open-ended conversation. You are expected to lead it.

## Step 1: Outline use cases, constraints, and assumptions.

Gather requirements and scope the problem. Ask questions to clarify use cases and constraints. Discuss assumptions.

Question examples:
- Who is going to use it?
- How are they going to use it?
- How many users are there?
- What does the system do?
- What are the inputs and outputs of the system?
- How much data do we expect to handle?
- How many requests per second do we expect?
- What is the expected read and write ratio?

## Step 2: Create a high level design

Outline a high level design with all important components.
- Sketch the main components and connections
- Justify your ideas

## Step 3: Design core components

Dive into details for each core component. For example, if you were asked to design a url shortening system, discuss:
- Generating and storing a hash of the full url
  - MD5 and Base62
  - Hash collisions
  - SQL or NoSQL
  - Database schema
- Translating a hashed url to the full url
  - Database lookup
- API and object-oriented design

## Step 4: Scale the Design

Identify and address bottlenecks, given the constraints. For example, do you need the following to address scalability issues?
- Load balancer
- Horizontal scaling
- Caching
- Database sharding
