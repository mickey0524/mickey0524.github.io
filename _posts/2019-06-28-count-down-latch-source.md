---
layout:     post
title:      "CountDownLatch Source"
subtitle:   "CountDownLatch 源码分析"
date:       2019-06-28 15:30:00
author:     "Mickey"
header-img: "img/post-bg-js-module.jpg"
tags:
    - java
---

这篇 blog 我们来讲一下 CountDownLatch 的源码，CountDownLatch 常用于一等多的场景，例如 shutdown 的时候，等待资源释放完毕之类的

CountDownLatch 依赖 AQS 的共享锁实现，因为多个线程可以同时调用 await，当等待的线程全部执行完毕后，调用 await 的线程需要同时被触发

## Sync 对象

```java
private static final class Sync extends AbstractQueuedSynchronizer {

    Sync(int count) {
        setState(count);
    }

    int getCount() {
        return getState();
    }

    protected int tryAcquireShared(int acquires) {
        return (getState() == 0) ? 1 : -1;
    }

    protected boolean tryReleaseShared(int releases) {
        for (;;) {
            int c = getState();
            if (c == 0)
                return false;
            int nextc = c-1;
            if (compareAndSetState(c, nextc))
                return nextc == 0;
        }
    }
}
```

Sync 类继承了 AQS，实现了 tryAcquireShared 和 tryReleaseShared 两个方法，可以看到只有 AQS 的 state 为 0 的时候，tryAcquireShared 才返回 1，线程才能获取锁，否则都需要排队，而 tryReleaseShared 则会对讲 state 减 1，当 state 为 0 的时候，返回 true，唤醒等待的线程

## 构造函数

```java
public CountDownLatch(int count) {
    if (count < 0) throw new IllegalArgumentException("count < 0");
    this.sync = new Sync(count);
}
```
构造函数传入 count，初始化一个 state 为 count 的 Sync 对象

## await 方法

```java
public void await() throws InterruptedException {
    sync.acquireSharedInterruptibly(1);
}
```

await 方法直接调用 AQS 的 acquireSharedInterruptibly 进行排队

## countDown 方法

```java
public void countDown() {
    sync.releaseShared(1);
}
```

countDown 方法调用 AQS 的 releaseShared 方法，当 tryReleaseShared 返回 true 的时候，唤醒挂起的线程

## 总结

这篇 blog 介绍了 CountDownLatch 的实现，希望对大家有所帮助

