---
layout:     post
title:      "ReentrantReadWriteLock Source"
subtitle:   "ReentrantReadWriteLock 源码分析"
date:       2019-06-30 16:00:00
author:     "Mickey"
header-img: "img/post-bg-js-module.jpg"
tags:
    - java
---

[ReentrantReadWriteLock 源码分析(基于Java 8)](https://www.jianshu.com/p/6923c126e762)

👆这篇 blog 讲解的非常清楚，推荐给大家，这里我自己记录一下个人的感悟

1. ReentrantReadWriteLock 创建了 ReadLock 类和 WriteLock 类
2. 读锁和写锁是互斥的，写锁不会抢占读锁
3. 当线程获取写锁的时候，是可以继续获取读锁的，这是锁降级，反之则不行，会造成死锁
4. 线程如果没有申请到读锁和写锁，会挂起，线程均放置在 AQS 的双向链表上，使用 isShared 方法区分
5. ReentrantReadWriteLock 的 Sync 类使用 AQS 的 state 的高 16 位记录获取读锁的线程数，使用低 16 位记录获取写锁的线程数（可能会有重入），tryAcquire 和 tryAcquireShared 方法通过获取高低位代表的 short 值来判断是否能直接执行，还是需要挂起
6. 由于读锁可以被多个线程获取，每个获得读锁的线程都能重入，因此需要一个记录线程读锁重入数量的地方，Sync 类的 HoldCounter 类就是用来做这个的，然后使用 ThreadLocal 来包裹 HoldCounter
7. 为了优化读锁释放，Sync 有一个 cachedHoldCounter 变量，用来记录最后一次获取 readLock 的 HoldCounter 的缓存，因为很多场景下，这次进行release readLock 的线程就是上次 acquire 的线程, 这样直接通过cachedHoldCounter 来进行获取, 节省了通过 readHolds 的 lookup 的过程
8. firstReader 和 firstReaderHoldCount 用来记录第一次获取读锁的线程以及重入数量，也是为了优化，准确的说是第一次获取 readLock 并且 没有 release 的线程, 一旦线程进行 release readLock, 则 firstReader 会被置位 null
9. ThreadLocal 中使用了弱引用，因此如果 HoldCounter.count 为 0 的时候，需要手动 remove 避免内存泄漏 