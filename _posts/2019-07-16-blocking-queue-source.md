---
layout:     post
title:      "BlockingQueue Source"
subtitle:   "BlockiBlockingQueueng 源码分析"
date:       2019-07-01 16:00:00
author:     "Mickey"
header-img: "img/post-bg-kuaidi.jpg"
tags:
    - java
---

这篇 blog 我们来讲一下 BlockingQueue 的几个子类的源码

# ArrayBlockingQueue

ArrayBlockingQueue 是一个定长的阻塞队列，可以用于实现最基本的生产者 - 消费者模型

## 属性

```java
// 存放队列元素
final Object[] items;

// 下一个 take，poll，peek，remove 操作的元素下标，这个是会循环的，当 takeIndex = cap
// 的时候，会变为 0
int takeIndex;

// 下一个 put，offer，add 操作的元素下标
int putIndex;

// 队列中有多少元素存在
int count;

// lock
final ReentrantLock lock;

// 队列空的时候消费等待的条件
private final Condition notEmpty;

// 队列满的时候写入等待的条件
private final Condition notFull;
```

## take

```java
public E take() throws InterruptedException {
    final ReentrantLock lock = this.lock;
    // 加锁
    lock.lockInterruptibly();
    try {
        // 当队列为空的时候，等待
        while (count == 0)
            notEmpty.await();
        return dequeue();
    } finally {
        lock.unlock();
    }
}

private E dequeue() {
    final Object[] items = this.items;
    @SuppressWarnings("unchecked")
    E x = (E) items[takeIndex];
    // 取出 takeIndex 下标的元素
    items[takeIndex] = null;
    // 当 takeIndex + 1 = items 的长度的时候
    // 重置 takeIndex 为 0 
    if (++takeIndex == items.length)
        takeIndex = 0;
    count--;
    if (itrs != null)
        itrs.elementDequeued();
    // signal 通知等待写入的线程
    notFull.signal();
    return x;
}
```

## put

```java
public void put(E e) throws InterruptedException {
    checkNotNull(e);
    final ReentrantLock lock = this.lock;
    // 加锁
    lock.lockInterruptibly();
    try {
        // 当队列满了的时候，阻塞
        while (count == items.length)
            notFull.await();
        enqueue(e);
    } finally {
        lock.unlock();
    }
}

private void enqueue(E x) {
    final Object[] items = this.items;
    // 写入
    items[putIndex] = x;
    if (++putIndex == items.length)
        putIndex = 0;
    count++;
    notEmpty.signal();
}
```

# LinkedBlockingQueue

LinkedBlockingQueue 是线程安全的链表

## 属性

```java
// LinkedBlockingList 的容量
private final int capacity;

// 当前列表中元素的数量
private final AtomicInteger count = new AtomicInteger();

// 链表队首元素，构造函数中 head = last = Node(null)
// 也就是说，从 head.next 开始存储真正的元素
transient Node<E> head;

// 链表队尾元素
private transient Node<E> last;

// 消费锁
private final ReentrantLock takeLock = new ReentrantLock();

// 当链表为空的时候，等待
private final Condition notEmpty = takeLock.newCondition();

// 生产锁
private final ReentrantLock putLock = new ReentrantLock();

// 当链表容量满了的时候，等待
private final Condition notFull = putLock.newCondition();
```

因为生成和消费分别工作于链表的两个端，因此可以使用两个锁 + Atomic 的操作

## take

```java
public E take() throws InterruptedException {
    E x;
    int c = -1;
    final AtomicInteger count = this.count;
    final ReentrantLock takeLock = this.takeLock;
    // 加 take 锁
    takeLock.lockInterruptibly();
    try {
        // 当 count 为 0 的时候，阻塞
        while (count.get() == 0) {
            notEmpty.await();
        }
        x = dequeue();
        // 原子操作使得 count - 1
        c = count.getAndDecrement();
        // 如果 count - 1 之前大于 1，调用 notEmpty.signal 触发其他等待的线程
        if (c > 1)
            notEmpty.signal();
    } finally {
        takeLock.unlock();
    }
    // 如果 c - 1 之前等于 capacity，现在说明可以写入了
    if (c == capacity)
        signalNotFull();
    return x;
}

// 从这个函数可以看出 head 是不存数据的，取出的是 head.next 的 item
private E dequeue() {
    Node<E> h = head;
    Node<E> first = h.next;
    h.next = h; // help GC
    head = first;
    E x = first.item;
    first.item = null;
    return x;
}
```

## put

```java
public void put(E e) throws InterruptedException {
    // 当 e 为 null 的时候，抛出异常
    if (e == null) throw new NullPointerException();

    int c = -1;
    Node<E> node = new Node<E>(e);
    final ReentrantLock putLock = this.putLock;
    final AtomicInteger count = this.count;
    // 加 put 锁
    putLock.lockInterruptibly();
    try {
        // 当队列满容量的时候，阻塞
        while (count.get() == capacity) {
            notFull.await();
        }
        // 将 node 设置为 tail，然后更新 tail
        enqueue(node);
        c = count.getAndIncrement();
        // 如果写入一个元素之后，依然还有容量，调用 signal 触发其他阻塞的元素
        if (c + 1 < capacity)
            notFull.signal();
    } finally {
        putLock.unlock();
    }
    // 如果写入之前 count 为 0 的话，调用 signalNotEmpty 出发等待消费的线程
    if (c == 0)
        signalNotEmpty();
}

private void enqueue(Node<E> node) {
    last = last.next = node;
}
```

