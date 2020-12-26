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
本文在统一虚拟内存架构下，针对随机访问负载（如图处理应用）优化了GPU内存的page fault机制。
##  背景和问题：

<center>

<img src="../images/batch-aware-图1.png" width="65%" height="65%" />

图 1  统一虚拟内存机制（右）
</center>

GPU已经成为服务器和数据中心中重要的计算平台之一。传统的分离式GPU只能访问其私有设备内存，可访问内存容量严重受限，既增加了软件开发的局限性（限制软件内存占用），也为处理内存占用超过几百GB的大规模应用增加了挑战（需要精细地设计数据和算法分区）和成本（仅仅为了增加GPU内存容量而购买更多的GPU）。

为了解决访问内存受限的问题，新型GPU支持统一虚拟内存（UVM）技术，UVM为CPU和GPU提供了统一的虚拟内存地址空间，数据能够通过请求分页（demand paging）机制自动地在CPU或GPU节点之间迁移，从而使得GPU能访问CPU内存。

**问题在于**，GPU在UVM下使用demand paging依赖于缺页异常（page fault），而缺页异常的延时非常高（20-50us）。
此外，GPU会比CPU更频繁地访问内存，进一步放大demand paging的性能影响。

### 相关工作：
发表在ASPLOS 2019上的一篇文章[A Framework for Memory Oversubscription Management in Graphics Processing Units](https://dl.acm.org/doi/10.1145/3297858.3304044)提出ETC框架，这个框架能够通过预测访问模式来选择使用哪种策略组合从而降低GPU内存的demand paging带来的开销。但是对于大规模随机的负载，ETC表现不佳：第一，ETC严重依赖于准确预测负载访问模式提前执行page eviction，但是对于随机负载，无法作答准确地预测。第二，ETC其中的一个机制memory-aware throttling旨在通过关闭一部分GPU核以减小负载的的工作集。然而实验表明，对于许多大规模随机负载，GPU核使用数量与应用的工作集大小没有直接关系。这是因为存在大量的GPU核内存共享。

因此本文针对大规模随机负载对GPU的demand paging机制进行优化。


## 研究动机（motivation）：

GPU为了摊销PCIe总线传输和异常处理系统调用的开销，每次处理多个page fault，即批处理。如图2所示，由于GPU CUDA架构存在很高的并行度，同时会有上百货上千个线程在运行，因此某一个时间段内会积累很多page fault请求。图中A,B,C，D为四批次page fault，page fault的处理时串行化的，page fault的处理分为预处理，查询内存页表（page table walk）和页迁移（page migration）：预处理过程需要给page fault排序、插入page prefething请求以及调度page evivtion；在执行完page table walk完成后，开始page migration。只有完成了A批次的page fault处理，才会执行后续的B，C，D。由于在处理A批次时，连续生成了B,C两个批次的page fault，当A处理完后，发现有B,C两个批次，GPU会同时处理所有累积的page fault。本文通过初步实验获得了一下重要结论：

1. 本文将page fault批处理划分为page fault handling和page migration两个阶段，第一个阶段的时间占比较大。



<center>

<img src="../images/batch-aware-paging.png" width="85%" height="65%" />

图 2  观察1：GPU以批处理的方式同时处理多个page fault
</center>

2. 如图2所示，实验发现，大多数批处理较小，且单次批处理越大，page fault的平均处理延时越低。


<center>

<img src="../images/batch-aware-图2.png" width="65%" height="65%" />

图 3  观察2：批处理越多，平均处理延时越低
</center>

1. 观察3：Page eviction位于关键路径。如图4所示，为了避免页A覆写页X，只有当X被逐出后才迁移A的数据到GPU内存。
   
  <center>

<img src="../images/batch-aware-eviction.png" width="65%" height="65%" />

图 4  观察3：page eviction位于关键路径。
</center> 

## 设计： 

**设计思想：**
- Fault handling：通过执行上下文切换，使不同批次的page fault能够被交错地完成，增加批处理大小（一批次处理的page fault数量），有效地降低GPU平均page fault处理时间。

- Page eviction：提前完成page eviction，从而将page eviction移除出page miration的关键路径。



设计：
1. Thread Oversubscription (TO)
   
要增加批处理的大小就是要增加同时执行的线程数，一种直观的方法是为GPU SM分配更多地[thread block](https://en.wikipedia.org/wiki/Thread_block_(CUDA_programming))，但是一个SM能同时运行的thread block数受物理资源的限制。因此本文基于VT（virtual thread）技术，为每个SM分配尽可能多的thread block（超过物理调度资源限制，如程序计数器和SIMT stack），并将thread block划分为活跃和挂起两种状态，通过上下文切换，保证任何一个时刻正在运行的thrad block不会超过物理调度资源限制。

如图5所示，在GPU开始运行时就为每个SM分配尽可能多的thread block（①），额外添加的thread block（图中inactive）一开始处于挂起状态，当active thread block中的所有warp（GPU基本执行单元）都因为page fault而被挂起后，GPU通过上下文切换执行inactive thread block（②）。
考虑到TO机制会增加page fault产生频率，造成过早的page eviction。为了避免这一负面影响，GPU runtime会统计page eviction rate，一旦rate降低到某个阈值没那么就会阻止上下文切换（③）。

图6说明了TO机制能够增加批处理大小的原理。假设物理资源限制一个SM同时只能执行一个thread block（TB），页A,B,C由TB1访问，D由TB2访问。当TB1的所有warp都被挂起后（即访问页C产生page fault后），通过上下文切换执行TB2，因此B,C,D的apge fault可以一起被处理，即增加批处理大小。



<center>

<img src="../images/batch-aware-TO.png" width="65%" height="65%" />

图 5  thread oversubscription机制
</center> 




  <center>

<img src="../images/batch-aware-batch.png" width="95%" height="65%" />

图 6  thread oversubscription增加批处理大小原理
</center> 


2. Unobtrusive Eviction (UE)
为了降低批处理第二阶段（page migration）的延时，对页分配和页迁移的时序进行了修改。
本文修改了GPU runtime并增加了一个GPU 内存状态追踪器，如图7所示，当GPU MMU产生了一个page fault时，top-half ISR进行响应（①），top-half ISR通过检查GPU内存状态追踪器，判断内存使用是否达到上限，如果达到上限，则发出提前逐出请求（②③），最后执行bottom-half ISR（④）。由于批处理过程第一阶段的延时是几十微秒级别，第一个内存页的逐出在页迁移开始之前已经完成。如果后续还需要逐出内存页，bottom-half ISR会调度后续的page eviction和page migration。

图10展示了page eviction的时序图，当开始批处理page fault时，如果内存容量到达上限，就开始一个4KB页的page eviction（①），在页X被逐出后，更新页表（②），因此页A的迁移无需page eviction开销（③），页Y的逐出可以通过DMA双向传输与页A的迁移同时完成。

 <center>

<img src="../images/batch-aware-图7.png" width="65%" height="65%" />

图 7 unobtrusive eviction执行逻辑
</center> 



  <center>

<img src="../images/batch-aware-neweviction.png" width="65%" height="65%" />

图 8 unobtrusive eviction时序
</center> 

## 实验：

实验平台：如下图所示，修改MacSim仿真器以仿真虚拟内存（TLB，页表，Page table walker），demand paging和VT。

<center>

<img src="../images/batch-aware-evaluation.png" width="65%" height="65%" />

图 9  仿真平台配置
</center>