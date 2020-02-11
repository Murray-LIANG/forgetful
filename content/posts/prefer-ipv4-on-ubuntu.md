---
title: "Prefer IPv4 on Ubuntu"
date: 2020-02-10T16:37:31+08:00
lastmod: 2020-02-10T16:37:31+08:00
draft: false
show_in_homepage: true
description_as_summary: true
license: ""

tags: [
    "ipv4",
    "ubuntu",
]

comment: true
toc: true
auto_collapse_toc: true
---

Edit `/etc/gai.conf` and locate the line and uncomment it:
```
precedence ::ffff:0:0/96  100
```