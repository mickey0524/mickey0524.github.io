---
layout:     post
title:      "ThreadPoolExecutor Source"
subtitle:   "ThreadPoolExecutor 源码分析"
date:       2019-06-26 15:00:00
author:     "Mickey"
header-img: "img/post-bg-alitrip.jpg"
tags:
    - java
---

这篇 blog 来分析一下 ThreadPoolExecutor 的源码，阿里巴巴 Java 开发手册推荐使用 ThreadPoolExecutor 来创建线程池对象，而不推荐使用 Executors，其实 Executors 中的方法也是调用了 ThreadPoolExecutor，不过传入的参数容易导致 OOM

## 属性

```
private final AtomicInteger ctl = new AtomicInteger(ctlOf(RUNNING, 0));
// COUNT_BITS = 29
private static final int COUNT_BITS = Integer.SIZE - 3;
private static final int CAPACITY   = (1 << COUNT_BITS) - 1;

// runState is stored in the high-order bits
private static final int RUNNING    = -1 << COUNT_BITS;
private static final int SHUTDOWN   =  0 << COUNT_BITS;
private static final int STOP       =  1 << COUNT_BITS;
private static final int TIDYING    =  2 << COUNT_BITS;
private static final int TERMINATED =  3 << COUNT_BITS;

// Packing and unpacking ctl
private static int runStateOf(int c)     { return c & ~CAPACITY; }
private static int workerCountOf(int c)  { return c & CAPACITY; }
private static int ctlOf(int rs, int wc) { return rs | wc; }
```

ctl 是一个 32 位的 Integer，ctl 的高 3 位用于存储线程池当前的工作状态，后 29 位用于存储线程池中 worker 的数量，runStateOf 方法和 workerCountOf 方法分别用于从 ctl 中获取 state 和 workerCount

RUNNING，SHUTDOWN，STOP，TIDYING，TERMINATED 指代线程池的 5 种工作状态，可以看到，它们的二进制高三位分别为 111，000，001，010，011（第一位是符号位）

* RUNNING：线程池接受新的 task，同时也处理排队的 task
* SHUTDOWN：线程池不接受新的 task，但是会处理排队的 task
* STOP：线程池不接受新的 task，也不处理排队的 task，并且会中断进行中的 task
* TIDYING：所有的 task 都已经结束了，workerCount 等于 0，即将调用 terminated() 方法
* TERMINATED：terminated() 方法执行完毕

```java
// 存放排队的 task 的队列
private final BlockingQueue<Runnable> workQueue;
// 用来给读写 workers 加锁
private final ReentrantLock mainLock = new ReentrantLock();
// 存储 worker 的 set
private final HashSet<Worker> workers = new HashSet<Worker>();
// 当 terminated 执行完毕的时候，执行 termination.signAll()
private final Condition termination = mainLock.newCondition();
// 记录线程池中线程的最大数目
private int largestPoolSize;
// 记录线程池一共完成了多少 task
private long completedTaskCount;
// 提供 newThread 方法用来创建线程
private volatile ThreadFactory threadFactory;
// 当 task 无法被调用的时候执行
private volatile RejectedExecutionHandler handler;
// 线程空闲的时候最多等候多久
private volatile long keepAliveTime;
// false 的话，允许线程无限期等待，true 的话，最多等候 keepAliveTime
private volatile boolean allowCoreThreadTimeOut;
// 线程池中最少有几个 worker
private volatile int corePoolSize;
// 线程池中最多有几个 worker
private volatile int maximumPoolSize;
```

## 构造函数

👇是 ThreadPoolExecutor 的构造函数

