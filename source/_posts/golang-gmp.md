---
title: Golang调度器GMP原理与调度全分析
date: 2021-01-29 17:22:45
tags: [golang, goroutine, gmp, 面试题]
cover: https://cdn.jsdelivr.net/gh/bob-zou/bob-zou.github.io/source/covers/golang-gmp.png
---
> 该文章主要详细具体的介绍Goroutine调度器过程及原理，可以对Go调度器的详细调度过程有一个清晰的理解。

## 内容提纲
- Golang调度器的由来
- Goroutine调度器的GMP模型及设计思想
- Goroutine调度场景过程全图文解析


**原文作者:** [刘丹冰 Aceld](https://segmentfault.com/u/aceld)
**原文地址:** [Golang三色标记、混合写屏障GC模式图文全分析](https://segmentfault.com/a/1190000021951119)
