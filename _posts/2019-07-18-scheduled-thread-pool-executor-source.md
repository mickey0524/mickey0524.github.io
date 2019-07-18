---
layout:     post
title:      "ScheduledThreadPoolExecutor Source"
subtitle:   "ScheduledThreadPoolExecutor 源码分析"
date:       2019-07-18 15:30:00
author:     "Mickey"
header-img: "img/post-bg-rwd.jpg"
tags:
    - java
---

ScheduledThreadPoolExecutor 可以用于注册任务，这些任务会在将来的某些阶段被调度触发（这篇博客只讲主要的思路）

ScheduledThreadPoolExecutor 继承于 ThreadPoolExecutor，对 ThreadPoolExecutor 感兴趣的可以移步[ThreadPoolExecutor 源码分析](https://mickey0524.github.io/2019/06/26/java-thread-pool-executor-source/)，ScheduledThreadPoolExecutor 使用 DelayedWorkQueue 作为 BlockingQueue，DelayedWorkQueue 中存放的元素是 ScheduledFutureTask

DelayedWorkQueue 其实就是一个堆，按照 ScheduledFutureTask 的触发时间排序

## schedule

schedule 方法会在 delay 时间之后，执行 command 的 run 方法，首先，生成一个 ScheduledFutureTask 实栗，然后调用 delayedExecute 方法将 ScheduledFutureTask 实例加入 DelayedWorkQueue 的堆，ensurePrestart 方法会在线程池中新启一个 Worker 线程

```java
public ScheduledFuture<?> schedule(Runnable command,
                                    long delay,
                                    TimeUnit unit) {
    if (command == null || unit == null)
        throw new NullPointerException();
    RunnableScheduledFuture<?> t = decorateTask(command,
        new ScheduledFutureTask<Void>(command, null,
                                        triggerTime(delay, unit)));
    delayedExecute(t);
    return t;
}

private void delayedExecute(RunnableScheduledFuture<?> task) {
    if (isShutdown())
        reject(task);
    else {
        super.getQueue().add(task);
        if (isShutdown() &&
            !canRunInCurrentRunState(task.isPeriodic()) &&
            remove(task))
            task.cancel(false);
        else
            ensurePrestart();
    }
}

void ensurePrestart() {
    int wc = workerCountOf(ctl.get());
    if (wc < corePoolSize)
        addWorker(null, true);
    else if (wc == 0)
        addWorker(null, false);
}
```

## ScheduledFutureTask

ScheduledFutureTask 是 DelayedWorkQueue 中存放的实栗，继承于 FutureTask，对 FutureTask 不熟悉的同学可以移步[FutureTask 源码分析](https://mickey0524.github.io/2019/07/01/future-task-source/)，我们知道 ThreadPoolExecutor 的 Worker 线程会从 BlockingQueue 中获取 task，调用 task 的 run 方法，这里 ScheduledFutureTask 的 run 方法会调用 FutureTask 的 run 方法，调用 Runnable 参数的 run 方法

```java
private class ScheduledFutureTask<V>
            extends FutureTask<V> implements RunnableScheduledFuture<V> {
    // Task 被触发的时间
    private long time;
    
    ScheduledFutureTask(Runnable r, V result, long ns) {
        super(r, result);
        this.time = ns;
    }

    public void run() {
        boolean periodic = isPeriodic();
        if (!canRunInCurrentRunState(periodic))
            cancel(false);
        else if (!periodic)
            ScheduledFutureTask.super.run();
        else if (ScheduledFutureTask.super.runAndReset()) {
            setNextRunTime();
            reExecutePeriodic(outerTask);
        }
    }
}
```

## DelayedWorkQueue 的 take 方法

在 ThreadPoolExecutor 的 getTask 方法中，会调用 BlockingQueue 的 take/poll 方法，如下所示，可以看到，take 方法获取堆中下标为 0 的 ScheduledFutureTask 实例，判断是否到了触发的时间，没有则调用 `available.await()` 进行等待，到达了则执行 finishPoll 方法，将 ScheduledFutureTask 实例出堆并返回

```java
public RunnableScheduledFuture<?> take() throws InterruptedException {
    final ReentrantLock lock = this.lock;
    lock.lockInterruptibly();
    try {
        for (;;) {
            RunnableScheduledFuture<?> first = queue[0];
            if (first == null)
                available.await();
            else {
                long delay = first.getDelay(NANOSECONDS);
                if (delay <= 0)
                    return finishPoll(first);
                first = null; // don't retain ref while waiting
                if (leader != null)
                    available.await();
                else {
                    Thread thisThread = Thread.currentThread();
                    leader = thisThread;
                    try {
                        available.awaitNanos(delay);
                    } finally {
                        if (leader == thisThread)
                            leader = null;
                    }
                }
            }
        }
    } finally {
        if (leader == null && queue[0] != null)
            available.signal();
        lock.unlock();
    }
}

private RunnableScheduledFuture<?> finishPoll(RunnableScheduledFuture<?> f) {
    int s = --size;
    RunnableScheduledFuture<?> x = queue[s];
    queue[s] = null;
    if (s != 0)
        siftDown(0, x);
    setIndex(f, -1);
    return f;
}
```