---
title: "Pacemaker Tutorial"
date: 2021-11-19T18:15:34+08:00
lastmod: 2021-11-19T18:15:42+08:00
draft: false
show_in_homepage: true
description_as_summary: true
license: ""

tags: [
    "packmaker",
    "ha",
    "cluster",
    "corosync",
]

comment: true
toc: true
auto_collapse_toc: true
---

# Pacemaker Tutorial

## MISC CRM Commands

```console
$ # You can set a node to standby mode, 
$ # which can be used to simulate a node becoming unavailable,
$ # with this command:
$ sudo crm node standby NodeName
 
$ # You can change a node’s status from standby 
$ # to online with this command:

$ sudo crm node online NodeName
```