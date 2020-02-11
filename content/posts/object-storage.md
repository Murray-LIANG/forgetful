---
title: "Object Storage"
date: 2020-02-10T16:37:31+08:00
lastmod: 2020-02-10T16:37:31+08:00
draft: false
show_in_homepage: true
description_as_summary: true
license: ""

tags: [
    "object storage",
    "block storage",
    "file storage",
]

comment: true
toc: true
auto_collapse_toc: true
---

## Object Storage vs Block vs File
块设备相对于操作系统来说就是block device，裸硬盘，提供最基本的IO Read、IO Write等操作，操作系统将block device映射到`/dev`目录，一般应用并不会直接访问块设备，一般先创建分区，然后格式化成文件系统，最后挂载到某个本地目录mount point来使用。

而文件系统存储，一般实现了Posix接口标准，直接提供文件系统的接口，如open，read，write，close，mkdir，walk，chown，chmod等等。文件是通过树的形式组织的，具有明显的层次结构。文件的访问需要从根目录一直到文件所在的位置。

而对象存储，通过HTTP RESTful的接口如GET，POST，PUT，DELETE来管理一些非结构化的数据，比如图片，文档。它没有目录的概念。

## 对象存储设备

从存储进化方向来看：Block >>> NAS >>> Object Storage，使得客户端越来越瘦，在Block时代，每个客户端需要使用存储，需要自己实现文件管理的功能，后来把这些功能移到存储服务器上便有了NAS。现如今，把对象管理的功能移到存储服务器上便有了Object Storage。

- 数据存储
- 智能分布
- 每个对象的元数据管理。对象元数据与filesystem中的inode类似，只不过是存放这对象的大小、数据块等信息。

## 元数据服务器

控制着整个对象存储的元数据，例如对象的分布情况，每个对象在那个存储设备上，等。

## 对象存储读访问流程
1. 客户端发出读请求。
2. 元数据服务器给出对象所在的存储设备。
3. 客户端直接向该设备读取对象。
4. 存储设备通过对象ID计算出对象所在的file或者block，读取出数据返回给客户端。
5. 客户端接收到存储设备返回的数据，可以将对象跟存储设备的对应关系cache下来。

## 对象存储写访问流程
1. 客户端发出写请求。
2. 元数据服务器根据当前各个存储设备的情况计算出，或者直接使用一致性hash算出对象应该存放在哪个存储设备。
3. 客户端直接向该设备写入对象。
4. 存储设备存放对象并将对象ID，所在的file或者block，等元数据记录下来。
5. 客户端成功接收存储设备返回的信息，并记录在cache中。
