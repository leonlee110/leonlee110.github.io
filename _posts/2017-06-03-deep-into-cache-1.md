---
title: 深入理解cache（一）
layout: page
categories: HardwareArchitecture
tags: cache
---

cache是万金油。无论是硬件设计，模块开发，还是架构设计，cache都是一个解决性能问题，加速业务性能的、简单直接的方案。为了更深入的理解cache，需要对cache的基本结构，cache的分析思想都有深入的理解。本文也就是对最近的学习做一些总结。

<!-- excerpt -->

## 1. 为什么要有cache

cache又成为缓存，主要是用来缓存数据。但是为什么需要缓存数据，这就得从当代计算机结构简单说起。计算机操作主要由指令和数据两部分组成，指令就是执行的操作，数据就是处理的对象，其中执行操作主要是CPU中的执行单元来完成，而数据则是存储在磁盘或者内存中。随着工艺的不断提升，CPU几乎每年都会更新一次，性能也随之得到突飞猛进的增长，但是存储单元受限于技术和成本的原因，则是发展相对缓慢（最近NVMe、XPoint等技术也在加快存储方面的改善）。计算和存储之前的性能差距越来越大，不论是吞吐还是延迟。

简单来讲，现在应该集中精力加快存储性能，特别是内存。除了技术上的原因，一个更重要的是成本，快速的存储介质成本极高，但是能否在性能和成本之前做个折中呢？cache应运而生。性能比内存好，但成本比内存高，因此容量比内存小（得多）。

## 2. cache的外部层次

在现代计算机体系结构中，一般使用三层cache来缓存数据。其中离core最近的L1 cache，分为缓存指令的Instruction Cache和缓存数据的Data Cache，这一级cache是每个物理core都有自己专有的。更大，但是延迟更高的L2 cache，则是指令和数据共存的，同时也是每个物理core都专有。最大的L3 cache则是CPU内的每个core共享的。分成多级也如前面介绍，从性能和成本折中的角度考虑。下图为
![cache_latency](/assets/cache/cache_latency.png){:class="img-responsive"}

通过lscpu即可了解系统的cache结构，如下图所示：
```
[root@localhost sys-utils]# ./lscpu
Architecture:          x86_64
CPU op-mode(s):        32-bit, 64-bit
CPU(s):                40
Thread(s) per core:    2
Core(s) per socket:    10
CPU socket(s):         2
NUMA node(s):          2
Vendor ID:             GenuineIntel
CPU family:            6
Model:                 63
Stepping:              2
CPU MHz:               2301.000
Virtualization:        VT-x
L1d cache:             32K
L1i cache:             32K
L2 cache:              256K
L3 cache:              25600K
NUMA node0 CPU(s):     0-9,20-29
NUMA node1 CPU(s):     10-19,30-39

```
通过sys文件系统中的文件也可以了解相关信息（实际上lscpu即是读取解读相关信息）：
```
[root@localhost sys-utils]# sudo cat /sys/devices/system/cpu/cpu0/cache/index0/size
32K
[root@localhost sys-utils]# sudo cat /sys/devices/system/cpu/cpu0/cache/index1/size
32K
[root@localhost sys-utils]# sudo cat /sys/devices/system/cpu/cpu0/cache/index2/size
256K
[root@localhost sys-utils]# sudo cat /sys/devices/system/cpu/cpu0/cache/index3/size
25600K
```

## 3. cache的内部结构

cache被组织成cache line来访问，并且cache line的大小一般和数据总线一致，cache line也是cache操作的基本单元。即使我们访问的是一个byte，也会缓存一整个cache line，这样一是从空间局部性的角度来说其他byte被后续访问的概率极高，缓存整个cache line能减少后续内存操作的次数，二是为了更充分的利用数据总线。

前面也介绍cache相比内存容量来说要小得多，那怎么合理利用这些有限的cache缓存更多的数据呢？在软件开发中，遇到这种问题一个通用的解决方案就是hash，通过hash将内存地址对应到其可以缓存的cache地址，同时借助某种算法来解决冲突问题。cache line的进一步组织结构类似。cache line被组织成set，每个set具有类似的hash特征，然后set中又分成多个way，来缓存不同的数据，类似于hash中数组来解决冲突。另外通过tag arry的辅助来确定同一set中不同的数据对应在哪一way的cache line中。
如下图：
![cache_arctech](/assets/cache/cache_architec.png){:class="img-responsive"}

