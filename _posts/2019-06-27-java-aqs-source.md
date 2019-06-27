---
layout:     post
title:      "AQS Source"
subtitle:   "AQS 源码分析"
date:       2019-06-27 14:30:00
author:     "Mickey"
header-img: "img/post-bg-e2e-ux.jpg"
tags:
    - java
---

这篇 blog 我们来讲一下 AQS 的源码，我们知道 JUC 包提供了许多并发相关的类，这些类底层都依赖了 AQS

AQS 全称是 AbstractQueuedSynchronizer，顾名思义，是一个用来构建锁和同步器的类，AQS 底层用了 CAS 技术来保证操作的原子性，同时使用 FIFO 的双向队列实现线程间的锁竞争

## AQS 属性

```java
private transient volatile Node head;

private transient volatile Node tail;

private volatile int state;

protected final boolean compareAndSetState(int expect, int update) {
    return unsafe.compareAndSwapInt(this, stateOffset, expect, update);
}
```

1. head 字段为双向队列的头节点，表示当前正在执行的节点
2. tail 字段为双向队列的尾节点
3. state 字段为同步字段，其中 state > 0 为有锁状态，每次加锁就在原有 state 基础上加 1，即代表当前持有锁的线程加了 state 次锁，反之解锁时每次减 1，当 state = 0 为无锁状态
4. 通过 compareAndSetState 方法操作 CAS 更改 state 状态，保证 state 的原子性

有没有发现，这几个字段都用 volatile 关键字进行修饰，以确保多线程间保证字段的可见性

## AQS 需要子类实现的方法

AQS 大量使用了模版设计模式，需要继承 AQS 的子类实现模版内部使用的方法

```java
// 获取独占锁方法
protected boolean tryAcquire(int arg) {
  throw new UnsupportedOperationException();
}
// 释放独占锁方法
protected boolean tryRelease(int arg) {
  throw new UnsupportedOperationException();
}
// 获取共享锁方法
protected int tryAcquireShared(int arg) {
  throw new UnsupportedOperationException();
}
// 释放共享锁方法
protected boolean tryReleaseShared(int arg) {
  throw new UnsupportedOperationException();
}
```

## Node 类

我们知道 AQS 内部维护了一个双向链表，Node 为链表中的元素，在 AQS 中 Node 是用来包装一个等待的线程的

```java
static final class Node {
    // SHARED 指代 Node 中包裹的线程申请的是共享锁
    static final Node SHARED = new Node();
    // SHARED 指代 Node 中包裹的线程申请的是排他锁
    static final Node EXCLUSIVE = null;
    // CANCELLED 指 Node 已经取消了
    static final int CANCELLED =  1;
    // SIGNAL 指 Node 等待触发
    static final int SIGNAL    = -1;
    // CONDITION 指 Node 等待 condition 的 signal/signalAll 方法触发
    static final int CONDITION = -2;
    // PROPAGATE 指共享锁的 Node 传递触发状态给链表中后续的节点
    static final int PROPAGATE = -3;
    // waitStatus 指当前 Node 节点的状态
    volatile int waitStatus;
    
    // Node 在链表中的前序节点
    volatile Node prev;
    // Node 在链表中的后继节点
    volatile Node next;
    // thread 为 Node 包裹的线程
    volatile Thread thread;

    Node nextWaiter;

    final boolean isShared() {
        return nextWaiter == SHARED;
    }

    final Node predecessor() throws NullPointerException {
        Node p = prev;
        if (p == null)
            throw new NullPointerException();
        else
            return p;
    }

    Node() {}

    Node(Thread thread, Node mode) {
        this.nextWaiter = mode;
        this.thread = thread;
    }

    Node(Thread thread, int waitStatus) {
        this.waitStatus = waitStatus;
        this.thread = thread;
    }
}
```

## 独占锁

独占锁的原理是如果有线程获取到锁，那么其它线程只能是获取锁失败，然后进入等待队列中等待被唤醒

### acquire 方法

```java
public final void acquire(int arg) {
  if (!tryAcquire(arg) &&
      acquireQueued(addWaiter(Node.EXCLUSIVE), arg))
    selfInterrupt();
}
```

1. acquire 方法首先调用子类实现的 tryAcquire 方法，如果返回 true，说明当前线程获取了锁
2. 当 tryAcquire 方法返回 false 的时候，执行 `addWaiter(Node.EXCLUSIVE)` 方法将当前线程封装成一个 Node 节点对象
3. 把当前线程执行封装成 Node 节点后，继续执行 acquireQueued 的逻辑，该逻辑主要是判断当前节点的前置节点是否是头节点，来尝试获取锁，如果获取锁成功，则当前节点就会成为新的头节点，这也是获取锁的核心逻辑

### addWaiter