# PriorityBlockingQueue

PriorityBlockingQueue 是线程安全的堆

## 属性

```java
// 存放元素的数组
private transient Object[] queue;

// 当前堆内有多少个元素
private transient int size;

// 构造函数中输入的 Comparator，为 null 的话使用 Comparable 去比较大小
private transient Comparator<? super E> comparator;

// lock
private final ReentrantLock lock;

// 当堆为空的时候，阻塞的条件
private final Condition notEmpty;

// flag，和 CAS 搭配使用，用于扩容的时候标识已经有线程在进行扩容了
private transient volatile int allocationSpinLock;
```

## take

```java
public E take() throws InterruptedException {
    final ReentrantLock lock = this.lock;
    // 加锁
    lock.lockInterruptibly();
    E result;
    try {
        // 当堆为空的时候，等待
        while ( (result = dequeue()) == null)
            notEmpty.await();
    } finally {
        lock.unlock();
    }
    return result;
}

private E dequeue() {
    int n = size - 1;
    if (n < 0)
        return null;
    else {
        Object[] array = queue;
        // index = 0 的元素出堆
        E result = (E) array[0];
        E x = (E) array[n];
        array[n] = null;
        Comparator<? super E> cmp = comparator;
        // 进行下降操作
        if (cmp == null)
            siftDownComparable(0, x, array, n);
        else
            siftDownUsingComparator(0, x, array, n, cmp);
        size = n;
        return result;
    }
}

private static <T> void siftDownComparable(int k, T x, Object[] array,
                                            int n) {
    if (n > 0) {
        Comparable<? super T> key = (Comparable<? super T>)x;
        int half = n >>> 1;           // loop while a non-leaf
        while (k < half) {
            int child = (k << 1) + 1; // assume left child is least
            Object c = array[child];
            int right = child + 1;
            // 比较 parent 的两个儿子节点的大小，得到逻辑上小的
            if (right < n &&
                ((Comparable<? super T>) c).compareTo((T) array[right]) > 0)
                c = array[child = right];
            // 在比较 x 与儿子节点中的较小值，如果 x 小，break，否则将儿子节点中的较小值写入当前索引
            if (key.compareTo((T) c) <= 0)
                break;
            array[k] = c;
            k = child;
        }
        array[k] = key;
    }
}

private static <T> void siftDownUsingComparator(int k, T x, Object[] array,
                                                int n,
                                                Comparator<? super T> cmp) {
    if (n > 0) {
        int half = n >>> 1;
        while (k < half) {
            int child = (k << 1) + 1;
            Object c = array[child];
            int right = child + 1;
            if (right < n && cmp.compare((T) c, (T) array[right]) > 0)
                c = array[child = right];
            if (cmp.compare(x, (T) c) <= 0)
                break;
            array[k] = c;
            k = child;
        }
        array[k] = x;
    }
}
```

## tryGrow

```java
private void tryGrow(Object[] array, int oldCap) {
    // 这里解锁，因为可能 lock 阻塞了 poll 操作
    lock.unlock();
    Object[] newArray = null;
    if (allocationSpinLock == 0 &&
        // CAS 保证一个线程扩容
        UNSAFE.compareAndSwapInt(this, allocationSpinLockOffset,
                                    0, 1)) {
        try {
            // 当 oldCap 小于 64 的时候，扩容为 oldCap * 2 + 2
            // 否则扩容为 oldCap * 1.5
            int newCap = oldCap + ((oldCap < 64) ?
                                    (oldCap + 2) : // grow faster if small
                                    (oldCap >> 1));
            if (newCap - MAX_ARRAY_SIZE > 0) {    // possible overflow
                int minCap = oldCap + 1;
                if (minCap < 0 || minCap > MAX_ARRAY_SIZE)
                    throw new OutOfMemoryError();
                newCap = MAX_ARRAY_SIZE;
            }
            if (newCap > oldCap && queue == array)
                newArray = new Object[newCap];
        } finally {
            allocationSpinLock = 0;
        }
    }
    if (newArray == null) // back off if another thread is allocating
        Thread.yield();
    // 再次获取 lock
    lock.lock();
    if (newArray != null && queue == array) {
        queue = newArray;
        // 将之前数组中的元素 copy 到新的数组
        System.arraycopy(array, 0, newArray, 0, oldCap);
    }
}
```

## offer

```java
public boolean offer(E e) {
    // 当写入的元素为 null 的时候，抛出异常
    if (e == null)
        throw new NullPointerException();
    final ReentrantLock lock = this.lock;
    // 加锁
    lock.lock();
    int n, cap;
    Object[] array;
    // 当容量满了的时候，扩容
    while ((n = size) >= (cap = (array = queue).length))
        tryGrow(array, cap);
    try {
        Comparator<? super E> cmp = comparator;
        if (cmp == null)
            // 使用 Comparable 进行堆的上升操作
            siftUpComparable(n, e, array);
        else
            // 使用自定义的 Comparator 进行上升操作
            siftUpUsingComparator(n, e, array, cmp);
        size = n + 1;
        notEmpty.signal();
    } finally {
        lock.unlock();
    }
    return true;
}
```