根据way数的不同，cache被组织成不同的模式：“Direct mapped cache"（N=1），"N-way set associative cache"，"Fully associative cache"（SETNUM=1）。经过简单的立即即可以发现，N越小，在统一set中遍历比较各tag的时间越少，定位到具体cache line越快，但是由于没有“冲突解决”的机制，cache miss的几率越大；而N越大，在统一set中定位的时间则会越长，但是冲突的几率越小。所以一般选择的是一种折中方案。在我的系统中是8 Ways。

老规矩，系统中怎样去查看？
```
[root@localhost ~]# cat /sys/devices/system/cpu/cpu0/cache/index0/coherency_line_size
64
[root@localhost ~]# cat /sys/devices/system/cpu/cpu0/cache/index0/number_of_sets
64
[root@localhost ~]# cat /sys/devices/system/cpu/cpu0/cache/index0/size
32K
[root@localhost ~]# cat /sys/devices/system/cpu/cpu0/cache/index0/ways_of_associativity
8
```

另外，有人专门针对这些做了相关试验，说明在开发中合理利用这些概念的必要性：[Gallery of Processor Cache Effects](http://igoro.com/archive/gallery-of-processor-cache-effects/)。

## 4. cache操作

在了解了上面基本概念后，为了更好的[profile performance of cache]()，还需要了解cache是怎样操作的，从而才能了解哪些地方可能促发cache性能问题，可能的解决思路是怎样的。

首先从层次结构来看，分析简单的inclusive模式，CPU首先访问的是L1 cache，如果数据（或者指令）不在，则促发miss event，进而访问L2 cache以至L3 cache。而对于Exclusive模式的cache，则有不同的访问模式，后续专门总结cache相关操作。由于各cache性能有极大差异，所以越快的cache命中率越高性能则约好，反而越差。这里即引出了一个关键指标cache miss。有不少围绕miss rate的研究，一个比较有意思的研究是：小容量cache是没用的，特别是L2[clearing the cloud](http://infoscience.epfl.ch/record/168849/files/clouds_techreport11.pdf)。

出于多个原因考虑，应用程序是通过虚拟地址来访问cache和内存的，而为了执行实际的物理操作，虚拟地址到物理地址的转换是必不可少的。为了加速这个操作，在硬件设计上引入了TLB来加速这种转换，因此TLB的利用效率对性能有极大影响。在获得物理地址后，即可访问内存了。TLB miss则进入评估的视野。

进入到cache内部，如前面介绍的cache组织结构，首先会通过地址中特定的位，来确定cache可能所在的set，然后在tag array中的每个tag比较相应的位，如果匹配成功则hit，如果没有任何tag配比则miss。确定好数据对应的way后，在通过剩余的offset位确定访问的数据在cache line中具体的位置，进而访问数据。所以如前面所说，way数的选择是一个对整体性能有影响的因素。

另外，Instruction Cache是一个特殊的部件，所有执行的指令都需要从此cache中获取，为了加速这种操作，引入了一个predict器件来预测需要的指令，这当然也是影响性能的部件之一。Predict Failuer Rate则可以看到预测失败对性能的影响。

## 5. PMU
PMU是CPU提供的统计硬件性能的组件，通过对各种event的统计，从而计算出各硬件操作的性能指标。在2.6.31以上的内核中增加了perf_event系统来获取这些性能指标信息，或者称之为微架构信息。一个简单的结构图如下，
![perf_architec](/assets/cache/perf_architec.png){:class="img-responsive"}

如图所示，Linux已经在内核中完成了相关支持，对于我们来说，使用应用层的perf工具即能利用PMU的功能，完成相关数据的抓去与分析。常见的perf命令以及功能有：
```
perf stat：直接统计PMU event的统计信息，例如本文想要深入研究的cache_miss等
perf top：类似于top操作，统计程序各函数指定event的信息，并试试刷新
perf record/report：获取程序非常详细的统计信息，并处理
perf list：列出所在平台支持的event
```

有cache相关的event主要是Cache-references、Cache-misses，通过特性的寄存器也可以采集到cache的开销clock。
