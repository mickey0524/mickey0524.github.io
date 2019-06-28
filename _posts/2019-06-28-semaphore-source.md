---
layout:     post
title:      "Semaphore Source"
subtitle:   "Semaphore 源码分析"
date:       2019-06-28 16:00:00
author:     "Mickey"
header-img: "img/home-bg-o.jpg"
tags:
    - java
---

这篇 blog 我们来讲一下 Semaphore 的源码，Semaphore 允许指定数量的线程同时运行，Semaphore 同样使用 AQS 的共享锁实现

## Sync 类

```java
abstract static class Sync extends AbstractQueuedSynchronizer {
    Sync(int permits) {
        setState(permits);
    }

    final int getPermits() {
        return getState();
    }

    final int nonfairTryAcquireShared(int acquires) {
        for (;;) {
            int available = getState();
            int remaining = available - acquires;
            if (remaining < 0 ||
                compareAndSetState(available, remaining))
                return remaining;
        }
    }

    protected final boolean tryReleaseShared(int releases) {
        for (;;) {
            int current = getState();
            int next = current + releases;
            if (next < current) // overflow
                throw new Error("Maximum permit count exceeded");
            if (compareAndSetState(current, next))
                return true;
        }
    }
}

static final class NonfairSync extends Sync {
    NonfairSync(int permits) {
        super(permits);
    }

    protected int tryAcquireShared(int acquires) {
        return nonfairTryAcquireShared(acquires);
    }
}

static final class FairSync extends Sync {
    FairSync(int permits) {
        super(permits);
    }

    protected int tryAcquireShared(int acquires) {
        for (;;) {
            if (hasQueuedPredecessors())
                return -1;
            int available = getState();
            int remaining = available - acquires;
            if (remaining < 0 ||
                compareAndSetState(available, remaining))
                return remaining;
        }
    }
}
```

Sync 类继承了 AQS，FairSync 和 NonfairSync 继承了 Sync，分别用于公平的 Semaphore 和 非公平的 Semaphore，公平的 Semaphore 严格遵循 FIFO 原则

NonfairSync 类的 tryAcquireShared 调用了 Sync 的 nonfairTryAcquireShared 方法，用当前 state 的数值减去 acquire 的数值，如果 remaining 大于等于 0，说明线程可以直接执行，如果小于 0，进入双向链表等待

FairSync 类的 tryAcquireShared 使用 hasQueuedPredecessors 方法来严格保证 FIFO，hasQueuedPredecessors 返回当前双向链表是否存在等待的线程，如果不存在，则和 NonfairSync 逻辑一样，如果存在，当前线程挂起进入双向链表，等待 release 方法触发

Sync 的 tryReleaseShared 方法，更新 state 值，触发双向链表中挂起的线程

## 构造函数

```java
public Semaphore(int permits) {
    sync = new NonfairSync(permits);
}

public Semaphore(int permits, boolean fair) {
    sync = fair ? new FairSync(permits) : new NonfairSync(permits);
}
```

构造函数初始化一个 Sync 对象

## acquire

```java
public void acquire() throws InterruptedException {
    sync.acquireSharedInterruptibly(1);
}
```

acquire 方法直接调用 AQS 的 acquireSharedInterruptibly 进行排队

## release

```java
public void release() {
    sync.releaseShared(1);
}
```

release 方法调用 AQS 的 releaseShared 方法，当 tryReleaseShared 返回 true 的时候，唤醒挂起的线程

## 总结

这篇 blog 介绍了 Semaphore 的实现，希望对大家有所帮助

