---
layout:     post
title:      "FutureTask Source"
subtitle:   "FutureTask 源码分析"
date:       2019-07-01 16:00:00
author:     "Mickey"
header-img: "img/post-bg-alitrip.jpg"
tags:
    - java
---

这篇 blog 我们来讲一下 FutureTask 的源码，FutureTask 是 java 实现异步编程的基础

```java
public class FutureTask<V> implements RunnableFuture<V>

...

public interface RunnableFuture<V> extends Runnable, Future<V> {
    void run();
}
```

从⬆️我们能够看到，FutureTask 实现了 Runnable 接口，因此 FutureTask 实例能够被提交到线程池上执行

FutureTask 的运行方式：

1. 将一个 Callable 置为 FutureTask 的内置成员
2. 执行 Callable 中的 call 方法
3. 调用 get() 或者 get(timeout, TimeUnit) 方法，获取 call 方法返回的结果

下面我们一个一个来看看

## 内部属性

```java
private volatile int state;
private static final int NEW          = 0;
private static final int COMPLETING   = 1;
private static final int NORMAL       = 2;
private static final int EXCEPTIONAL  = 3;
private static final int CANCELLED    = 4;
private static final int INTERRUPTING = 5;
private static final int INTERRUPTED  = 6;
```

FutureTask 使用 state 变量表示当前的执行状态

* NEW：任务处于新建状态
* COMPLETING：任务正在完成中
* NORMAL：任务正常完成
* EXCEPTIONAL：call 方法执行过程中抛出了异常
* CANCELLED：调用 cancel 方法，取消任务
* INTERRUPTING：任务正在中断
* INTERRUPTED：thread.interrupt() 执行完毕

```java
private Callable<V> callable;
private Object outcome;
private volatile Thread runner;
private volatile WaitNode waiters;
```

* callable：创建 FutureTask 实栗的时候传入的，用来异步执行获取结果
* outcome：执行 callable 的 call 方法返回的结果，可能是结果或者执行过程中抛出的异常
* runner：执行 FutureTask run 方法的线程
* waiters：WaitNode 组成的单向链表，用于存储调用 get() 或者 get(timeout, TimeUnit) 方法的线程

## 构造函数

```java
public FutureTask(Callable<V> callable) {
    if (callable == null)
        throw new NullPointerException();
    this.callable = callable;
    this.state = NEW;
}
```

## run 方法

```java
public void run() {
    // CAS 设置 runner
    if (state != NEW ||
        !UNSAFE.compareAndSwapObject(this, runnerOffset,
                                     null, Thread.currentThread()))
        return;
    try {
        Callable<V> c = callable;
        if (c != null && state == NEW) {
            V result;
            boolean ran;
            try {
                // 执行 callable 的 call 方法
                result = c.call();
                ran = true;
            } catch (Throwable ex) {
                result = null;
                ran = false;
                // 执行中抛异常, 更新 state 状态, 释放等待的线程(调用 finishCompletion)
                setException(ex);
            }
            if (ran)
                // 执行成功, 进行赋值操作，释放等待的线程(调用 finishCompletion)

                set(result);
        }
    } finally {
        int s = state;
        if (s >= INTERRUPTING)
            handlePossibleCancellationInterrupt(s);
    }
}
```

## set

```java
protected void set(V v) {
    // CAS 更新 state 为 COMPLETING
    if (UNSAFE.compareAndSwapInt(this, stateOffset, NEW, COMPLETING)) {
        outcome = v; // 设置结果
        UNSAFE.putOrderedInt(this, stateOffset, NORMAL);
        // 唤醒等待的线程
        finishCompletion();
    }
}
```

## finishCompletion

遍历链表，唤醒等待的线程

