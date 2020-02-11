---
title: "Raid Group and Storage Group and Storage Pool"
date: 2020-02-10T16:37:31+08:00
lastmod: 2020-02-10T16:37:31+08:00
draft: false
show_in_homepage: true
description_as_summary: true
license: ""

tags: [
    "raid group",
    "storage group",
    "storage pool",
]

comment: true
toc: true
auto_collapse_toc: true
---

## Raid Group

将多个硬盘组合起来的一个集合，以实现更大容量、更快读写速度、更高冗余度等目的。

位于RAID Group之上的逻辑结构称为LUN。供主机使用。

## Storage Group

为了实现LUN Masking（LUN的安全屏蔽机制，即1.仅将LUN分配给特定的主机；2.阻止主机看到存储中所有的LUN），需要有一个容器来“存放”LUN与主机的关系，这个容器就是Storage Group。

先创建一个Storage Group，再连接进主机（Connect Hosts)，然后将LUN添加进这个Storage Group，主机就可以看到添加进去的LUN。

## Storage Pool

Pool为了实现存储虚拟化（Storage Virtualization）而诞生的。对CLARiiON来说，就是其引入的Virtual Provisioning功能。该功能可以让用户在Pool中创建Thin或者Thick LUN来分配存储资源，并且实现全自动存储分层（FAST）。

## Raid Group vs Storage Pool

Pool LUN并不能完全替代RAID Group LUN，如Hot Spare、Write Intent Log、Clone Private LUN、MirrorView、Clone必须要求RAID Group。

Storage Pool只是在RAID Group上做了一层抽象，底层依然是一个个的RAID Group。到Storage Pool的IO会被重定向到底层的RG，而这个重定向是通过查询抽象层所实现的表结构做到的。

单个传统RAID Group会受到16个磁盘的限制，而Storage Pool本身可含上百个的磁盘（很多个RAID Group）。

如果对性能要求严苛，并且需要物理上做到数据隔离的场合，则使用RAID Group。

https://www.dell.com/community/%E6%95%B0%E6%8D%AE%E5%AD%98%E5%82%A8%E8%AE%A8%E8%AE%BA%E5%8C%BA/%E8%AF%B7%E6%95%99Raid-Group-Storage-Pool-Storage-Group%E7%AD%89%E4%B8%80%E4%BA%9B%E6%A6%82%E5%BF%B5%E7%9A%84%E5%8C%BA%E5%88%AB/td-p/6806158