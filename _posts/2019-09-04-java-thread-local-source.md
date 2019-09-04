---
layout:     post
title:      "ThreadLocal Source"
subtitle:   "ThreadLocal 源码分析"
date:       2019-09-04 17:30:00
author:     "Mickey"
header-img: "img/post-bg-e2e-ux.jpg"
tags:
    - java
---

这篇 blog 来分析一下 ThreadLocal 的源码，如果有同学不了解 ThreadLocal 的作用，可以自行 Google

## ThreadLocal 使用栗子

```java
public class ThreadLocalTest {
    private static final ThreadLocal<String> threadLocal = new ThreadLocal<>();

    public static void main(String[] args) {
        for (int i = 0; i < 5; i++) {
            new Thread(new MyRunnable()).start();
        }

        try {
            Thread.sleep(5000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }

    public static class MyRunnable implements Runnable {

        @Override
        public void run() {
            String name = Thread.currentThread().getName();
            threadLocal.set(name);

            try {
                Thread.sleep(1000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }

            System.out.println(name + threadLocal.get());
        }
    }
}

...

Thread-4Thread-4
Thread-3Thread-3
Thread-1Thread-1
Thread-0Thread-0
Thread-2Thread-2
```

## ThreadLocal API

👆我们看到了 ThreadLocal 的简单使用场景，这部分我们来看一下这些 API 的源码

在 Thread 类中，存在一个 `ThreadLocal.ThreadLocalMap` 类型的 Field，这是一个 map，map 的 key 是 ThreadLocal 实栗，value 是你想存储的值（暂时这么理解），后面讲 ThreadLocalMap 源码的时候会发现，ThreadLocalMap 中使用 Entry 数组存储 key 和 value

### get

get 操作的话，首先获取当前线程，然后拿到当前线程的 ThreadLocalMap，从👆可以知道，ThreadLocalMap 的 key 为 ThreadLocal 实栗，所以 getEntry 传入 this 获得 ThreadLocalMap.Entry 实栗，进而获取 value

```java
public T get() {
    Thread t = Thread.currentThread();
    ThreadLocalMap map = getMap(t);
    if (map != null) {
        ThreadLocalMap.Entry e = map.getEntry(this);
        if (e != null) {
            @SuppressWarnings("unchecked")
            T result = (T)e.value;
            return result;
        }
    }
    return setInitialValue();
}
```

### set

set 逻辑和 get 类似，获取 ThreadLocalMap 实栗，然后写入 key 和 value

```java
public void set(T value) {
    Thread t = Thread.currentThread();
    ThreadLocalMap map = getMap(t);
    if (map != null)
        map.set(this, value);
    else
        createMap(t, value);
}
```

### remove

remove 同样获取 ThreadLocalMap 实栗，然后从中将当前 threadLocal 对应的 Entry 删除

```java
public void remove() {
    ThreadLocalMap m = getMap(Thread.currentThread());
    if (m != null)
        m.remove(this);
}
```

## ThreadLocalMap

ThreadLocalMap 是 ThreadLocal 的重中之重，我们来看看 ThreadLocalMap 是如何实现的

### ThreadLocalMap.Entry

Entry 继承 WeakReference，包裹了 ThreadLocal 实栗，同时使用 value 字段存储数值

```java
static class Entry extends WeakReference<ThreadLocal<?>> {
    /** The value associated with this ThreadLocal. */
    Object value;

    Entry(ThreadLocal<?> k, Object v) {
        super(k);
        value = v;
    }
}
```

### ThreadLocalMap 属性和构造函数

```java
// Entry 数组初始的长度
private static final int INITIAL_CAPACITY = 16;

// Entry 数组
private Entry[] table;

// 数组中元素的个数
private int size = 0;

// Entry 数组扩容的门槛
private int threshold; // Default to 0
```

ThreadLocalMap 定义了两种构造函数，一种根据传入的键值对生成一个 ThreadLocalMap 实栗，另外一种是原型设计模式，clone 一个传入的 ThreadLocalMap