```java
static final class WaitNode {
    volatile Thread thread;
    volatile WaitNode next;
    WaitNode() { thread = Thread.currentThread(); }
}

private void finishCompletion() {
    for (WaitNode q; (q = waiters) != null;) {
        // CAS 将链表头设置为 null
        if (UNSAFE.compareAndSwapObject(this, waitersOffset, q, null)) {
            // 遍历链表
            for (;;) {
                Thread t = q.thread;
                if (t != null) {
                		// 帮助 GC
                    q.thread = null;
                    // 唤醒等待的线程
                    LockSupport.unpark(t);
                }
                WaitNode next = q.next;
                if (next == null)
                    break;
                // 帮助 GC
                q.next = null;
                q = next;
            }
            break;
        }
    }

    done();

    callable = null;
}
```

## get

```java
public V get() throws InterruptedException, ExecutionException {
    int s = state;
    // 当 FutureTask 没有执行完毕，调用 awaitDone 进入 链表等待
    if (s <= COMPLETING)
        s = awaitDone(false, 0L);
    return report(s);
}

public V get(long timeout, TimeUnit unit)
    throws InterruptedException, ExecutionException, TimeoutException {
    if (unit == null)
        throw new NullPointerException();
    int s = state;
    if (s <= COMPLETING &&
        (s = awaitDone(true, unit.toNanos(timeout))) <= COMPLETING)
        throw new TimeoutException();
    return report(s);
}
```

## report

```java
private V report(int s) throws ExecutionException {
    Object x = outcome;
    // 当状态为正常的时候，转型返回
    if (s == NORMAL)
        return (V)x;
    // 失败的时候抛出异常
    if (s >= CANCELLED)
        throw new CancellationException();
    throw new ExecutionException((Throwable)x);
}
```

## awaitDone

```java
private int awaitDone(boolean timed, long nanos)
    throws InterruptedException {
    // 计算等待的最长时间
    final long deadline = timed ? System.nanoTime() + nanos : 0L;
    WaitNode q = null;
    boolean queued = false;
    for (;;) {
        // 线程中断的时候，调用 removeWaiter 从链表中删除节点
        if (Thread.interrupted()) {
            removeWaiter(q);
            throw new InterruptedException();
        }

        int s = state;
        // 执行完毕，直接返回状态
        if (s > COMPLETING) {
            if (q != null)
                q.thread = null;
            return s;
        }
        // 当 state 为 COMPLETING，状态马上会更改，先 yield 一下
        else if (s == COMPLETING) // cannot time out yet
            Thread.yield();
        // 初始化一个 WaitNode
        else if (q == null)
            q = new WaitNode();
        // CAS 头插法，设置 WaitNode 头部节点
        else if (!queued)
            queued = UNSAFE.compareAndSwapObject(this, waitersOffset,
                                                 q.next = waiters, q);
        else if (timed) {
            nanos = deadline - System.nanoTime();
            // 挂起超时，执行 removeWaiter
            if (nanos <= 0L) {
                removeWaiter(q);
                return state;
            }
            // 限时挂起
            LockSupport.parkNanos(this, nanos);
        }
        else
            // 挂起
            LockSupport.park(this);
    }
}
```

## removeWaiter

```java
private void removeWaiter(WaitNode node) {
    if (node != null) {
        node.thread = null; // 将移除的节点的thread＝null, 为移除做标示
        retry:
        for (;;) {
            // q 为链表中的当前节点，pred 为 q 之前的节点，s 为 q 的后继节点
            for (WaitNode pred = null, q = waiters, s; q != null; q = s) {
                s = q.next;
                if (q.thread != null)
                    pred = q;
                // prev.next -> s
                else if (pred != null) {
                    pred.next = s;
                    if (pred.thread == null)
                        continue retry;
                }
                // 头部节点就是需要删除的
                else if (!UNSAFE.compareAndSwapObject(this, waitersOffset,
                                                      q, s))
                    continue retry;
            }
            break;
        }
    }
}

```

## 总结

1. 需要实现一个链表(每个节点包含当前线程的引用)
2. 通过 LockSupport.park 对线程进行阻塞
3. 有个公共方法进行节点的唤醒(task 完成, task 执行异常，线程 Interrupt, 或 await 超时), 并且次方法要线程安全

## FutureTask 例子

[FutureTask 在高并发环境下确保任务只执行一次](https://blog.csdn.net/chenliguan/article/details/54345993)