```java
public ThreadPoolExecutor(int corePoolSize,
                          int maximumPoolSize,
                          long keepAliveTime,
                          TimeUnit unit,
                          BlockingQueue<Runnable> workQueue,
                          ThreadFactory threadFactory,
                          RejectedExecutionHandler handler) {
    if (corePoolSize < 0 ||
        maximumPoolSize <= 0 ||
        maximumPoolSize < corePoolSize ||
        keepAliveTime < 0)
        throw new IllegalArgumentException();
    if (workQueue == null || threadFactory == null || handler == null)
        throw new NullPointerException();
    this.acc = System.getSecurityManager() == null ?
            null :
            AccessController.getContext();
    this.corePoolSize = corePoolSize;
    this.maximumPoolSize = maximumPoolSize;
    this.workQueue = workQueue;
    this.keepAliveTime = unit.toNanos(keepAliveTime);
    this.threadFactory = threadFactory;
    this.handler = handler;
}
```

## Worker

Worker 是线程池中用来执行 task 的工具类，run 方法会调用 ThreadPoolExecutor 中的 runWorker 方法，会循环执行 queue 中的 task

```java
private final class Worker
    extends AbstractQueuedSynchronizer
    implements Runnable
{
    // Worker 工作的线程
    final Thread thread;
    // Worker 执行的第一个 task，可能为 null（队列中存在等待执行的 task）
    Runnable firstTask;
    // Worker 完成的 task 数量
    volatile long completedTasks;

    // 初始化构造函数
    Worker(Runnable firstTask) {
        setState(-1);
        this.firstTask = firstTask;
        // 调用线程工厂创建一个新的 thread
        this.thread = getThreadFactory().newThread(this);
    }

    // run 方法执行 runWorker 方法，runWorker 用来执行 task
    public void run() {
        runWorker(this);
    }

    // Lock 相关的方法
    // AQS 的 state 为 0 表示没有加锁的状态
    // 1 表示加锁的状态
    protected boolean isHeldExclusively() {
        return getState() != 0;
    }

    protected boolean tryAcquire(int unused) {
        if (compareAndSetState(0, 1)) {
            setExclusiveOwnerThread(Thread.currentThread());
            return true;
        }
        return false;
    }

    protected boolean tryRelease(int unused) {
        setExclusiveOwnerThread(null);
        setState(0);
        return true;
    }

    public void lock()        { acquire(1); }
    public boolean tryLock()  { return tryAcquire(1); }
    public void unlock()      { release(1); }
    public boolean isLocked() { return isHeldExclusively(); }

    void interruptIfStarted() {
        Thread t;
        if (getState() >= 0 && (t = thread) != null && !t.isInterrupted()) {
            try {
                t.interrupt();
            } catch (SecurityException ignore) {
            }
        }
    }
}
```

## execute

execute 方法用来将 Runnable 对象提交给线程池执行，我们结合代码来看看，execute 可以划分为 3 个步骤

1. 如果当前线程池中的 Worker 数量小于 corePoolSize，直接新建一个 Worker，然后将 command 作为 firstTask
2. 如果线程池处于运行状态，而且 task 成功排队进入 workQueue，进行二次检查，如果线程池关闭了且成功将 command 从 workQueue 中删除，我们执行 reject 方法，表示执行 task 失败，如果发现 Worker 数目为 0 的话，我们创建一个新的 Worker，但是不传入 firstTask（已经在 workQueue 中排队了）
3. 构造函数中传入了 corePoolSize 和 maximumPoolSize（指代线程池允许存在的最大 Worker 数），尝试创建一个新的 Worker（例如 ArrayBlockingQueue 队列满了），失败的话调用 reject 方法

```java
public void execute(Runnable command) {
    if (command == null)
        throw new NullPointerException();
 
    int c = ctl.get();
    if (workerCountOf(c) < corePoolSize) {
        if (addWorker(command, true))
            return;
        c = ctl.get();
    }
    if (isRunning(c) && workQueue.offer(command)) {
        int recheck = ctl.get();
        if (! isRunning(recheck) && remove(command))
            reject(command);
        else if (workerCountOf(recheck) == 0)
            addWorker(null, false);
    }
    else if (!addWorker(command, false))
        reject(command);
}
```

## addWorker

addWorker 方法用来创建一个新的 Worker 实例，firstTask 为 Worker 的第一个执行的 task，core 为 true 表明当前线程池中的 Worker 数量还没到 corePoolSize，为 false 说明超过了 corePoolSize，需要按照 maximumPoolSize 的基准去比较

