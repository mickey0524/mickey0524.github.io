---
layout:     post
title:      "CompletableFuture Source"
subtitle:   "CompletableFuture 源码分析"
date:       2019-08-04 16:30:00
author:     "Mickey"
header-img: "img/post-bg-rwd.jpg"
tags:
    - java
---

本来想自己写一下的，发现已经有别的同学总结过了

[JUC源码解析 CompletableFuture](https://www.cnblogs.com/aniao/p/aniao_cf.html)

我认为，这个类，掌握了 Completion 的链接，然后对应 complete 和 whenCompleteAsync 两个函数单步调试即可