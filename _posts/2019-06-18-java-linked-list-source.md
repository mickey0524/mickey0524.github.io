---
layout:     post
title:      "LinkedList source"
subtitle:   "LinkedList 源码分析"
date:       2019-06-18 14:30:00
author:     "Mickey"
header-img: "img/post-bg-kuaidi.jpg"
tags:
    - java
---

这篇 blog 来分析一下 LinkedList 的源码。LinkedList 是基于双向链表实现的，支持在头部和尾部插入元素

```java
public class LinkedList<E>
    extends AbstractSequentialList<E>
    implements List<E>, Deque<E>, Cloneable, java.io.Serializable
```

## LinkedList 属性

```java
/**
 * 链表中元素的个数
 */
transient int size = 0;

/**
 * 链表头部节点
 */
transient Node<E> first;

/**
 * 链表尾部节点
 */
transient Node<E> last;

/**
 * 链表节点类，item 保存元素，next 指向链表中下一个节点
 * prev 指向链表中上一个节点
 */
private static class Node<E> {
    E item;
    Node<E> next;
    Node<E> prev;

    Node(Node<E> prev, E element, Node<E> next) {
        this.item = element;
        this.next = next;
        this.prev = prev;
    }
}
```

## 构造函数

LinkedList 和 ArrayList 不同，使用双向链表存储元素，构造函数较为简单

```java
public LinkedList() {
}

public LinkedList(Collection<? extends E> c) {
    this();
    addAll(c);
}
```

## 在头部插入元素

创建一个新的 Node 实例，然后 newNode.next 指向之前的 first，之前的 first.prev 指向 newNode，然后将 first 更新为 newNode

```java
private void linkFirst(E e) {
    final Node<E> f = first;
    final Node<E> newNode = new Node<>(null, e, f);
    first = newNode;
    if (f == null)
        last = newNode;
    else
        f.prev = newNode;
    size++;
    modCount++;
}
```

## 在尾部插入元素

创建一个新的 Node 实例，然后 newNode.prev 指向之前的 last，之前的 last.next 指向 newNode，然后将 last 更新为 newNode

```java
void linkLast(E e) {
    final Node<E> l = last;
    final Node<E> newNode = new Node<>(l, e, null);
    last = newNode;
    if (l == null)
        first = newNode;
    else
        l.next = newNode;
    size++;
    modCount++;
}
```

## 在头部删除元素

将 first 更新为 first.next，然后将 first.next.prev 设置为 null

```java
private E unlinkFirst(Node<E> f) {
    // assert f == first && f != null;
    final E element = f.item;
    final Node<E> next = f.next;
    f.item = null;
    f.next = null; // help GC
    first = next;
    if (next == null)
        last = null;
    else
        next.prev = null;
    size--;
    modCount++;
    return element;
}
```

## 在尾部删除元素

将 last 更新为 last.prev，然后将 last.prev.next 设置为 null

```java
private E unlinkLast(Node<E> l) {
    // assert l == last && l != null;
    final E element = l.item;
    final Node<E> prev = l.prev;
    l.item = null;
    l.prev = null; // help GC
    last = prev;
    if (prev == null)
        first = null;
    else
        prev.next = null;
    size--;
    modCount++;
    return element;
}
```

## 序列化

LinkedList 的序列化较为简单，遍历整个链表，序列化每个 node 节点的 item 

```java
private void writeObject(java.io.ObjectOutputStream s)
    throws java.io.IOException {
    // Write out any hidden serialization magic
    s.defaultWriteObject();

    // Write out size
    s.writeInt(size);

    // Write out all elements in the proper order.
    for (Node<E> x = first; x != null; x = x.next)
        s.writeObject(x.item);
}

private void readObject(java.io.ObjectInputStream s)
    throws java.io.IOException, ClassNotFoundException {
    // Read in any hidden serialization magic
    s.defaultReadObject();

    // Read in size
    int size = s.readInt();

    // Read in all elements in the proper order.
    for (int i = 0; i < size; i++)
        linkLast((E)s.readObject());
}
```
