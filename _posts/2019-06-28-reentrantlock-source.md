---
layout:     post
title:      "ReentrantLock Source"
subtitle:   "ReentrantLock 源码分析"
date:       2019-06-28 18:00:00
author:     "Mickey"
header-img: "img/post-bg-universe.jpg"
tags:
    - java
---

这篇 blog 我们来讲一下 ReentrantLock 的源码，ReentrantLock 是一种依赖 AQS 的可重入排他锁，和 Semaphore 类型，ReentrantLock 也分为公平和非公平两种

## Sync

Sync 一个抽象类，继承了 AQS

```java
abstract static class Sync extends AbstractQueuedSynchronizer {

    abstract void lock();

    final boolean nonfairTryAcquire(int acquires) {
        final Thread current = Thread.currentThread();
        int c = getState();
        if (c == 0) {
            if (compareAndSetState(0, acquires)) {
                setExclusiveOwnerThread(current);
                return true;
            }
        }
        else if (current == getExclusiveOwnerThread()) {
            int nextc = c + acquires;
            if (nextc < 0) // overflow
                throw new Error("Maximum lock count exceeded");
            setState(nextc);
            return true;
        }
        return false;
    }

    protected final boolean tryRelease(int releases) {
        int c = getState() - releases;
        if (Thread.currentThread() != getExclusiveOwnerThread())
            throw new IllegalMonitorStateException();
        boolean free = false;
        if (c == 0) {
            free = true;
            setExclusiveOwnerThread(null);
        }
        setState(c);
        return free;
    }

    protected final boolean isHeldExclusively() {
        return getExclusiveOwnerThread() == Thread.currentThread();
    }

    final ConditionObject newCondition() {
        return new ConditionObject();
    }

    final Thread getOwner() {
        return getState() == 0 ? null : getExclusiveOwnerThread();
    }

    final int getHoldCount() {
        return isHeldExclusively() ? getState() : 0;
    }

    final boolean isLocked() {
        return getState() != 0;
    }

}
```

如👆所示，nonfairTryAcquire 方法非常清晰的解释了 ReentrantLock 是一个可重入锁，如果当前 state 为 0，说明没有线程持有锁，当前线程通过 CAS 获取锁，如果当前持有锁的线程就是当前的线程，则 state + 1

tryRelease 方法通过 CAS 将 state 减一，如果 state 为 0，说明没有线程持有锁了，否则，说明是之前持有锁的线程多次请求了锁，这里对应释放

## NonfairSync

```java
static final class NonfairSync extends Sync {

    final void lock() {
        if (compareAndSetState(0, 1))
            setExclusiveOwnerThread(Thread.currentThread());
        else
            acquire(1);
    }

    protected final boolean tryAcquire(int acquires) {
        return nonfairTryAcquire(acquires);
    }
}
```

NonfairSync 是非公平锁，tryAcquire 方法直接调用 Sync 的 nonfairTryAcquire 方法

## FairSync

```java
static final class FairSync extends Sync {

    final void lock() {
        acquire(1);
    }

    protected final boolean tryAcquire(int acquires) {
        final Thread current = Thread.currentThread();
        int c = getState();
        if (c == 0) {
            if (!hasQueuedPredecessors() &&
                compareAndSetState(0, acquires)) {
                setExclusiveOwnerThread(current);
                return true;
            }
        }
        else if (current == getExclusiveOwnerThread()) {
            int nextc = c + acquires;
            if (nextc < 0)
                throw new Error("Maximum lock count exceeded");
            setState(nextc);
            return true;
        }
        return false;
    }
}
```

FairSync 是公平锁，和 Semaphore 的公平锁类似，这里也使用了 hasQueuedPredecessors 函数

## 构造函数

```java
public ReentrantLock() {
    sync = new NonfairSync();
}

public ReentrantLock(boolean fair) {
    sync = fair ? new FairSync() : new NonfairSync();
}
```

构造函数初始化一个 Sync 类

## lock

```java
public void lock() {
    sync.lock();
}
```

lock 方法调用 Sync 对象的 lock 方法，非公平锁会先进行一次 CAS 尝试，公平锁，直接将线程加入 AQS 等待队列

## unlock

```java
public void unlock() {
    sync.release(1);
}
```

unlock 方法调用 release 方法，唤醒挂起的线程

## Condition

我们知道 ReentrantLock 的 Condition 是一个很大的特色，这个参照 [AQS](https://mickey0524.github.io/2019/06/27/java-aqs-source/) 文章的 ConditionObject 

## 总结

这篇文章讲解了 ReentrantLock，希望对大家有所帮助