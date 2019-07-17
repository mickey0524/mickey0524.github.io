---
layout:     post
title:      "Deque Source"
subtitle:   "Deque 源码分析"
date:       2019-07-17 17:30:00
author:     "Mickey"
header-img: "img/post-bg-e2e-ux.jpg"
tags:
    - java
---

这篇文章我们来讲一下 ArrayDeque 和 LinkedBlockingDeque 的源码

## ArrayDeque

### 属性

```java
// ArrayDeque 使用数组来存储元素，数组的长度也是 2 的幂次方
// 在 head 和 tail 更改的时候，使用除留余数法来替代取余操作
transient Object[] elements;

// 双端队列头部
transient int head;

// 双端队列尾部
transient int tail;
```

### offer

offer 会调用 addLast 方法

```java
public void addLast(E e) {
    // 当 e 为 null 的时候，抛出 NPE 异常
    if (e == null)
        throw new NullPointerException();
    // 将元素写入 tail 位置
    elements[tail] = e;
    // 除留余数法获取下一个插入的 tail 位置，当 tail == head 的时候，进行扩容
    if ( (tail = (tail + 1) & (elements.length - 1)) == head)
        doubleCapacity();
}

// doubleCapacity 函数会将 [head, length - 1] 的数据放在新数组前面，[0, head - 1]
// 放在新数组后面，保证扩容后的顺序   
private void doubleCapacity() {
    assert head == tail;
    int p = head;
    int n = elements.length;
    int r = n - p;
    int newCapacity = n << 1;
    if (newCapacity < 0)
        throw new IllegalStateException("Sorry, deque too big");
    Object[] a = new Object[newCapacity];
    System.arraycopy(elements, p, a, 0, r);
    System.arraycopy(elements, 0, a, r, p);
    elements = a;
    head = 0;
    tail = n;
}
```

### poll

poll 获取 head 下标的元素

```java
public E pollFirst() {
    int h = head;

    E result = (E) elements[h];
    if (result == null)
        return null;

    elements[h] = null;
    head = (h + 1) & (elements.length - 1);
    return result;
}
```

## LinkedBlockingDeque

LinkedBlockingDeque 和 LinkedBlockingQueue 不一样，Queue 是生产者 - 消费者模型，Deque 是双端队列

### 属性

```java
// 链表的头部
transient Node<E> first;

// 链表的尾部
transient Node<E> last;

// 链表中元素的个数
private transient int count;

// 链表的容量上限
private final int capacity;

// lock
final ReentrantLock lock = new ReentrantLock();

// 链表为空的时候，等待
private final Condition notEmpty = lock.newCondition();

// 链表满了的时候，等待
private final Condition notFull = lock.newCondition();
```

### offer

其实就是在 LinkedList 的基础上，加了锁，然后触发 notEmpty

```java
public boolean offerLast(E e) {
    if (e == null) throw new NullPointerException();
    Node<E> node = new Node<E>(e);
    final ReentrantLock lock = this.lock;
    lock.lock();
    try {
        return linkLast(node);
    } finally {
        lock.unlock();
    }
}
```

### poll

其实就是在 LinkedList 的基础上，加了锁，然后触发 notFull

```java
public E pollFirst() {
    final ReentrantLock lock = this.lock;
    lock.lock();
    try {
        return unlinkFirst();
    } finally {
        lock.unlock();
    }
}
```