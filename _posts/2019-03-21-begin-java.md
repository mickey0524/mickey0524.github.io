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

最近在面试的时候，发现很多公司的技术栈都是 Java，虽然语言不是壁垒，但还是会有很多不便，另外，我想看看 Flink 的源码，于是开始学习 Java

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
		
		...
		
		public static void main(String[] args) {
		    LRUCache<Integer, String> cache = new LRUCache<>();
		    cache.put(1, "a");
		    cache.put(2, "b");
		    cache.put(3, "c");
		    cache.get(1);
		    cache.put(4, "d");
		    System.out.println(cache.keySet());
		}
		
		...
		
		[3, 1, 4]
		```
		
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
		```
		ReentrantLock lock = new ReentrantLock(true); // 设置为公平锁
		```
	
	* synchornized 默认是类方法和实例方法走相同的锁，用 ReentrantLock 可以设置实例锁和类锁，这样就能分开
	
	* 可以设置 Condition 一个或者多个条件锁，每个条件对象管理那些已经进入被保护的代码段但还不能运行的线程，当一个线程调用 await 方法之后，它放弃了锁，被阻塞了，进入了条件的等待集合，等待 signal/signalAll 方法将其唤醒，下面举一个银行转钱的例子

		```java
		public class Bank {
			private final double[] accounts;
			private Lock bankLock;
			private Condition sufficientFunds;
			
			public Bank(int n, double initialBalance) {
				accounts = new double[n];
				Arrays.fill(accounts, initialBalance);
				banklock = new ReentrantLock();
				sufficientFunds = bankLock.newCondition();
			}
			
			public void transfer(int from, int to, double amount) throws InterruptedException {
				bankLock.lock();
				try {
					while (accounts[from] < amount) {
						sufficientFunds.await();
					}
					accounts[from] -= amount;
					accounts[to] += amount;
					sufficientFunds.signalAll();
				}
				finally {
					bankLock.unlock();
				}
			}
		}
		```

		
	* 在申请锁的时候，为了避免无休止等待，可以使用 tryLock 方法，在一段时候后，未获得锁，立即返回

		```java
		if (myLock.tryLock(100, TimeUnit.MILLISECONDS)) {
			...
		} else {
			...
		}
		```

* 当在 synchronized 中需要使用 wait()/notity()/notityAll() 的时候，使用 new Byte[0] 比 new Object 更加省空间

* synchronized 作用于普通函数和静态函数的时候，两个线程不能同时访问一个对象的两个 synchronized 方法，无论是类方法还是对象方法
	
* Java 反射与注解

	[Java 反射与注解](https://www.cnblogs.com/xiashengwang/p/8942252.html)
	
* Java 线程操作中的 wait 方法和 notity 方法

	这两个方法多与 synchronized(obj) 一同使用，wait 方法是释放当前获取的锁，同时线程休眠，等待其他线程调用 obj.notity() 或 obj.notityAll() 唤醒继续执行，notity 方法是唤醒其他执行 wait 方法的线程，但是并不马上释放锁，而是等 synchronized 块自己执行完毕
	
	```java
	// 三线程打印问题
	public class MyThreadPrinter2 implements Runnable {   
		  
	    private String name;   
	    private Byte[] prev;   
	    private Byte[] self;   
	  
	    private MyThreadPrinter2(String name, Byte[] prev, Byte[] self) {   
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
	        Byte[] a = new Byte[0];   
	        Byte[] b = new Byte[0];   
	        Byte[] c = new Byte[0]; 
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
	
* Java volatile 关键字

	当一个变量只有赋值操作的时候，用 volatile 很好，但是 volatile 并不能保证原子性

	* 立即刷新可见性
	* 禁止指令重排序

	[Java volatile](https://www.cnblogs.com/dolphin0520/p/3920373.html)

* Java 阻塞队列

	线程安全的队列，可以用于生产者消费者模型
	
	* LinkedBlockingQueue
	* ArrayBlockingQueue
	* DelayQueue 阻塞时间有限的阻塞队列，只有那些延迟已经超过时间的元素可以从队列中移出
	* PriorityBlockingQueue 优先级阻塞队列
	* void put(E element) 添加元素，在必要时阻塞
	* E take() 移除并放回头元素，必要时阻塞
	* boolean offer(E element, long time, TimeUnit unit) 添加给定的元素
	* E poll(long time, TimeUnit unit) 移除并返回头元素

* Java ConcurrentHashMap 用法

	[ConcurrentHashMap 用法](https://blog.csdn.net/zero__007/article/details/49833819)
	
* Java 多线程使用方法

	* 继承 Thread 类，实现 run 方法，无返回值
		
		```
		public class Thread1 extends Thread {
			void run() {}
		}
		
		...
		
		new Thread1().start();
		```
		
	* 实现 Runnable 接口，实现 run 方法，无返回值，需要使用 Thread 或 Executor 调用

		```
		public class Thread1 implements Runnable {}
		
		...
		
		new Thread(new Thread1()).start();
		
		...
		
		ExecutorService pool = Executors.newCachedThreadPool();
		pool.submit(new Thread1());
		``` 
	
	* 实现 Callable 接口，实现 call 方法，有返回值

		```
		public class Thread2 implements Callable<T> {
			T call() {
				return T;
			}
		}
		
		ExecutorService pool = Executors.newCachedThreadPool();
		Future<T> f = pool.submit(new Thread2());
		System.out.Println(f.get());
		
		Thread2 task = new Thread2();
		FutureTask<Integer> futureTask = new FutureTask<Integer>(task);
		pool.submit(futureTask);
		System.out.Println(futureTask.get());
		```

* Java 类如果继承了两个接口有相同的默认方法，那么需要在类中显式的定义，如果是继承的类和接口之间有方法冲突，那么遵从类优先的规则

	```
	interface Named {
		default String getName() { return "Named"; }
	}
	
	interface Person {
		default String getName() { return "Person"; }
	}
	
	class Student implements Person, Named {
		public String getName() {
			return Person.super.getName();
		}
	}
	```
	
* 所有数组类型都有一个 public 的 clone 方法，而不是 protected。可以用这个方法建立一个新数组，包含原数组所有元素的副本。例如：
	
	```
	int[] luckyNumbers = { 2, 3, 5, 7, 11, 13 };
	int[] cloned = luckyNumbers.clone();
	cloned[5] = 12; // doesn't change luckyNumbers[5]
	```
	
* Java String hashCode 实现

	这个 31 是经验之谈，开发人员发现使用 31 的时候 hash 能分布较为均匀

	```
	public int hashCode() {
		int var1 = this.hash;
		if (var1 == 0 && this.value.length > 0) {
			char[] var2 = this.value;
			for (int var3 = 0; var3 < this.value.length; var3++) {
				var1 = 31 * var1 + var2[var3];
			}
			this.hash = var1;
		}
		
		return var1;
	}
	```
	
	哈希表里计算 hash 的方法
	
	```
	int hash(Object key) {
		int h = key.hashCode();
		return (h ^ (h >>> 16)) & (cap - 1)
	}
	```
	
	高 16 位与低 16 位做异或操作，让 hash 值具有高低位的特性，& (cap - 1) 其实使用了除留余数法，& (cap - 1) == % cap
	
* [Java 设计模式](https://github.com/CyC2018/CS-Notes/blob/master/docs/notes/%E8%AE%BE%E8%AE%A1%E6%A8%A1%E5%BC%8F.md#8-%E7%8A%B6%E6%80%81state)

* [Java 虚拟机](https://github.com/CyC2018/CS-Notes/blob/master/docs/notes/Java%20%E8%99%9A%E6%8B%9F%E6%9C%BA.md#%E4%B8%89%E5%86%85%E5%AD%98%E5%88%86%E9%85%8D%E4%B8%8E%E5%9B%9E%E6%94%B6%E7%AD%96%E7%95%A5)

* [JVM 组成](https://juejin.im/post/5cad272a5188254eb942fabe?utm_source=gold_browser_extension)

* [BIO 和 NIO 数量问题](https://juejin.im/post/5c8aea1df265da2de33f6a09)

* [Java ThreadLocal](https://www.cnblogs.com/dolphin0520/p/3920407.html)

* Java 接口和抽象类的区别

    * 接口中的方法都是 public 的，抽象类没有这个限制
    * 接口中的变量都是 final 的，需要初始化定义，抽象类没有这个限制
    * 一个类可以实现多个接口，但只能继承一个抽象类
    * 接口更多专注于实现方法，抽象类更多专注于类结构

* Java 中 == 和 equals

	==：它的作用是判断两个对象的地址是不是相等。即，判断两个对象是不是同一个对象(基本数据类型==比较的是值，引用数据类型==比较的是内存地址)
	
	equals()：它的作用也是判断两个对象是否相等。但它一般有两种使用情况
	
	* 情况1：类没有覆盖 equals() 方法。则通过 equals() 比较该类的两个对象时，等价于通过“==”比较这两个对象
	* 情况2：类覆盖了 equals() 方法。一般，我们都覆盖 equals() 方法来两个对象的内容相等；若它们的内容相等，则返回 true (即，认为这两个对象相等)

	```java
	public class test1 {
		public static void main(String[] args) {
		    String a = new String("ab"); // a 为一个引用
		    String b = new String("ab"); // b为另一个引用,对象的内容一样
		    String aa = "ab"; // 放在常量池中
		    String bb = "ab"; // 从常量池中查找
		    if (aa == bb) // true
		        System.out.println("aa==bb");
		    if (a == b) // false，非同一对象
		        System.out.println("a==b");
		    if (a.equals(b)) // true
		        System.out.println("aEQb");
		    if (42 == 42.0) { // true
		        System.out.println("true");
		    }
		}
	}
	```
	
	String 中的 equals 方法是被重写过的，因为 object 的 equals 方法是比较的对象的内存地址，而 String 的 equals 方法比较的是对象的值。
	
	当创建 String 类型的对象时，虚拟机会在常量池中查找有没有已经存在的值和要创建的值相同的对象，如果有就把它赋给当前引用。如果没有就在常量池中重新创建一个 String 对象。
	
* hashCode 与 equals 

	* 如果两个对象相等，则 hashcode 也是一定相等的
	* 两个对象相等，两个对象分别调用 equals 方法都返回 true
	* 两个对象 hashcode 相等，它们也不一定是相等的，有 hash 碰撞的情况存在
	* equals 方法被覆盖，hashcode 方法也要同时被覆盖
	* hashCode() 的默认行为是对堆上的对象产生独特值。如果没有重写 hashCode()，则该 class 的两个对象无论如何都不会相等（即使这两个对象指向相同的数据）

* Java 的 String 类型是不可改变的，因为成员变量 value 是 final 类型的，但是可以用反射的黑魔法去修改，不过一般不这样做

	```java
	String s = "asd";
	    
	Field f = s.getClass().getDeclaredField("value");
	f.setAccessible(true);
	
	char[] v = (char []) f.get(s);
	v[0] = 'b';
	
	System.out.println(s); // "bsd"
	```

* Java try finally 中的 return 语句

	* 如果 try 中有 return 语句，将其保存在局部变量中
	* 执行完 finally 中的语句，将局部变量返回
	* 如果 finally 中也有 return 语句，则忽略 try 中的，return finally 中的语句

* Java HashMap 和 HashTable 的区别

	* HashMap 是线程不安全的，HashTable 是线程安全的
	* HashMap 中，null 可以作为键，HashTable 不行
	* HashMap 当链表长度大于8的时候，会转为红黑树，HashTable 不会
	* HashMap 的默认大小为 16，扩容的时候变为原来的两倍，HashTable 的默认大小为 11，扩容的时候变为原来的 2n + 1
	* 当给定初始大小的时候，HashTable 会使用你给定的数值，HashMap 会将其扩充为2的幂次

* HashMap 的大小为啥要为2的幂次

	HashMap 内部计算哈希槽的时候使用了除留余数法，取余 (%) 操作中如果除数是2的幂次则等价于与其除数减一的与 (&) 操作（也就是说 hash % length == hash & (length - 1) 的前提是 length 是2的 n 次方）并且采用二进制位操作 &，相对于 % 能够提高运算效率，这就解释了 HashMap 的长度为什么是2的幂次方

* ConcurrentHashMap 和 Hashtable 的区别

	ConcurrentHashMap 和 Hashtable 的区别主要体现在实现线程安全的方式上不同
	
	* 底层数据结构

		JDK1.7 的 ConcurrentHashMap 底层采用分段的数组 + 链表实现，JDK1.8 采用的数据结构跟 HashMap1.8 的结构一样，数组 + 链表/红黑二叉树。Hashtable 和 JDK1.8 之前的 HashMap 的底层数据结构类似都是采用数组 + 链表的形式，数组是 HashMap 的主体，链表则是主要为了解决哈希冲突而存在的
	
	* 实现线程安全的方式

		* 在 JDK1.7 的时候，ConcurrentHashMap（分段锁）对整个桶数组进行了分割分段(Segment)，每一把锁只锁容器其中一部分数据，多线程访问容器里不同数据段的数据，就不会存在锁竞争，提高并发访问率。 到了 JDK1.8 的时候已经摒弃了 Segment 的概念，而是直接用 Node 数组 + 链表 + 红黑树的数据结构来实现，并发控制使用 synchronized 和 CAS 来操作。（JDK1.6以后 对 synchronized锁做了很多优化）整个看起来就像是优化过且线程安全的 HashMap，虽然在JDK1.8中还能看到 Segment 的数据结构，但是已经简化了属性，只是为了兼容旧版本
		* HashTable 使用 synchronized 来保证线程安全，效率非常低下。当一个线程访问同步方法时，其他线程也访问同步方法，可能会进入阻塞或轮询状态，如使用 put 添加元素，另一个线程不能使用 put 添加元素，也不能使用 get，竞争会越来越激烈效率越低
	
	![hashTable](/img/in-post/begin-java/hashTable.jpeg)
	
	![conCurrentHashMap1.7](/img/in-post/begin-java/conCurrentHashMap1.7.jpeg)
	
	![conCurrentHashMap1.8](/img/in-post/begin-java/conCurrentHashMap1.8.jpeg)
	
