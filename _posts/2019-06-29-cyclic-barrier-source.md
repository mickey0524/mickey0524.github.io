---
layout:     post
title:      "CyclicBarrier Source"
subtitle:   "CyclicBarrier 源码分析"
date:       2019-06-29 15:30:00
author:     "Mickey"
header-img: "img/post-bg-rwd.jpg"
tags:
    - java
---

这篇 blog 我们来讲一下 CyclicBarrier 的源码，CyclicBarrier 依赖 ReentrantLock 实现功能，感兴趣的同学可以先看看 [ReentrantLock 源码分析](https://mickey0524.github.io/2019/06/28/reentrantlock-source/)

CyclicBarrier 用于多个线程的互相等待，当多个线程都做好了准备后，一同开始运行，可以用于系统的初始化，当多个子线程都申请好了资源之后，系统启动

## 属性

```java
// CyclicBarrier 可以重复使用，使用 Generation 代表一代
private static class Generation {
    boolean broken = false;
}

// 使用 ReentrantLock 来维持同步
private final ReentrantLock lock = new ReentrantLock();
// 使用 Condition 在线程执行 await 的时候挂起，全部线程执行完毕后，通过 signalAll 唤醒挂起的线程 
private final Condition trip = lock.newCondition();
// CyclicBarrier 需要等待多少个 await
private final int parties;
// CyclicBarrier 触发的时候，执行的回调（先于线程执行）
private final Runnable barrierCommand;

private Generation generation = new Generation();
// 当前 CyclicBarrier 还有多少个线程没执行 await
private int count;
```

## 构造函数

```java
public CyclicBarrier(int parties, Runnable barrierAction) {
    if (parties <= 0) throw new IllegalArgumentException();
    this.parties = parties;
    this.count = parties;
    this.barrierCommand = barrierAction;
}

public CyclicBarrier(int parties) {
    this(parties, null);
}
```

构造函数非常简单，对应设置属性

## await

```java
public int await() throws InterruptedException, BrokenBarrierException {
    try {
        return dowait(false, 0L);
    } catch (TimeoutException toe) {
        throw new Error(toe); // cannot happen
    }
}
```

await 方法调用了 dowait 方法

## dowait

```java
private int dowait(boolean timed, long nanos)
    throws InterruptedException, BrokenBarrierException,
           TimeoutException {
    // 加锁
    final ReentrantLock lock = this.lock;
    lock.lock();
    try {
        final Generation g = generation;

        if (g.broken)
            throw new BrokenBarrierException();

        if (Thread.interrupted()) {
            breakBarrier();
            throw new InterruptedException();
        }
        // 减少等待线程的数量
        int index = --count;
        // 当 index 等于 0 的时候，触发所有的线程，这里没有用 < 0 防御用户传入 0 是因为构造函数传入 <= 0 的时候就会抛出异常
        if (index == 0) {
            boolean ranAction = false;
            try {
                // 如果设置了 barrierAction，优先执行
                final Runnable command = barrierCommand;
                if (command != null)
                    command.run();
                ranAction = true;
                // 调用 nextGeneration 方法，执行 trip.signAll 触发所有挂起的线程，开始下一代
                nextGeneration();
                return 0;
            } finally {
                if (!ranAction)
                    breakBarrier();
            }
        }

        // 自旋挂起，等待触发
        for (;;) {
            try {
                if (!timed)
                    trip.await();
                else if (nanos > 0L)
                    nanos = trip.awaitNanos(nanos);
            } catch (InterruptedException ie) {
                if (g == generation && ! g.broken) {
                    breakBarrier();
                    throw ie;
                } else {
                    Thread.currentThread().interrupt();
                }
            }

            if (g.broken)
                throw new BrokenBarrierException();

            if (g != generation)
                return index;

            if (timed && nanos <= 0L) {
                breakBarrier();
                throw new TimeoutException();
            }
        }
    } finally {
        lock.unlock();
    }
}

private void nextGeneration() {
    // 触发所有线程，完成这一代
    trip.signalAll();
    // 重置 count
    count = parties;
    // 开始下一代
    generation = new Generation();
}
```

## reset 方法

```java
public void reset() {
    final ReentrantLock lock = this.lock;
    lock.lock();
    try {
        breakBarrier();   // break the current generation
        nextGeneration(); // start a new generation
    } finally {
        lock.unlock();
    }
}
```

reset 方法直接 break 本代，然后开始新的一代

## 总结

这篇文章我们介绍了一下 CyclicBarrier，希望对大家有所帮助