```java
private Node addWaiter(Node mode) {
    Node node = new Node(Thread.currentThread(), mode);
    Node pred = tail;
    if (pred != null) {
        node.prev = pred;
        if (compareAndSetTail(pred, node)) {
            pred.next = node;
            return node;
        }
    }
    enq(node);
    return node;
}
```

addWaiter 方法创建一个新的 Node 实例，当 AQS 的 tail 变量不为 null 的时候，会使用 CAS 尝试更新一下 tail，如果 tail 为 null 或者 CAS 失败会调用 enq 方法

### enq

```java
private Node enq(final Node node) {
    for (;;) {
        Node t = tail;
        if (t == null) { // Must initialize
            if (compareAndSetHead(new Node()))
                tail = head;
        } else {
            node.prev = t;
            if (compareAndSetTail(t, node)) {
                t.next = node;
                return t;
            }
        }
    }
}
```

enq 是一个自旋操作，当 node 被加入双向链表后，才会 return，这里需要注意，如果 tail 为 null 的话，enq 方法会创建一个空的 Node 实例，也就是初始化的时候，head 节点就是一个空的 Node 实例

### acquireQueued

```java
final boolean acquireQueued(final Node node, int arg) {
    boolean failed = true;
    try {
        // 线程中断标记字段
        boolean interrupted = false;
        for (;;) {
            // 获取当前节点的 prev 节点
            final Node p = node.predecessor();
            // 如果 prev 节点是 head，再次尝试获取锁
            if (p == head && tryAcquire(arg)) {
                // 获取成功的话，当前节点就成为了 head
                setHead(node);
                p.next = null; // help GC
                failed = false;
                return interrupted;
            }
            // 获取锁失败，进入挂起逻辑
            if (shouldParkAfterFailedAcquire(p, node) &&
                parkAndCheckInterrupt())
                interrupted = true;
        }
    } finally {
        if (failed)
            cancelAcquire(node);
    }
}
```

在 Node 被加入双向链表之后，进入了 acquireQueued 方法，acquireQueued 主要做了以下事情：

1. 判断当前节点的 pred 节点是否为 head 节点，如果是，尝试获取锁
2. 获取锁失败之后，进入挂起逻辑，等待触发

我们👆说过，head 节点是双向链表头部的节点，持有锁，因此，如果当前节点的 pred 节点是 head 节点，很可能此时 head 节点已经释放锁了，所以此时需要再次尝试获取锁

### shouldParkAfterFailedAcquire

```java
private static boolean shouldParkAfterFailedAcquire(Node pred, Node node) {
    int ws = pred.waitStatus;
    if (ws == Node.SIGNAL)
        return true;
    if (ws > 0) {
        do {
            node.prev = pred = pred.prev;
        } while (pred.waitStatus > 0);
        pred.next = node;
    } else {
        compareAndSetWaitStatus(pred, ws, Node.SIGNAL);
    }
    return false;
}
```

1. 如果 prev 节点的状态为 SIGNAL，说明 prev 节点也在等待触发，那么 node 肯定也需要挂起，直接返回 true
2. 如果 prev 节点大于 0，说明 prev 节点被 cancel 了，处于 CANCELLED 状态，将 prev 设置为 prev 之前的状态不为 CANCELLED 的节点
3. 如果程序走到最后一个 else，那么 prev 节点的状态为 0 或者 PROPAGATE，我们直接将 prev 的状态设置为 SIGNAL

### parkAndCheckInterrupt

```java
private final boolean parkAndCheckInterrupt() {
    LockSupport.park(this);
    return Thread.interrupted();
}
```

parkAndCheckInterrupt 方法调用 `LockSupport.park(this)` 方法挂起，等待执行

### release

```java
public final boolean release(int arg) {
    if (tryRelease(arg)) {
        Node h = head;
        if (h != null && h.waitStatus != 0)
            unparkSuccessor(h);
        return true;
    }
    return false;
}
```

release 方法执行 AQS 子类实现的 tryRelease 方法，返回 true 的话，调用 unparkSuccessor 唤醒 head 的下一个节点，这里和之前的 enq 也有呼应，enq 中会将一个空的 Node 作为 head

### unparkSuccessor

```java
private void unparkSuccessor(Node node) {
    int ws = node.waitStatus;
    if (ws < 0)
        compareAndSetWaitStatus(node, ws, 0);

    Node s = node.next;
    if (s == null || s.waitStatus > 0) {
        s = null;
        for (Node t = tail; t != null && t != node; t = t.prev)
            if (t.waitStatus <= 0)
                s = t;
    }
    if (s != null)
        LockSupport.unpark(s.thread);
}
```

unparkSuccessor 方法获取 node 的后继节点中第一个状态不为 CANCELLED 的节点，然后调用 `LockSupport.unpark(s.thread)` 唤醒之前 parkAndCheckInterrupt 方法挂载的线程

## 共享锁
