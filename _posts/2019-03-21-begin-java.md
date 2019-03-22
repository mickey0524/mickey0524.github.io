---
layout:     post
title:      "begin-java"
subtitle:   "java小白入门"
date:       2019-03-21 20:00:00
author:     "Mickey"
header-img: "img/post-bg-universe.jpg"
tags:
    - java
---

最近在面试的时候，发现很多公司的技术栈都是 Java，虽然语言不是壁垒，但还是会有很多不变，另外，我想看看 Flink 的源码，于是开始学习 Java

* Java 泛型的介绍

    [Java 泛型](http://www.importnew.com/24029.html)
    
* Java 集合

	* LinkedList

		LinkedList 的底层是双向链表

		[LinkedList 用法](https://blog.csdn.net/sinat_36246371/article/details/53709625)
		
	* HashSet

		HashSet 的访问是没有顺序的
		
		[HashSet 用法](https://blog.csdn.net/tingzhiyi/article/details/52152487)
		
	* TreeSet

		TreeSet 是有序的，基于红黑树实现
		
		[TreeSet 用法](https://www.cnblogs.com/Tony-cheen/p/5681831.html)
		
	* PriorityQueue

		使用堆实现的优先级队列
		
		[PriorityQueue 用法](https://blog.csdn.net/fansunion/article/details/79623809)
		
	* WeakHashMap

		弱引用的 HashMap，当一个 key 没有被使用，GC 会将其删除
		
	* LinkedHashMap

		天然的 LRU 缓存 - 链接散列映射将用访问顺序， 而不是插入顺序， 对映射条目进行迭代。 每次调用 get 或 put, 受到影响的条目将从当前的位置删除，并放到条目链表的尾部(只有条目在链表中的位 置会受影响， 而散列表中的桶不会受影响。一个条目总位于与键散列码对应的桶中)
		
* Arrays.asList() 方法将返回一个视图对象，带有访问底层数据的 get 和 set 方法，改变数组的所有方法均会抛出异常

* List 子范围

	List childList = parentList.subList(10, 20);
	
* 有序集合和有序映射可以根据排序顺序建立子范围

	```java
	SortedSet<E> subSet(E from, E to)
	SortedSet<E> headSet(E to)
	SortedSet<E> tailSet(E from)
	
	SortedMap<K, V> subMap(K from, K to)
	SortedMap<K, V> headMap(K to)
	SortedMap<K, V> tailMap(K from)
	```
	
* 集合算法

	* Collections.sort() 集合排序
	* Collections.shuffle() 混排
	* Collections.binarySearch(c, element, comparator) 二分查找

		如果返回值 >= 0，说明目标 c 在集合内，否则，insertionPoint = -i - 1
		
		```java
		if (i < 0) {
			c.add(-i - 1, element);
		}
		```
		
* 集合和数组的转换

	```java
	String[] arr = new String[]{"1", "2"};
   	java.util.List list = new ArrayList<String>(Arrays.asList(arr));
   	String[] arrTmp = (String[]) list.toArray(new String[0]);
	```

* BitSet 位集

	BitSet 可以用于位图，例如判断某个数是否存在可以用 BitSet，在数据量不大的前提下好于 BollmFilter
	
	```java
	bitSet.get(i);  // 如果第 i 位处于“ 开” 状态， 就返回 true; 否则返回 false
	bitSet.set(i);  // 将第 i 位置为“ 开” 状态
	bitSet.clear(i);  // 将第 i 位置为“ 关” 状态
	```

* Java ReentrantLock 锁的用处

	* synchronized 调度的时候优先考虑优先级高的线程，ReentrantLock 可以设置锁为公平的
	* 可以设置 Condition 一个或者多个条件锁，每个条件对象管理那些已经进入被保护的代码段但还不能运行的线程

* 当在 synchronized 中需要使用 wait()/notity()/notityAll() 的时候，使用 new Byte[0] 比 new Object 更加省空间

* synchronized 作用域普通函数和静态函数的时候，其实是有两个锁，对象锁和类锁，两个线程不能同时访问一个对象的两个 synchronized 普通方法，但是两个线程可以一个访问 synchronized 普通方法，一个访问 synchronized 类方法，因为是两个锁的缘故
	
* Java 反射与注解

	[Java 反射与注解](https://www.cnblogs.com/xiashengwang/p/8942252.html)
	
* Java 线程操作中的 wait 方法和 notity 方法

	这两个方法多与 synchronized(obj) 一同使用，wait 方法是释放当前获取的锁，同时线程休眠，等待其他线程调用 obj.notity() 或 obj.notityAll() 唤醒继续执行，notity 方法是唤醒其他执行 wait 方法的线程，但是并不马上释放锁，而是等 synchronized 块自己执行完毕
	
	```java
	// 三线程打印问题
	public class MyThreadPrinter2 implements Runnable {   
		  
	    private String name;   
	    private Object prev;   
	    private Object self;   
	  
	    private MyThreadPrinter2(String name, Object prev, Object self) {   
	        this.name = name;   
	        this.prev = prev;   
	        this.self = self;   
	    }   
	  
	    @Override  
	    public void run() {   
	        int count = 10;   
	        while (count > 0) {   
	            synchronized (prev) {   
	                synchronized (self) {   
	                    System.out.print(name);   
	                    count--;  
	                    
	                    self.notify();   
	                }   
	                try {   
	                    prev.wait();   
	                } catch (InterruptedException e) {   
	                    e.printStackTrace();   
	                }   
	            }   
	  
	        }   
	    }   
	  
	    public static void main(String[] args) throws Exception {   
	        Object a = new Object();   
	        Object b = new Object();   
	        Object c = new Object();   
	        MyThreadPrinter2 pa = new MyThreadPrinter2("A", c, a);   
	        MyThreadPrinter2 pb = new MyThreadPrinter2("B", a, b);   
	        MyThreadPrinter2 pc = new MyThreadPrinter2("C", b, c);   
	           
	           
	        new Thread(pa).start();
	        Thread.sleep(100);  //确保按顺序A、B、C执行
	        new Thread(pb).start();
	        Thread.sleep(100);  
	        new Thread(pc).start();   
	        Thread.sleep(100);  
	        }   
	}
	```
	
* Java 多线程

	[Java 多线程](https://blog.csdn.net/evankaka/article/details/44153709)
