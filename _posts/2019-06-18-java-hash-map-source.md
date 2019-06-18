---
layout:     post
title:      "HashMap source"
subtitle:   "HashMap 源码分析"
date:       2019-06-18 17:30:00
author:     "Mickey"
header-img: "img/post-bg-ios9-web.jpg"
tags:
    - java
---

这篇 blog 来分析一下 HashMap 的源码（本篇文章不涉及红黑树）

## 存储结构

内部包含了一个 Node 类型的数组 table

```java
transient Node<K,V>[] table;
```

Node 存储着键值对，它包含了四个字段，从 next 字段我们可以看出 Node 是一个链表。即数组中的每个位置被当成一个桶，一个桶存放一个链表。HashMap 使用拉链法来解决冲突，同一个链表中存放哈希值和散列桶取模运算结果相同的 Node

```java
static class Node<K,V> implements Map.Entry<K,V> {
    final int hash;
    final K key;
    V value;
    Node<K,V> next;

    Node(int hash, K key, V value, Node<K,V> next) {
        this.hash = hash;
        this.key = key;
        this.value = value;
        this.next = next;
    }

    public final K getKey()        { return key; }
    public final V getValue()      { return value; }
    public final String toString() { return key + "=" + value; }

    public final int hashCode() {
        return Objects.hashCode(key) ^ Objects.hashCode(value);
    }

    public final V setValue(V newValue) {
        V oldValue = value;
        value = newValue;
        return oldValue;
    }

    public final boolean equals(Object o) {
        if (o == this)
            return true;
        if (o instanceof Map.Entry) {
            Map.Entry<?,?> e = (Map.Entry<?,?>)o;
            if (Objects.equals(key, e.getKey()) &&
                Objects.equals(value, e.getValue()))
                return true;
        }
        return false;
    }
}
```

## 计算 key hash

首先，获取 key 的 hashCode，然后将 key 和 key 右移 16 位的结果做异或操作，得到 key 的 hash

```java
static final int hash(Object key) {
    int h;
    return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
}
```

## tableSizeFor 方法

tableSizeFor 方法用于获取大于等于 cap 的最大的 2 的幂指数，例如 `tableSizeFor(5) = 8`，这是为了 hash 取模的时候能够使用除留余数法，👇代码 `n |= n >>> 1` 能够保证两位为1，以此类推，最后 n + 1 就是大于等于 cap 的最大的 2 的幂指数

```java
static final int tableSizeFor(int cap) {
    int n = cap - 1;
    n |= n >>> 1;
    n |= n >>> 2;
    n |= n >>> 4;
    n |= n >>> 8;
    n |= n >>> 16;
    return (n < 0) ? 1 : (n >= MAXIMUM_CAPACITY) ? MAXIMUM_CAPACITY : n + 1;
}
```

## get 操作

首先，get 操作调用 hash 方法获取 key 的 hash 值，然后调用 getNode 方法去 table 中获取对应的 Node

getNode 方法使用除留余数法得到链表的头部节点，然后沿着链表依此比较 hash 值和 key 值是否相等，如果发现 Node 为 TreeNode，则去红黑树中查找

```java
public V get(Object key) {
    Node<K,V> e;
    return (e = getNode(hash(key), key)) == null ? null : e.value;
}

final Node<K,V> getNode(int hash, Object key) {
    Node<K,V>[] tab; Node<K,V> first, e; int n; K k;
    if ((tab = table) != null && (n = tab.length) > 0 &&
        (first = tab[(n - 1) & hash]) != null) {
        if (first.hash == hash && // always check first node
            ((k = first.key) == key || (key != null && key.equals(k))))
            return first;
        if ((e = first.next) != null) {
            if (first instanceof TreeNode)
                return ((TreeNode<K,V>)first).getTreeNode(hash, key);
            do {
                if (e.hash == hash &&
                    ((k = e.key) == key || (key != null && key.equals(k))))
                    return e;
            } while ((e = e.next) != null);
        }
    }
    return null;
}
```

## put 方法

put 方法首先获取 key 的 hash 值，然后调用 putVal 方法

putVal 首先判断 table 是否被初始化过，如果没有，先调用 resize 方法确保键值对能够有空间写入，然后使用除留余数法获取 k 对应的链表下标，如果 table[idx] 为空直接写入，最后就是沿着链表依次比对，如果匹配成功，修改 Node 的 value，否则，在链表最后新建一个 Node 实例

