---
layout: post
title: Durable Transactional Memory Can Scale with TimeStone
author: Weizhou Huang
tags:
 - Asplos
 - 2020
 - Trasactional memory
---

# [Durable Transactional Memory Can Scale with TimeStone](https://dl.acm.org/doi/abs/10.1145/3373376.3378483)

##背景和问题：

持久化内存技术（如Intel Optane）具有传统内存的特性（如字节可寻址能力和第访问延迟）的同时还提供了持久性和大内存容量。这使得软件能够完全驻留在内存并且保证数据掉电不丢失。同时，持久化内存技术给系统开发人员带来了新的挑战，比如需要保证数据的崩溃一致性、性能的可扩展性，最小写放大等。

现有针对数据崩溃一致性的研究可以分为两类：

一类是并发持久化数据结构，另一类是

持久化内存有望很快成为数据中心多核服务器的一部分，因此在设计基于持久化内存的软件时，多核可扩展性是一个重要的衡量指标。


**问题在于，现有机制无法在不影响多核可扩展性和性能的前提下实现崩溃一致性。**
现有研究在保证崩溃一致性时需要在关键路径上使用高延迟的日志机制，而现代无序处理器重排高速缓存行进一步增加了崩溃一致性机制设计的复杂度。

