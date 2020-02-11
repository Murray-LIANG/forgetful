---
title: "RAID"
date: 2020-02-10T16:37:31+08:00
lastmod: 2020-02-10T16:37:31+08:00
draft: false
show_in_homepage: true
description_as_summary: true
license: ""

tags: [
    "raid",
    "storage",
]

comment: true
toc: true
auto_collapse_toc: true
---

# RAID

http://oserror.com/backend/raid-principle/

## RAID的一般有如下作用

- 数据冗余： 指把数据的校验信息存放在冗余的磁盘中，在某些磁盘数据损坏时，能从其他未损坏的磁盘中，重新构建数据。
- 性能提升： 指RAID能把多块独立的磁盘组成磁盘阵列，通过把数据切成分片的方式，使得读/写数据能走多块磁盘，从而提升性能。