```java
public V put(K key, V value) {
    return putVal(hash(key), key, value, false, true);
}

final V putVal(int hash, K key, V value, boolean onlyIfAbsent,
               boolean evict) {
    Node<K,V>[] tab; Node<K,V> p; int n, i;
    if ((tab = table) == null || (n = tab.length) == 0)
        n = (tab = resize()).length;
    if ((p = tab[i = (n - 1) & hash]) == null)
        tab[i] = newNode(hash, key, value, null);
    else {
        Node<K,V> e; K k;
        if (p.hash == hash &&
            ((k = p.key) == key || (key != null && key.equals(k))))
            e = p;
        else if (p instanceof TreeNode)
            e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value);
        else {
            for (int binCount = 0; ; ++binCount) {
                if ((e = p.next) == null) {
                    p.next = newNode(hash, key, value, null);
                    if (binCount >= TREEIFY_THRESHOLD - 1) // -1 for 1st
                        treeifyBin(tab, hash);
                    break;
                }
                if (e.hash == hash &&
                    ((k = e.key) == key || (key != null && key.equals(k))))
                    break;
                p = e;
            }
        }
        if (e != null) { // existing mapping for key
            V oldValue = e.value;
            if (!onlyIfAbsent || oldValue == null)
                e.value = value;
            afterNodeAccess(e);
            return oldValue;
        }
    }
    ++modCount;
    if (++size > threshold)
        resize();
    afterNodeInsertion(evict);
    return null;
}
```

## resize 方法

resize 方法用于给 HashMap 扩容，这个方法比较重要，分析见下方代码中的注释

```java
final Node<K,V>[] resize() {
	// 获取 table
    Node<K,V>[] oldTab = table;
    // 获取当前 table 数组的长度
    int oldCap = (oldTab == null) ? 0 : oldTab.length;
    // 获取当前的扩容门槛
    int oldThr = threshold;
    // newCap 和 newThr 是本次扩容之后的容量和扩容门槛
    int newCap, newThr = 0;
    // oldCap 大于 0 说明之前 table 已经被初始化了
    if (oldCap > 0) {
        if (oldCap >= MAXIMUM_CAPACITY) {
            threshold = Integer.MAX_VALUE;
            return oldTab;
        }
        // 容量和门槛都乘 2
        else if ((newCap = oldCap << 1) < MAXIMUM_CAPACITY &&
                 oldCap >= DEFAULT_INITIAL_CAPACITY)
            newThr = oldThr << 1;
    }
    // table 等于 null，但是设置了 threshold
    // 说明初始化的时候调用的是 HashMap(int initialCapacity)
    // 因此，我们直接将 oldThr 赋给 newCap
    else if (oldThr > 0)
        newCap = oldThr;
    // 说明初始化调用的是 HashMap()
    else {
        newCap = DEFAULT_INITIAL_CAPACITY;
        newThr = (int)(DEFAULT_LOAD_FACTOR * DEFAULT_INITIAL_CAPACITY);
    }
    if (newThr == 0) {
        float ft = (float)newCap * loadFactor;
        newThr = (newCap < MAXIMUM_CAPACITY && ft < (float)MAXIMUM_CAPACITY ?
                  (int)ft : Integer.MAX_VALUE);
    }
    threshold = newThr;
    // 创建一个长度为 newCap 的新数组，然后将旧数组中的元素写入新数组
    Node<K,V>[] newTab = (Node<K,V>[])new Node[newCap];
    table = newTab;
    if (oldTab != null) {
        for (int j = 0; j < oldCap; ++j) {
            Node<K,V> e;
            if ((e = oldTab[j]) != null) {
                oldTab[j] = null;  // GC
                // 如果链表是有一个 Node 实例，直接写入新的数组
                if (e.next == null)
                    newTab[e.hash & (newCap - 1)] = e;
                // 红黑树操作
                else if (e instanceof TreeNode)
                    ((TreeNode<K,V>)e).split(this, newTab, j, oldCap);
                else {
                    // 这里会生成两个链表，loHead，loTail 指代 hash 值与上 oldCap 等于 0 的链表的头和尾巴
                    Node<K,V> loHead = null, loTail = null;
                    // hiHead，hiTail 指代 hash 值与上 oldCap 不等于 0 的链表的头和尾巴
                    Node<K,V> hiHead = null, hiTail = null;
                    Node<K,V> next;
                    do {
                        next = e.next;
                        // 将链表上的元素写入两个链表
                        if ((e.hash & oldCap) == 0) {
                            if (loTail == null)
                                loHead = e;
                            else
                                loTail.next = e;
                            loTail = e;
                        }
                        else {
                            if (hiTail == null)
                                hiHead = e;
                            else
                                hiTail.next = e;
                            hiTail = e;
                        }
                    } while ((e = next) != null);
                    if (loTail != null) {
                        loTail.next = null;
                       	 // 将一条链表直接放在 newTab[j]
                        newTab[j] = loHead;
                    }
                    if (hiTail != null) {
                        hiTail.next = null;
                        // 将另外一条链表放在 newTab[j + oldCap]
                        // 因为容量翻倍
                        newTab[j + oldCap] = hiHead;
                    }
                }
            }
        }
    }
    return newTab;
}
```