```java
ThreadLocalMap(ThreadLocal<?> firstKey, Object firstValue) {
    table = new Entry[INITIAL_CAPACITY];
    int i = firstKey.threadLocalHashCode & (INITIAL_CAPACITY - 1);
    table[i] = new Entry(firstKey, firstValue);
    size = 1;
    setThreshold(INITIAL_CAPACITY);
}

private ThreadLocalMap(ThreadLocalMap parentMap) {
    Entry[] parentTable = parentMap.table;
    int len = parentTable.length;
    setThreshold(len);
    table = new Entry[len];

    for (int j = 0; j < len; j++) {
        Entry e = parentTable[j];
        if (e != null) {
            @SuppressWarnings("unchecked")
            ThreadLocal<Object> key = (ThreadLocal<Object>) e.get();
            if (key != null) {
                Object value = key.childValue(e.value);
                Entry c = new Entry(key, value);
                int h = key.threadLocalHashCode & (len - 1);
                while (table[h] != null)
                    h = nextIndex(h, len);
                table[h] = c;
                size++;
            }
        }
    }
}
```

### ThreadLocalMap 解决哈希冲突

我们知道 Java 的 HashMap 使用拉链法解决哈希冲突，ThreadLocalMap 使用 Entry 数组存储键值对，使用了开放地址法，当发生哈希冲突的时候，会沿着 Entry 数组寻找第一个为 null 的下标写入

```java
private static int nextIndex(int i, int len) {
    return ((i + 1 < len) ? i + 1 : 0);
}

...

int h = key.threadLocalHashCode & (len - 1);
while (table[h] != null)
    h = nextIndex(h, len);
table[h] = c;
```

### getEntry

getEntry 获取 ThreadLocal 的 hashCode，使用除留余数法获取下标，是为使用了开发地址法解决哈希冲突的原因，该下标对应的不一定是传入的 key，这种情况下需要沿着 Entry 数组遍历（具体代码大家自行去看源码）

```java
private Entry getEntry(ThreadLocal<?> key) {
    int i = key.threadLocalHashCode & (table.length - 1);
    Entry e = table[i];
    if (e != null && e.get() == key)
        return e;
    else
        return getEntryAfterMiss(key, i, e);
}
```

### remove

remove 获取 ThreadLocal 的 hashCode，使用除留余数法获取下标，然后沿着 Entry 数组找到对应的 Entry 实栗，调用 clear 方法释放，然后调用 expungeStaleEntry 方法进行 Entry 实栗的 rehash（删除元素之后，当前 index key 为 null）

```java
private void remove(ThreadLocal<?> key) {
    Entry[] tab = table;
    int len = tab.length;
    int i = key.threadLocalHashCode & (len-1);
    for (Entry e = tab[i];
            e != null;
            e = tab[i = nextIndex(i, len)]) {
        if (e.get() == key) {
            e.clear();
            expungeStaleEntry(i);
            return;
        }
    }
}
```

### set

set 方法也是一样，找到对应的下标进行写入（具体代码大家自行去看源码）

```java
private void set(ThreadLocal<?> key, Object value) {

    Entry[] tab = table;
    int len = tab.length;
    int i = key.threadLocalHashCode & (len-1);

    for (Entry e = tab[i];
        e != null;
        e = tab[i = nextIndex(i, len)]) {
        ThreadLocal<?> k = e.get();

        if (k == key) {
            e.value = value;
            return;
        }

        if (k == null) {
            replaceStaleEntry(key, value, i);
            return;
        }
    }

    tab[i] = new Entry(key, value);
    int sz = ++size;
    if (!cleanSomeSlots(i, sz) && sz >= threshold)
        rehash();
}
```

### resize

当 size 大于 3/4 的 threshold 的时候（避免滞后），会触发 resize 操作扩容，resize 方法将 Entry 数组的长度扩大两倍，然后遍历老数组，将 key 不为 null 的 Entry 数组拷贝到新数组（使用开放地址法解决冲突）

```java
private void rehash() {
    expungeStaleEntries();

    // Use lower threshold for doubling to avoid hysteresis
    if (size >= threshold - threshold / 4)
        resize();
}

private void resize() {
    Entry[] oldTab = table;
    int oldLen = oldTab.length;
    int newLen = oldLen * 2;
    Entry[] newTab = new Entry[newLen];
    int count = 0;

    for (int j = 0; j < oldLen; ++j) {
        Entry e = oldTab[j];
        if (e != null) {
            ThreadLocal<?> k = e.get();
            if (k == null) {
                e.value = null; // Help the GC
            } else {
                int h = k.threadLocalHashCode & (newLen - 1);
                while (newTab[h] != null)
                    h = nextIndex(h, newLen);
                newTab[h] = e;
                count++;
            }
        }
    }

    setThreshold(newLen);
    size = count;
    table = newTab;
}
```

## 总结

这篇 blog 我们介绍了 ThreadLocal 这个类，还是讲的比较清楚的，希望对大家有所帮助
