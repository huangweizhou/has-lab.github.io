---
layout: post
title: Elastic Cuckoo Page Tables Rethinking Virtual Memory Translation for Parallelism
author: Weizhou Huang
tags:
 - ASPLOS
 - 2020
 - Virtual memory
---

# [Elastic Cuckoo Page Tables: Rethinking Virtual Memory Translation for Parallelism](http://skarlat2.web.engr.illinois.edu/publications/cuckoo_asplos20.pdf)
注：本文仅对问题进行撰写，设计部分请自行阅读原文

## 背景和问题：
新兴的内存密集型应用对内存的需求空前增加。这导致内存地址转换开销也随之增加。

页表结构对内存地址转换的开销影响巨大,这正因为：当TLB未命中时，需要查询内存页表，而这一过程位于内存访问的关键路径。

当前页表结构是基于radix树的设计，每次内存地址转换时需要遍历多级内存页表，产生多次内存访问，从而造成较高的开销。而这个开销还有可能进一步增加：1）TLB受限于访问延迟、空间和能耗；2）内存规模会增加到TB甚至PB级别，3）各种各样的内存密集型负载不断出现。

radix树结构的页表是针对低内存容量的系统设计的，而不再适用于大内存系统，并且基于指针的顺序的访问操作导致其无法充分利用内存级并行。

背景知识：
cuckoo hashing是一个冲突解决算法，允许一个元素拥有多个可能的hash位置（通过同时多个hash fuction实现）。一个元素同时只会出现在其中一个hash位置，但是可以在不同的位置之间迁移。如果

## 挑战：
基于hash的页表存在的局限性：

- hash collision : 产生sequestial memory reference

- hash resize : 开销高

- 基于hash的页表无法被多个进程共享或支持多种映射尺寸

- 缺乏访问的局部性

- 每个PTE需要关联一个虚拟内存页号，相比radix page table占用更多地内存空间
  
后两个挑战已经有研究证明通过重新设计PTE可以解决，然而前三个挑战有待解决。
##  设计：


利用elastic cuckoo hashing的特性解决上述前三个挑战

##  实验：

