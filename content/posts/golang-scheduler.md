---
title: "Golang Scheduler"
date: 2020-02-15T15:06:55+08:00
lastmod: 2020-02-15T15:07:06+08:00
draft: false
show_in_homepage: true
description_as_summary: true
license: ""

tags: [
    "golang",
    "scheduler",
    "schedule",
]

comment: true
toc: true
auto_collapse_toc: true
---

## What does the Go runtime need with a scheduler?

Why create a userspace scheduler when the OS can schedule threads for you?

The POSIX thread API is very much **a logical extension to the existing Unix process model** and as such, **threads get a lot of the same controls as processes**. Threads have:
- their own signal mask, 
- can be assigned CPU affinity, 
- can be put into cgroups and 
- can be queried for which resources they use.