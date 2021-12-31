---
title: "Interview Questions"
date: 2021-11-12T16:41:13+08:00
lastmod: 2021-11-12T16:41:20+08:00
draft: false
show_in_homepage: true
description_as_summary: true
license: ""

tags: [
    "interview questions",
]

comment: true
toc: true
auto_collapse_toc: true
---


- 自我介绍
- 简单介绍项目
- 经常使用的数据结构有哪些，他们有什么区别，在要用的时候会怎么选
- 看项目，使用了REST和gRPC，这两个都是API，在使用过程中有什么相同和不同，什么时候用
哪一个
- protobuffer和JSON的用处和不同
- 有没有贡献过社区项目，使用git的话，一般是个什么样的步骤
- 有没有接触过异步编程，简单说说
- 比较私人的问题，可以选择不回答，为什么从美国回国找工作


## 数据结构
- 数组与链表
- 排序，及其复杂度
   - 插入：稳定，平均/最差时间复杂度 O(n²)，元素基本有序时最好时间复杂度 O(n)，空间复杂度 O(1)。
   - 希尔：不稳定，平均时间复杂度 O(n^1.3^)，最差时间复杂度 O(n²)，最好时间复杂度 O(n)，空间复杂度 O(1)。
   - 堆：不稳定，时间复杂度 O(nlogn)，空间复杂度 O(1)。
   - 冒泡：稳定，平均/最坏时间复杂度 O(n²)，元素基本有序时最好时间复杂度 O(n)，空间复杂度 O(1)。
   - 快排：不稳定，平均/最好时间复杂度 O(nlogn)，元素基本有序时最坏时间复杂度 O(n²)，空间复杂度 O(logn)。
   - 合并：任何情况时间复杂度都为 O(nlogn)，空间复杂度为 O(n)。
- heap/堆如何进行插入和删除的
- 树，树的遍历

## Linux
[linux interview questions](linux-interview-questions.md)

## 计算机网络
https://www.eet-china.com/mp/a68780.html
- 计算机网络体系结构
- TCP vs UDP，TCP如何保证可靠性
- 