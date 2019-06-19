---
layout:     post
title:      "LinkedHashMap source"
subtitle:   "LinkedHashMap 源码分析"
date:       2019-06-19 11:30:00
author:     "Mickey"
header-img: "img/post-bg-nextgen-web-pwa.jpg"
tags:
    - java
---

这篇 blog 来分析一下 LinkedHashMap 的源码，LinkedHashMap 继承了 HashMap

```java
public class LinkedHashMap<K,V> extends HashMap<K,V> implements Map<K,V>
```

LinkedHashMap 最重要的是以下用于维护顺序的函数，它们会在 put、get 等方法中调用

```
void afterNodeAccess(Node<K,V> p) { }
void afterNodeInsertion(boolean evict) { }
protected boolean removeEldestEntry(Map.Entry<K,V> eldest) {
    return false;
}
```

## 存储结构

```java
/**
 * 双向链表的头部（最早到来的节点）
 */
transient LinkedHashMap.Entry<K,V> head;

/**
 * 双向链表的尾部（最晚到来的节点）
 */
transient LinkedHashMap.Entry<K,V> tail;

/**
 * accessOrder 决定了顺序，默认为 false，维护的是插入顺序
 * 当设为 true 的时候，维护的是访问顺序
 */
final boolean accessOrder;
```

## 构造函数

LinkedHashMap 定义了 5 个构造函数，前四个直接在函数内调用了 HashMap 相应的构造函数，然后将 accessOrder 设置为 false，我们来看看第五个

```java
public LinkedHashMap(int initialCapacity,
                     float loadFactor,
                     boolean accessOrder) {
    super(initialCapacity, loadFactor);
    this.accessOrder = accessOrder;
}
```

因此，使用 LinkedHashMap 来实现 LRU 缓存的时候，需要传入三个参数

```java
class LRUCache<K, V> extends LinkedHashMap<K, V> {
	private static final int MAX_ENTRIES = 3;
	
	protected boolean removeEldestEntry(Map.Entry eldest) {
   		return size() > MAX_ENTRIES;
  	}
	
  	LRUCache() {
  		super(MAX_ENTRIES, 0.75f, true);
  	}
}
```

## afterNodeAccess()

当一个节点被访问时，如果 accessOrder 为 true，则会将该节点移到链表尾部。也就是说指定为 LRU 顺序之后，在每次访问一个节点时，会将这个节点移到链表尾部，保证链表尾部是最近访问的节点，那么链表首部就是最近最久未使用的节点

```java
void afterNodeAccess(Node<K,V> e) { // move node to last
    LinkedHashMap.Entry<K,V> last;
    if (accessOrder && (last = tail) != e) {
        LinkedHashMap.Entry<K,V> p =
            (LinkedHashMap.Entry<K,V>)e, b = p.before, a = p.after;
        p.after = null;
        if (b == null)
            head = a;
        else
            b.after = a;
        if (a != null)
            a.before = b;
        else
            last = b;
        if (last == null)
            head = p;
        else {
            p.before = last;
            last.after = p;
        }
        tail = p;
        ++modCount;
    }
}
```

## afterNodeInsertion()

在 put 等操作之后执行，当 removeEldestEntry() 方法返回 true 时会移除最晚的节点，也就是链表首部节点 first。

evict 只有在构建 Map 的时候才为 false，在这里为 true

```java
void afterNodeInsertion(boolean evict) { // possibly remove eldest
    LinkedHashMap.Entry<K,V> first;
    if (evict && (first = head) != null && removeEldestEntry(first)) {
        K key = first.key;
        removeNode(hash(key), key, null, false, true);
    }
}
```

removeEldestEntry() 默认为 false，如果需要让它为 true，需要继承 LinkedHashMap 并且覆盖这个方法的实现，这在实现 LRU 的缓存中特别有用，通过移除最近最久未使用的节点，从而保证缓存空间足够，并且缓存的数据都是热点数据

## NewNode()

LinkedHashMap 重写了 NewNode 方法，会将新生成的 Node 节点加入 LinkedHashMap 内部的双向链表上

```java
Node<K,V> newNode(int hash, K key, V value, Node<K,V> e) {
    LinkedHashMap.Entry<K,V> p =
        new LinkedHashMap.Entry<K,V>(hash, key, value, e);
    linkNodeLast(p);
    return p;
}

// link at the end of list
private void linkNodeLast(LinkedHashMap.Entry<K,V> p) {
    LinkedHashMap.Entry<K,V> last = tail;
    tail = p;
    if (last == null)
        head = p;
    else {
        p.before = last;
        last.after = p;
    }
}
```