```java
private boolean addWorker(Runnable firstTask, boolean core) {
    retry:
    for (;;) {
        int c = ctl.get();
        // 获取当前线程池的状态
        int rs = runStateOf(c);

        // 线程池关闭了，返回 false
        if (rs >= SHUTDOWN &&
            ! (rs == SHUTDOWN &&
               firstTask == null &&
               ! workQueue.isEmpty()))
            return false;

        for (;;) {
            int wc = workerCountOf(c);
            // 如果当前线程池中 Worker 的数量已经到达上限，返回 false
            if (wc >= CAPACITY ||
                wc >= (core ? corePoolSize : maximumPoolSize))
                return false;
            // Worker 数目 + 1，break 出循环
            if (compareAndIncrementWorkerCount(c))
                break retry;
            c = ctl.get();
            if (runStateOf(c) != rs)
                continue retry;
        }
    }
    // workerStarted 指代 Worker 是否执行了 start 方法
    boolean workerStarted = false;
    // workerAdded 指代 Worker 是否加入 workers 这个 set
    boolean workerAdded = false;
    Worker w = null;
    try {
        w = new Worker(firstTask);
        final Thread t = w.thread;
        if (t != null) {
            final ReentrantLock mainLock = this.mainLock;
            mainLock.lock();
            try {
                int rs = runStateOf(ctl.get());

                if (rs < SHUTDOWN ||
                    (rs == SHUTDOWN && firstTask == null)) {
                    // 如果 Worker 中的 thread 已经处于工作状态，抛出异常
                    if (t.isAlive())
                        throw new IllegalThreadStateException();
                    // 将新的 Worker 加入 workers
                    workers.add(w);
                    int s = workers.size();
                    if (s > largestPoolSize)
                        largestPoolSize = s;
                    workerAdded = true;
                }
            } finally {
                mainLock.unlock();
            }
            // 如果 Worker 加入 workers 这个 set，调用 t.start 方法，启动线程，同时将 workerStarted 设置为 true
            if (workerAdded) {
                t.start();
                workerStarted = true;
            }
        }
    } finally {
        if (! workerStarted)
            addWorkerFailed(w);
    }
    return workerStarted;
}
```

## runWorker

addWorker 中执行了 `t.start()` 方法来触发 thread 的执行，threadFactory 会将 Worker 作为 Runnable 参数传入 Thread 的构造函数，因此 `t.start()` 会调用 `Worker.run()` 方法，而 Worker 的 run 方法调用了 runWorker 方法，我们来看看源码（给出关键部分用来理解）

runWorker 方法使用一个 while 循环，持续处理 task，首先，runWorker 定义 task 为 Worker 的 firstTask，当处理完毕之后，task 重新设置为 null，然后 while 循环会调用 getTask 方法，获取新的 task，如果 getTask 返回 null，会执行 processWorkerExit 方法销毁 Worker（和 keepAliveTime 和 allowCoreThreadTimeOut 有关）

```java
final void runWorker(Worker w) {
    Thread wt = Thread.currentThread();
    Runnable task = w.firstTask;
    w.firstTask = null;
    w.unlock(); // allow interrupts
    boolean completedAbruptly = true;
    try {
        while (task != null || (task = getTask()) != null) {
            w.lock();

            if ((runStateAtLeast(ctl.get(), STOP) ||
                 (Thread.interrupted() &&
                  runStateAtLeast(ctl.get(), STOP))) &&
                !wt.isInterrupted())
                wt.interrupt();
            try {
                beforeExecute(wt, task);
                task.run();
            } finally {
                task = null;
                w.completedTasks++;
                w.unlock();
            }
        }
        completedAbruptly = false;
    } finally {
        processWorkerExit(w, completedAbruptly);
    }
}
``` 

## getTask

getTask 主要用于 Worker 从 workQueue 中获取待执行的 task

