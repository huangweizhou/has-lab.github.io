---
layout: post
title: Triad-NVM: Persistency for Integrity-Protected and Encrypted Non-Volatile Memories
author: Weizhou Huang
tags:
 - ISCA
 - 2019
 - Hybrid memory
 - Security
---

# [Triad-NVM: Persistency for Integrity-Protected and Encrypted Non-Volatile Memories](https://ieeexplore.ieee.org/stamp/stamp.jsp?tp=&arnumber=8980320)

本文研究了在内存加密和完整性验证的系统中如何设计持久化机制。

## 背景与问题：

非易失性内存（non-volatile main memory，NVMM）具有掉电数据不丢失的特性，同时也产生了在上电阶段（boot episode）数据被窥探或篡改的安全隐患。

挑战在于：如何同时保留安全保证，崩溃/重启后的数据恢复，良好的持久性性能，和快速恢复的能力。现有研究通常将内存安全与持久化分开来研究，而忽视了相互之间的影响。

## 设计：




## 实验：
GEM5(fs)， PMDK

## 新颖性： 
在混合内存架构下，结合了安全性和持久化提出了新的问题，并通过实验数据educate读者。

