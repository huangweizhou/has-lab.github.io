---
layout: post
title: Batch-Aware Unified Memory Management in GPUs for Irregular Workloads
author: Weizhou Huang
tags:
 - Asplos
 - 2020
 - Virtual memory
---

# [Batch-Aware Unified Memory Management in GPUs for Irregular Workloads](https://ramyadhadidi.github.io/files/kim-asplos20.pdf)
本文在统一虚拟内存架构下，优化了GPU内存的page fault机制。
##  背景和问题：

<center>

<img src="../images/batch-aware-图1.png" width="65%" height="65%" />

图 1  统一虚拟内存机制（右）
</center>

GPU已经成为服务器和数据中心中重要的计算平台之一。传统的分离式GPU只能访问其私有设备内存，可访问内存容量严重受限，既增加了软件开发的局限性（限制软件内存占用），也为处理内存占用超过几百GB的大规模应用增加了挑战（需要精细地设计数据和算法分区）和成本（仅仅为了增加GPU内存容量而购买更多的GPU）。

为了解决访问内存受限的问题，新型GPU支持统一虚拟内存（UVM）技术，UVM为CPU和GPU提供了统一的虚拟内存地址空间，数据能够通过请求分页（demand paging）机制自动地在CPU或GPU节点之间迁移，从而使得GPU能访问CPU内存。

问题在于，GPU在UVM下使用demand paging依赖于缺页异常（page fault），而缺页异常的延时非常高（20-50us）。
此外，GPU会比CPU更频繁地访问内存，进一步放大demand paging的性能影响。