```java
private Runnable getTask() {
    // timedOut 变量指代上一个 poll 方法是否超时 
    boolean timedOut = false;

    for (;;) {
        int c = ctl.get();
        int rs = runStateOf(c);

        // 检查线程池是否关闭
        if (rs >= SHUTDOWN && (rs >= STOP || workQueue.isEmpty())) {
            decrementWorkerCount();
            return null;
        }

        int wc = workerCountOf(c);

        // allowCoreThreadTimeOut 为 true 或者当前线程池里的 Worker 数目大于最小数目
        boolean timed = allowCoreThreadTimeOut || wc > corePoolSize;

        if ((wc > maximumPoolSize || (timed && timedOut))
            && (wc > 1 || workQueue.isEmpty())) {
            if (compareAndDecrementWorkerCount(c))
                return null;
            continue;
        }

        try {
            // 如果 timed 为 true，说明 Worker 只等待 keepAliveTime 时间
            Runnable r = timed ?
                workQueue.poll(keepAliveTime, TimeUnit.NANOSECONDS) :
                workQueue.take();
            if (r != null)
                return r;
            // r 为 null，说明超时了，timedOut 设置为 true
            timedOut = true;
        } catch (InterruptedException retry) {
            timedOut = false;
        }
    }
}
```

## Executors.newCachedThreadPool 传入的 BlockingQueue 是 SynchronousQueue

[Java SynchronousQueue](https://blog.csdn.net/yanyan19880509/article/details/52562039) 

## RejectedExecutionHandler

JDK 提供了 4 种 RejectedExecutionHandler 接口的实现，它们都是以 ThreadPoolExecutor 类的静态内部类的形式定义的，它们的具体实现以及拒绝策略如下

* AbortPolicy （默认）

    抛出未检查的 RejectedExecutionException，调用者自己捕获处理

    ```java
    public static class AbortPolicy implements RejectedExecutionHandler {
        public AbortPolicy() { }

        public void rejectedExecution(Runnable r, ThreadPoolExecutor e) {
            throw new RejectedExecutionException("Task " + r.toString() +
                                                " rejected from " +
                                                e.toString()); // 抛异常！
        }
    }
    ```

* DiscardPolicy

    抛弃新提交的任务

    ```java
    public static class DiscardPolicy implements RejectedExecutionHandler {
        public DiscardPolicy() { }

        public void rejectedExecution(Runnable r, ThreadPoolExecutor e) {
        }
    }
    ```

* DiscardOldestPolicy

    抛弃下一个被执行的任务，然后重新尝试提交任务

    ```java
    public static class DiscardOldestPolicy implements RejectedExecutionHandler {
        public DiscardOldestPolicy() { }
        
        public void rejectedExecution(Runnable r, ThreadPoolExecutor e) {
            if (!e.isShutdown()) { // 先判断线程池关没
                e.getQueue().poll(); // 丢到等待队列中下一个要被执行的任务
                e.execute(r); // 重新尝试提交新来的任务
            }
        }
    }
    ```

    不要和 PriorityBlockingQueue 一起使用，会丢失优先级最高的任务

* CallerRunsPolicy （既不抛出异常，也不抛弃任务)

    它不会在线程池中执行该任务，而是在调用 execute 提交这个任务的线程执行

    如当主线程提交了任务时，任务队列已满，此时该任务会在主线程中执行。这样主线程在一段时间内不会提交任务给线程池，使得工作者线程有时间来处理完正在执行的任务

    可以实现服务器在高负载下的性能缓慢降低

    ```java
    public static class CallerRunsPolicy implements RejectedExecutionHandler {
        public CallerRunsPolicy() { }

        public void rejectedExecution(Runnable r, ThreadPoolExecutor e) {
            if (!e.isShutdown()) {
                // 直接在把它提交来的线程调用它的 run 方法，相当于没有新建一个线程来执行它，
                // 而是直接在提交它的线程执行它，这样负责提交任务的线程一段时间内不会提交新的任务来
                r.run(); 
            }
        }
    }    
    ```

## 总结

这篇 blog 我们介绍了 ThreadPoolExecutor 这个类，还是讲的比较清楚的，希望对大家有所帮助
