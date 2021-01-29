---
title: Golang三色标记、混合写屏障GC模式图文全分析
date: 2021-01-28 11:05:00
tags: [golang, gc, 面试题]
cover: https://cdn.jsdelivr.net/gh/bob-zou/bob-zou.github.io/themes/diaspora/source/img/cover.jpg
---
> 垃圾回收(Garbage Collection，简称GC)是编程语言中提供的自动的内存管理机制，自动释放不需要的对象，让出存储器资源，无需程序员手动执行。
> 
> Golang中的垃圾回收主要应用三色标记法，GC过程和其他用户goroutine可并发运行，但需要一定时间的STW(stop the world)，STW的过程中，CPU不执行用户代码，全部用于垃圾回收，这个过程的影响很大，Golang进行了多次的迭代优化来解决这个问题。

## 内容提纲
 - G0 V1.3之前的标记-清除(mark and sweep)算法
 - Go V1.3之前的标记-清扫(mark and sweep)的缺点
 - Go V1.5的三色并发标记法
 - Go V1.5的三色标记为什么需要STW
 - Go V1.5的三色标记为什么需要屏障机制(“强-弱” 三色不变式、插入屏障、删除屏障 )
 - Go V1.8混合写屏障机制
 - Go V1.8混合写屏障机制的全场景分析

## Go V1.3之前的标记-清除(mark and sweep)算法
此算法主要有两个主要的步骤：
 - 标记(Mark phase)
 - 清除(Sweep phase)

**第一步** 暂停程序业务逻辑
操作非常简单，但是有一点需要额外注意：mark and sweep算法在执行的时候，需要程序暂停，即`STW(stop the world)`。 
也就是说，这段时间程序会卡在那儿。
![setp-1](mas-1.png)

**第二步** 开始标记，程序找出它所有可达的对象，并做上标记
![setp-2](mas-2.png)

**第三步** 标记完了之后，然后开始清除未标记的对象
![setp-3](mas-3.png)

**第四步** 停止暂停，让程序继续跑。
然后循环重复这个过程，直到process程序生命周期结束。

## 标记-清扫(mark and sweep)的缺点
 - `STW(stop the world)` 让程序暂停，程序出现卡顿 (`重要问题`)。
 - 标记需要扫描整个heap
 - 清除数据会产生heap碎片

Go V1.3版本之前就是以上来实施的, 流程是
![before v1.3](mas-4.png)

Go V1.3 做了简单的优化,将STW提前, 减少STW暂停的时间范围
![v1.3](mas-5.png)

**这里面最重要的问题就是：mark-and-sweep 算法会暂停整个程序**

Go是如何面对并这个问题的呢？接下来Go V1.5版本 就用`三色并发标记法`来优化这个问题.

## Go V1.5的三色并发标记法
三色标记法 实际上就是通过三个阶段的标记来确定清楚的对象都有哪些. 我们来看一下具体的过程.

## 没有STW的三色标记法

## 屏障机制

## Go V1.8的混合写屏障(hybrid write barrier)机制

## 总结
以上便是Golang的GC全部的标记-清除逻辑及场景演示全过程。

GoV1.3- 普通标记清除法，整体过程需要启动STW，效率极低。
GoV1.5- 三色标记法， 堆空间启动写屏障，栈空间不启动，全部扫描之后，需要重新扫描一次栈(需要STW)，效率普通
GoV1.8-三色标记法，混合写屏障机制， 栈空间不启动，堆空间启动。整个过程几乎不需要STW，效率较高。

参考文献:
 - https://www.cnblogs.com/wangyiyang/p/12191591.html
 - https://www.jianshu.com/p/eb6b3aff9ca5
 - https://zhuanlan.zhihu.com/p/74853110

**原文作者:** [刘丹冰 Aceld](https://segmentfault.com/u/aceld)
**原文地址:** [Golang三色标记、混合写屏障GC模式图文全分析](https://segmentfault.com/a/1190000022030353)
