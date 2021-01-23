---
layout: post
title: 利用应用感知的内存通道分区降低多核系统的内存干扰
author: Liu Shi
tags:
    - Micro
    - 2011
    - Multicore
---

[Reducing Memory Interference in Multicore Systems via Application-Aware Memory Channel Partitioning](https://people.inf.ethz.ch/omutlu/pub/memory-channel-partitioning-micro11.pdf)

## 背景和问题

不同应用的内存请求间互相干扰，这样会导致系统性能下降，但是以前的工作认为这是内存访问调度研究的问题。根据以往知识，作者指出，1) 地址总线和数据总线是独立访问的，可以并行执行，2) 在进行内存页转换时，应用特征可以用cache miss和row buffer局部性表征。

以memory-intensity和row-buffer hit ratio作为应用特征衡量，作者做了两组实验，示意图如下，以阐述应用间干扰问题。
1）memory-intensity
![image](/images/2020-11-19-memory-intensity.png)
2）row-buffer hit ratio
![image](/images/2020-11-19-row-buffer-hit.png)

## 相关工作：
现有研究将将应用间的内存干扰视为内存请求的调度问题，因此主要针对内存控制器的调度逻辑进行大幅度修改。本文通过数据放置策略，将数据（内存页）放在不同的channel中以实现应用间的隔离，而避免了对内存控制器的修改。

## 设计
### 设计思想：
基于内存硬件的固有并行性（channel,rank,chip,bank等）进行硬件隔离/分区，并在内存带宽利用率（即单个应用能获得的并行性）和隔离程度之间找到平衡点。

作者提出了两个设计，内存通道分区机制，和将此机制与调度器结合。

- 注：本文对数据分区的最小粒度是内存页（4KB），并假设一个内存页对应内存硬件中的一个ROW。
### 1）内存通道分区
按照memory-intensity和row-buffer hit ratio对应用进行分组，在分组内，按照正比于应用数分配通道。


识别方法：


### 2）调度
识别内存请求稀疏的应用，在内存调度器中优先服务这些应用下发的请求，并且允许这些应用的页映射到任何内存通道上，因为这类应用对其他应用的干扰很小，为其分配专门的内存通道反而会浪费内存固有并行度和带宽。

识别方法：


