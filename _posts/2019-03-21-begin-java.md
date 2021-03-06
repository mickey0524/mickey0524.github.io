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

* github 上优秀的 Java 知识集合

	* [CS-Notes](https://github.com/CyC2018/CS-Notes)
	* [Java Guide](https://github.com/Snailclimb/JavaGuide)　

* Java 泛型的介绍

    [Java 泛型](http://www.importnew.com/24029.html)
    
	* 泛型类
		
		```java
		public class Box<T> {
			private T t;
			
			public T getT() {
				return t;	
			}
			
			public void setT(T t) {
				this.t = t;
			}
		}
		```

	* 泛型方法
			
		```java
		public class Util {
		    public static <K, V> boolean compare(Pair<K, V> p1, Pair<K, V> p2) {
		        return p1.getKey().equals(p2.getKey()) &&
		               p1.getValue().equals(p2.getValue());
		    }
		}
			
		public class Pair<K, V> {
		    private K key;
		    private V value;
		    public Pair(K key, V value) {
		        this.key = key;
		        this.value = value;
		    }
		    public void setKey(K key) { this.key = key; }
		    public void setValue(V value) { this.value = value; }
		    public K getKey()   { return key; }
		    public V getValue() { return value; }
		}
		```
 
	* 边界符
		
		```java
		public interface Comparable<T> {
			public int compareTo(T o);
		}
			
		public static <T extends Comparable<T>> int countGreaterThan(T[] anArray, T elem) {
		    int count = 0;
		    for (T e : anArray)
		        if (e.compareTo(elem) > 0)
		            ++count;
		    return count;
		}
		```
    
    * 泛型通配符

    	* <? extends T>，向外取数
    	* <? super T>，向内写数
    
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
		
		[PriorityQueue 用法](http://www.importnew.com/6932.html)
		
	* WeakHashMap

		弱引用的 HashMap，当一个 key 没有被使用，GC 会将其删除
		
	* LinkedHashMap

		天然的 LRU 缓存 - 链接散列映射将用访问顺序，而不是插入顺序，对映射条目进行迭代。每次调用 get 或 put, 受到影响的条目将从当前的位置删除，并放到条目链表的尾部(只有条目在链表中的位置会受影响，而散列表中的桶不会受影响。一个条目总位于与键散列码对应的桶中)
		
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
		
		[LinkedHashMap](http://www.importnew.com/18726.html)
		
* Arrays.asList() 方法将返回一个视图对象，带有访问底层数据的 get 和 set 方法，改变数组的所有方法均会抛出异常

* float 和 double

    1.1 这种字面量是 double 类型的，java 不能将精度高的赋予精度低的

    ```java
    float a = 1.1; // 报错
    float a = 1.1f; // 正确
    ```

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

* synchronized 作用于普通函数和静态函数的时候，两个线程能同时访问普通函数和静态函数，因为一个锁作用的是实例对象，一个锁作用的是类对象

* 同步方法常量池中会有一个 ACC\_SYNCHRONIZED 标志，当某个线程访问某个方法的时候，会检查是否有 ACC\_SYNCHRONIZED，如果有，则需要先获得监视器锁，然后开始执行方法，方法执行之后再释放监视器锁，同步代码块则是使用 monitorenter 和 monitorexit 两个指令实现的，monitorenter 可以理解为加锁，monitorexit 可以理解为释放锁，每个对象自身维护着一个被加锁次数的计数器，当计数器为 1 时，只有获得锁的线程才能再次获得锁，即可重入锁，当计数器为0时表示任意线程可以获得该锁

    那么对象如何与监视器关联呢，在 Java 中，对象包含三块：对象头、实例数据、填充数据，其中对象头中就包含 Mark Word，Mark Word 一般存储对象的 hashCode、GC分代年龄以及锁信息，锁信息就包含指向互斥量（重量级锁）的指针，指向了一个监视器；监视器是通过 ObjectMonitor 来实现的，代码如下:

    ```
    ObjectMonitor() {
        _header       = NULL;
        _count        = 0; //记录个数
        _waiters      = 0,
        _recursions   = 0;
        _object       = NULL;
        _owner        = NULL;
        _WaitSet      = NULL; //处于wait状态的线程，会被加入到_WaitSet
        _WaitSetLock  = 0 ;
        _Responsible  = NULL ;
        _succ         = NULL ;
        _cxq          = NULL ;
        FreeNext      = NULL ;
        _EntryList    = NULL ; //处于等待锁block状态的线程，会被加入到该列表
        _SpinFreq     = 0 ;
        _SpinClock    = 0 ;
        OwnerIsThread = 0 ;
    }
    ```

    从上面代码可以看到有 ObjectMonitor 两个队列,分别是 _WaitSet 和 _EntryList，_owner 指向持有 ObjectMonitor 对象的线程，当多个线程获取到对象 monitor 后进入 _owner 区域，并把 _owner 设置为指向当前线程，并把 _count 数量加1；当调用 wait() 方法后，将释放当前持有的 monitor，_owner 置为空，_count 减 1 操作，同时，将该线程进入 _WaitSet 集合中等待唤醒

* Java 反射与注解

	[Java 反射与注解](https://www.cnblogs.com/xiashengwang/p/8942252.html)
	
	* 利用反射获取成员变量 Field

		当获取 private 类型的变量时，需要设置 `field.setAccessible(true);`

		```java
		public Field getDeclaredField(String name) // 获得该类自身声明的所有变量，不包括其父类的变量
		public Field getField(String name) // 获得该类自所有的public成员变量，包括其父类变量
		
		//具体实现
		Field[] allFields = class1.getDeclaredFields();//获取class对象的所有属性 
		Field[] publicFields = class1.getFields();//获取class对象的public属性 
		Field ageField = class1.getDeclaredField("age");//获取class指定属性 
		Field desField = class1.getField("des");//获取class指定的public属性
		```
		
		```java
		public class ReflectDemo {
		    public static void main(String[] args){
		        try {
		            Class c = Class.forName("com.tengj.reflect.Person");
		            //获取成员变量
		            Field field = c.getDeclaredField("msg"); //因为msg变量是private的，所以不能用getField方法
		            Object o = c.newInstance();
		            field.setAccessible(true);//设置是否允许访问，因为该变量是private的，所以要手动设置允许访问，如果msg是public的就不需要这行了。
		            Object msg = field.get(o);
		            System.out.println(msg);
		        } catch (Exception e) {
		            e.printStackTrace();
		        }
		    }
		}
		```
		
	* 利用反射获取成员方法 Method

		```java
		public Method getDeclaredMethod(String name, Class<?>... parameterTypes) // 得到该类所有的方法，不包括父类的 
		public Method getMethod(String name, Class<?>... parameterTypes) // 得到该类所有的public方法，包括父类的
		
		//具体使用
		Method[] methods = class1.getDeclaredMethods();//获取class对象的所有声明方法 
		Method[] allMethods = class1.getMethods();//获取class对象的所有public方法 包括父类的方法 
		Method method = class1.getMethod("info", String.class);//返回次Class对象对应类的、带指定形参列表的public方法 
		Method declaredMethod = class1.getDeclaredMethod("info", String.class);//返回次Class对象对应类的、带指定形参列表的方法
		```
		
		```java
		public void fun(String name,int age) {
	        System.out.println("我叫"+name+",今年"+age+"岁");
		}
		
		Class c = Class.forName("com.tengj.reflect.Person");  //先生成class
		Object o = c.newInstance();                           //newInstance可以初始化一个实例
		Method method = c.getMethod("fun", String.class, int.class);//获取方法
		method.invoke(o, "tengj", 10);                         //通过invoke调用该方法，参数第一个为实例对象，后面为具体参数值
		```
		
	* 利用反射获取构造函数 Constructor

  		```java
  		public Constructor<T> getDeclaredConstructor(Class<?>... parameterTypes) //  获得该类所有的构造器，不包括其父类的构造器
		public Constructor<T> getConstructor(Class<?>... parameterTypes) // 获得该类所以public构造器，包括父类

		//具体
		Constructor<?>[] allConstructors = class1.getDeclaredConstructors();//获取class对象的所有声明构造函数 
		Constructor<?>[] publicConstructors = class1.getConstructors();//获取class对象public构造函数 
		Constructor<?> constructor = class1.getDeclaredConstructor(String.class);//获取指定声明构造函数 
		Constructor publicConstructor = class1.getConstructor(String.class);//获取指定声明的public构造函数
  		```
  		
  		```java
  		public A(String a, int b) {
		    // code body
		}
		
		Constructor constructor = a.getDeclaredConstructor(String.class, int.class);
  		```
  		
  		```java
  		public class ReflectDemo {
		    public static void main(String[] args){
		        try {
		            Class c = Class.forName("com.tengj.reflect.Person");
		            //获取构造函数
		            Constructor constructor = c.getDeclaredConstructor(String.class);
		            constructor.setAccessible(true);//设置是否允许访问，因为该构造器是private的，所以要手动设置允许访问，如果构造器是public的就不需要这行了。
		            constructor.newInstance("tengj");
		        } catch (Exception e) {
		            e.printStackTrace();
		        }
		    }
		}
  		```
  
  	* 注解例子

  		统计方法的调用次数，只有被注解的方法才会参与统计

		```java
		package main;
		
		import java.lang.annotation.ElementType;
		import java.lang.annotation.Retention;
		import java.lang.annotation.RetentionPolicy;
		import java.lang.annotation.Target;
		import java.lang.reflect.InvocationTargetException;
		import java.lang.reflect.Method;
		import java.util.HashMap;
		import java.util.Map;
		
		
		@Retention(RetentionPolicy.RUNTIME)
		@Target(ElementType.METHOD)
		@interface TestAnnotation {
		}
		
		class Test {
		    private HashMap<String, Integer> hashMap;
		
		    public Test() {
		        this.hashMap = new HashMap<>();
		    }
		
		    private void printA() {
		        System.out.println("A");
		    }
		
		    private void printB() {
		        System.out.println("B");
		    }
		
		    @TestAnnotation
		    private void printC() {
		        System.out.println("C");
		    }
		
		    public void print(String name) {
		        try {
		            Method method = this.getClass().getDeclaredMethod(name);
		            method.setAccessible(true);
		
		            if (method.isAnnotationPresent(TestAnnotation.class)) {
		                hashMap.put(name, hashMap.getOrDefault(name, 0) + 1);
		                method.invoke(this);
		            }
		        } catch (NoSuchMethodException | InvocationTargetException | IllegalAccessException e) {
		            e.printStackTrace();
		        }
		    }
		
		    public HashMap<String, Integer> getHashMap() {
		        return this.hashMap;
		    }
		}
		
		public class Main {
			public static void main(String[] args) {
				Test t = new Test();

				t.print("printA");
				t.print("printB");
				t.print("printC");

				for (Map.Entry<String, Integer> entry : t.getHashMap().entrySet()) {
					System.out.println(entry.getKey() + ": " + entry.getValue());
				}
			}
		}
		```
	
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
	        Thread.sleep(100);  // 确保按顺序A、B、C执行
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
	
	```java
	public class Singleton {

	    private volatile static Singleton uniqueInstance;
	
	    private Singleton() {
	    }
	
	    public static Singleton getUniqueInstance() {
	       //先判断对象是否已经实例过，没有实例化过才进入加锁代码
	        if (uniqueInstance == null) {
	            //类对象加锁
	            synchronized (Singleton.class) {
	                if (uniqueInstance == null) {
	                    uniqueInstance = new Singleton();
	                }
	            }
	        }
	        return uniqueInstance;
	    }
	}
	```
	
	👆的代码展示了如何实现一个单例，这里将 uniqueInstance 变量设为 volatile，禁止了指令重排
	
	uniqueInstance = new Singleton(); 这段代码其实是分为三步执行
	
	1. 为 uniqueInstance 分配内存空间
	2. 初始化 uniqueInstance
	3. 将 uniqueInstance 指向分配的内存地址

	但是由于 JVM 具有指令重排的特性，执行顺序有可能变成 1->3->2。指令重排在单线程环境下不会出现问题，但是在多线程环境下会导致一个线程获得还没有初始化的实例。例如，线程 T1 执行了 1 和 3，此时 T2 调用 getUniqueInstance() 后发现 uniqueInstance 不为空，因此返回 uniqueInstance，但此时 uniqueInstance 还未被初始化

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
	
	* 上述几种方式，实现接口好一点

		* Java 不支持多重继承，但是可以实现多个接口
		* 类可能只要求可执行就行，继承整个 Thread 类开销过大

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

	* 运行时数据区域

		* 线程独有

			* 程序计数器，存储虚拟机字节码指令
			* Java 虚拟机栈，存储栈帧，Java 方法执行的时候，会创建一个栈帧用于存储局部变量表、操作数栈、常量池引用等信息，从方法调用直至执行完成的过程，就对应着一个栈帧在 Java 虚拟机栈中入栈和出栈的过程
			* 本地方法栈，本地方法栈与 Java 虚拟机栈类似，它们之间的区别只不过是本地方法栈为本地方法服务（C，C++编写的）
		* 堆，对象分配内存的地方，GC 的主要区域，分为新生代和老年代
		* 方法区，存放被加载的类信息，常量，静态变量等，Java1.7 之前存放于永久代中，由于 full gc 永久代的大小都要改变，经常抛出 OOM 异常，Java1.8 之后存放于本地内存中
		* 运行时常量池，方法区的一部分，Class 文件中的常量池（编译器生成的字面量和符号引用）会在类加载后被放入这个区域
	
	* 如何判断一个对象是否能被回收

		* 引用计数法，循环引用无法回收
		* 可达性分析

			以 GC Roots 为起始点进行搜索，可达的对象都是存活的，不可达的对象可被回收
			
			Java 虚拟机使用该算法来判断对象是否可被回收，GC Roots 一般包含以下内容：
			
			* 虚拟机栈中局部变量表中引用的对象
			* 本地方法栈中 JNI 中引用的对象
			* 方法区中类静态属性引用的对象
			* 方法区中的常量引用的对象
	
	* 引用类型
	
		* 强引用，被强引用关联的对象不会被回收
		* 软引用，软引用关联的对象只有在内存不够的情况下才会被回收
		* 弱引用，下一次 GC 的时候被回收
		* 虚灵引用，为一个对象设置虚引用的唯一目的是能在这个对象被回收时收到一个系统通知
	
	* 垃圾收集算法

		* 标记 - 清除
		* 标记 - 整理
		* 复制
	
	* 7种垃圾收集器
	
		[垃圾收集器](https://github.com/CyC2018/CS-Notes/blob/master/docs/notes/Java%20%E8%99%9A%E6%8B%9F%E6%9C%BA.md#%E5%9E%83%E5%9C%BE%E6%94%B6%E9%9B%86%E5%99%A8)
	
	* 类加载的七个过程：加载，验证，准备，解析，初始化，使用，卸载
	* 类加载器类型：启动类加载器，扩展类加载器，应用程序类加载器

* [JVM 组成](https://juejin.im/post/5cad272a5188254eb942fabe?utm_source=gold_browser_extension)

* [BIO 和 NIO 数量问题](https://juejin.im/post/5c8aea1df265da2de33f6a09)

* [Java ThreadLocal](https://www.cnblogs.com/dolphin0520/p/3920407.html)

* Java 接口和抽象类的区别

    * 接口中的方法都是 public 的，抽象类没有这个限制
    * 接口中的变量都是 final 的，需要初始化定义，抽象类没有这个限制
    * 一个类可以实现多个接口，但只能继承一个抽象类
    * 接口更多专注于实现方法，抽象类更多专注于类结构

* Java 中 == 和 equals

	==：它的作用是判断两个对象的地址是不是相等。即，判断两个对象是不是同一个对象(基本数据类型 ==   比较的是值，引用数据类型 == 比较的是内存地址)
	
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

* HashMap 如何保证数组长度为 2 的幂次方

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
	
	这里解释一下，最开始 n 的二进制高位肯定有一个是 1，n |= n >>> 1 取保了前两位都是 1，n |= n >>> 2 确保了前四位都是 1，依次类推，n |= n >>> 16 确保了高位都是 1，然后 n + 1 完事

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
	
* comparable 和 comparator 的区别

	这两个都是接口，comparable 出自 java.lang 包，有一个 compareTo(Object object) 用来排序，comparator 出自 java.util 包，有一个 compara(Object obj1, Object obj2) 方法用于排序，我们想实现自定义排序，就需要使用这两个
	
* 如何求ArrayList集合的交集 并集 差集 去重复并集

	```
	import java.util.ArrayList;
	import java.util.Arrays;
	
	public class Main {
	    public static void main(String[] args) throws Exception {
	        Integer[] arr1 = new Integer[]{1, 2, 3, 4};
	        Integer[] arr2 = new Integer[]{2, 3, 4, 5};
	
	        ArrayList<Integer> l1 = new ArrayList<>(Arrays.asList(arr1));
	        ArrayList<Integer> l2 = new ArrayList<>(Arrays.asList(arr2));
	
	        // 交集
	        // l1.retainAll(l2);
	
	        // 并集
	        // l1.addAll(l2);
	
	        // 差集
	        // l1.removeAll(l2);
	
	        // 无重复并集
	        l2.removeAll(l1);
	        l1.addAll(l2);
	    }
	}
	```

* [ArrayList源码分析](https://github.com/Snailclimb/JavaGuide/blob/master/docs/java/ArrayList.md)

* ArrayList 源码内有一个 ensureCapacity 方法，在 ArrayList 内部是没有调用的，ArrayList 调用的是 ensureCapacityInternal 方法，这个方法是留给用户使用的，当要 add 大数据量的时候，可以显示调用 ensureCapacity 方法，避免多次扩容

    ```java
    public class EnsureCapacityTest {
        public static void main(String[] args) {
            ArrayList<Object> list = new ArrayList<Object>();
            final int N = 10000000;
            long startTime = System.currentTimeMillis();
            for (int i = 0; i < N; i++) {
                list.add(i);
            }
            long endTime = System.currentTimeMillis();
            System.out.println("使用ensureCapacity方法前："+(endTime - startTime));

            list = new ArrayList<Object>();
            long startTime1 = System.currentTimeMillis();
            list.ensureCapacity(N);
            for (int i = 0; i < N; i++) {
                list.add(i);
            }
            long endTime1 = System.currentTimeMillis();
            System.out.println("使用ensureCapacity方法后："+(endTime1 - startTime1));
        }
    }

    运行结果

    使用ensureCapacity方法前：4637
    使用ensureCapacity方法后：241
    ```

* Java LinkedList API 备忘

	* peek，取链表 first（不删除），如果 first 为 null，返回 null

		```java
		public E peek() {
			final Node<E> f = first;
			return (f == null) ? null : f.item;
		}
		```
		
	* element，取链表 first（不删除），如果 first 为 null，抛出异常

		```java
		public E element() {
      		return getFirst();
    	}
    	
		public E getFirst() {
	   		final Node<E> f = first;
	   			if (f == null)
	          	throw new NoSuchElementException();
	   		return f.item;
	   }
		```
	
	* poll，取链表 first 同时删除，如果 first 为 null，返回 null

		```java
		public E poll() {
			final Node<E> f = first;
			return (f == null) ? null : unlinkFirst(f);
		}
		```
		
	* remove，取链表 first 同时删除，如果 first 为 null，抛出异常

		```java
	    public E removeFirst() {
	        final Node<E> f = first;
	        if (f == null)
	            throw new NoSuchElementException();
	        return unlinkFirst(f);
	    }
	    public E remove() {
	        return removeFirst();
	    }
		```
	
	* offer，add，这两个 API 都是向链表尾部添加一个元素

* [Java synchronized 关键字](https://github.com/Snailclimb/JavaGuide/blob/master/docs/java/synchronized.md)

* [Java 并发编程](https://github.com/CyC2018/CS-Notes/blob/master/docs/notes/Java%20%E5%B9%B6%E5%8F%91.md)

* Java 锁优化

	* 自旋锁，自旋锁的思想是让一个线程在请求一个共享数据的锁时执行忙循环（自旋）一段时间，如果在这段时间内能获得锁，就可以避免进入阻塞状态
	* 锁消除，锁消除是指对于被检测出不可能存在竞争的共享数据的锁进行消除
	* 锁粗化，如果一个方法内连续加锁，释放锁，Java 会将加锁范围扩展到方法
	* 轻量级锁，优先 CAS 操作申请锁，如果两个线程同时等待锁，轻量级锁膨胀为重量级锁
	* 偏向锁，锁偏向于第一个来的线程，这个线程获取锁后，剩下的所有操作都不用同步，CAS 都不用，等到其他线程来请求锁，从当前状态退化

* Java 内存模型三大特性

	![Java 内存模型](/img/in-post/begin-java/java-memory-model.png)

	* 原子性

		Java 内存模型保证了 read、load、use、assign、store、write、lock 和 unlock 操作具有原子性，例如对一个 int 类型的变量执行 assign 赋值操作，这个操作就是原子性的。但是 Java 内存模型允许虚拟机将没有被 volatile 修饰的 64 位数据（long，double）的读写操作划分为两次 32 位的操作来进行，即 load、store、read 和 write 操作可以不具备原子性。
		
	* 可见性

		可见性指当一个线程修改了共享变量的值，其它线程能够立即得知这个修改。Java 内存模型是通过在变量修改后将新值同步回主内存，在变量读取前从主内存刷新变量值来实现可见性的
		
		* volatile
		* synchronized，对一个变量执行 unlock 操作之前，必须把变量值同步回主内存
		* final，被 final 关键字修饰的字段在构造器中一旦初始化完成，并且没有发生 this 逃逸（其它线程通过 this 引用访问到初始化了一半的对象），那么其它线程就能看见 final 字段的值
		
	* 有序性

		在本线程内观察，所有操作都是有序的。在一个线程观察另一个线程，所有操作都是无序的，无序是因为发生了指令重排序。在 Java 内存模型中，允许编译器和处理器对指令进行重排序，重排序过程不会影响到单线程程序的执行，却会影响到多线程并发执行的正确性
		
		volatile 关键字通过添加内存屏障的方式来禁止指令重排，即重排序时不能把后面的指令放到内存屏障之前
		
* Java ConcurrentSkipListMap 采用跳表实现

* 当使用 synchronized 的时候，不要使用 synchronized(String a)，因为 JVM 中，字符串常量池具有缓冲功能

* synchronized 关键字的底层原理

	* synchronized 同步语句块的情况

		synchronized 同步语句块的实现使用的是 monitorenter 和 monitorexit 指令，其中 monitorenter 指令指向同步代码块的开始位置，monitorexit 指令则指明同步代码块的结束位置。 当执行 monitorenter 指令时，线程试图获取锁也就是获取 monitor (monitor 对象存在于每个Java对象的对象头中，synchronized 锁便是通过这种方式获取锁的，也是为什么Java中任意对象可以作为锁的原因) 的持有权.当计数器为0则可以成功获取，获取后将锁计数器设为1也就是加1。相应的在执行 monitorexit 指令后，将锁计数器设为0，表明锁被释放。如果获取对象锁失败，那当前线程就要阻塞等待，直到锁被另外一个线程释放为止
		
	* synchronized 同步方法的情况

		synchronized 修饰的方法并没有 monitorenter 指令和 monitorexit 指令，取得代之的确实是 ACC_SYNCHRONIZED 标识，该标识指明了该方法是一个同步方法，JVM 通过该 ACC_SYNCHRONIZED 访问标志来辨别一个方法是否声明为同步方法，从而执行相应的同步调用
		
* 为什么要使用线程池

	* 降低资源消耗：通过重复利用已创建的线程降低线程创建和销毁造成的消耗
	* 提高响应速度：当任务到达时，任务可以不需要的等到线程创建就能立即执行
	* 提高线程的可管理性：线程是稀缺资源，如果无限制的创建，不仅会消耗系统资源，还会降低系统的稳定性，使用线程池可以进行统一的分配，调优和监控

* String 字符串拼接

	```java
	String str1 = "str";
	String str2 = "ing";
	  
	String str3 = "str" + "ing"; // 常量池中的对象
	String str4 = str1 + str2; // 在堆上创建的新的对象	  
	String str5 = "string"; // 常量池中的对象
	System.out.println(str3 == str4); // false
	System.out.println(str3 == str5); // true
	System.out.println(str4 == str5); // false
	```
	
	尽量避免多个字符串拼接，因为这样会重新创建对象。如果需要改变字符串的话，可以使用 StringBuilder 或者 StringBuffer
	
* Java 中两种浮点数类型的包装类 Float,Double 并没有实现常量池技术

	```java
	Integer i1 = 33;
	Integer i2 = 33;
	System.out.println(i1 == i2); // 输出 true
	Integer i11 = 333;
	Integer i22 = 333;
	System.out.println(i11 == i22); // 输出 false
	Double i3 = 1.2;
	Double i4 = 1.2;
	System.out.println(i3 == i4); // 输出 false
	```
	
* Integer 问题

	```java
	Integer i1 = 40;
	Integer i2 = 40;
	Integer i3 = 0;
	Integer i4 = new Integer(40);
	Integer i5 = new Integer(40);
	Integer i6 = new Integer(0);
	  
	System.out.println("i1=i2   " + (i1 == i2));
	System.out.println("i1=i2+i3   " + (i1 == i2 + i3));
	System.out.println("i1=i4   " + (i1 == i4));
	System.out.println("i4=i5   " + (i4 == i5));
	System.out.println("i4=i5+i6   " + (i4 == i5 + i6));   
	System.out.println("40=i5+i6   " + (40 == i5 + i6));
	
	i1=i2   true
	i1=i2+i3   true
	i1=i4   false
	i4=i5   false
	i4=i5+i6   true
	40=i5+i6   true 
	```
	
	* `Integer i1 = 40;` Java 在编译的时候会直接将代码封装成Integer i1=Integer.valueOf(40);，从而使用常量池中的对象。
	* `Integer i1 = new Integer(40);` 这种情况下会创建新的对象。
	* 语句i4 == i5 + i6，因为+这个操作符不适用于Integer对象，首先i5和i6进行自动拆箱操作，进行数值相加，即i4 == 40。然后Integer对象无法与数值进行直接比较，所以i4自动拆箱转为int值40，最终这条语句转为40 == 40进行数值比较

* [Java 虚拟机如何创建一个对象](https://github.com/Snailclimb/JavaGuide/blob/master/docs/java/jvm/Java%E5%86%85%E5%AD%98%E5%8C%BA%E5%9F%9F.md)

* GC 中不可达的对象并非 "非死不可"

	即使在可达性分析法中不可达的对象，也并非是“非死不可”的，这时候它们暂时处于“缓刑阶段”，要真正宣告一个对象死亡，至少要经历两次标记过程；可达性分析法中不可达的对象被第一次标记并且进行一次筛选，筛选的条件是此对象是否有必要执行 finalize 方法。当对象没有覆盖 finalize 方法，或 finalize 方法已经被虚拟机调用过时，虚拟机将这两种情况视为没有必要执行。

	被判定为需要执行的对象将会被放在一个队列中进行第二次标记，除非这个对象与引用链上的任何一个对象建立关联，否则就会被真的回收。
    
    finalize() 方法是对象逃脱死亡的最后一次机会，不过虚拟机不保证等待 finalize() 方法执行结束，也就是说，虚拟机只触发 finalize() 方法的执行，如果这个方法要执行超久，那么虚拟机并不等待它执行结束，所以最好不要用这个方法

    finalize() 方法能做的，try-finally 都能做，所以忘了这个方法吧

* 如何判断一个常量是废弃常量

	运行时常量池主要回收的是废弃的常量。那么，我们如何判断一个常量是废弃常量呢？

	假如在常量池中存在字符串 "abc"，如果当前没有任何String对象引用该字符串常量的话，就说明常量 "abc" 就是废弃常量，如果这时发生内存回收的话而且有必要的话，"abc" 就会被系统清理出常量池。

* 如何判断一个类是无用的类

	方法区主要回收的是无用的类，那么如何判断一个类是无用的类的呢？

	判定一个常量是否是“废弃常量”比较简单，而要判定一个类是否是“无用的类”的条件则相对苛刻许多。类需要同时满足下面3个条件才能算是 “无用的类” ：

	* 该类所有的实例都已经被回收，也就是 Java 堆中不存在该类的任何实例。
	* 加载该类的 ClassLoader 已经被回收。
	* 该类对应的 java.lang.Class 对象没有在任何地方被引用，无法在任何地方通过反射访问该类的方法。
	
	虚拟机可以对满足上述3个条件的无用类进行回收，这里说的仅仅是“可以”，而并不是和对象一样不使用了就会必然被回收。

* BIO 面向的是流，一次性处理一个字节，NIO 面向的是块，效率好很多，NIO 三个关键类，Selector，Buffer，Channel

    ![NIO](/img/in-post/begin-java/NIO.jpg)

    ```java
    public class NIOServer {
    
        public static void main(String[] args) throws IOException {
    
            Selector selector = Selector.open();
    
            ServerSocketChannel ssChannel = ServerSocketChannel.open();
            ssChannel.configureBlocking(false);
            ssChannel.register(selector, SelectionKey.OP_ACCEPT);
    
            ServerSocket serverSocket = ssChannel.socket();
            InetSocketAddress address = new InetSocketAddress("127.0.0.1", 8888);
            serverSocket.bind(address);
    
            while (true) {
    
                selector.select();
                Set<SelectionKey> keys = selector.selectedKeys();
                Iterator<SelectionKey> keyIterator = keys.iterator();
    
                while (keyIterator.hasNext()) {
    
                    SelectionKey key = keyIterator.next();
    
                    if (key.isAcceptable()) {
    
                        ServerSocketChannel ssChannel1 = (ServerSocketChannel) key.channel();
    
                        // 服务器会为每个新连接创建一个 SocketChannel
                        SocketChannel sChannel = ssChannel1.accept();
                        sChannel.configureBlocking(false);
    
                        // 这个新连接主要用于从客户端读取数据
                        sChannel.register(selector, SelectionKey.OP_READ);
    
                    } else if (key.isReadable()) {
    
                        SocketChannel sChannel = (SocketChannel) key.channel();
                        System.out.println(readDataFromSocketChannel(sChannel));
                        sChannel.close();
                    }
    
                    keyIterator.remove();
                }
            }
        }
    
        private static String readDataFromSocketChannel(SocketChannel sChannel) throws IOException {
    
            ByteBuffer buffer = ByteBuffer.allocate(1024);
            StringBuilder data = new StringBuilder();
    
            while (true) {
    
                buffer.clear();
                int n = sChannel.read(buffer);
                if (n == -1) {
                    break;
                }
                buffer.flip();
                int limit = buffer.limit();
                char[] dst = new char[limit];
                for (int i = 0; i < limit; i++) {
                    dst[i] = (char) buffer.get(i);
                }
                data.append(dst);
                buffer.clear();
            }
            return data.toString();
        }
    }
    ```
* Java CAS 用版本号解决 ABA 问题

	[JAVA CAS 解决 ABA 问题](https://www.cnblogs.com/java20130722/p/3206742.html)
	
* Java CopyOnWriteArrayList 读不加锁，写的话拷贝出一个新数组，然后修改这个新数组，最后将原来的内存指针指向新的数组

	* 读操作

		```java
		/** The array, accessed only via getArray/setArray. */
		private transient volatile Object[] array;
		public E get(int index) {
		    return get(getArray(), index);
		}
		@SuppressWarnings("unchecked")
		private E get(Object[] a, int index) {
		    return (E) a[index];
		}
		final Object[] getArray() {
		    return array;
		}		
		```
	
	* 写操作

		```java
		/**
		 * Appends the specified element to the end of this list.
		 *
		 * @param e element to be appended to this list
		 * @return {@code true} (as specified by {@link Collection#add})
		 */
		public boolean add(E e) {
		    final ReentrantLock lock = this.lock;
		    lock.lock();//加锁
		    try {
		        Object[] elements = getArray();
		        int len = elements.length;
		        Object[] newElements = Arrays.copyOf(elements, len + 1);//拷贝新数组
		        newElements[len] = e;
		        setArray(newElements);
		        return true;
		    } finally {
		        lock.unlock();//释放锁
		    }
		}		
		```

* Java finally 和 finalize 有什么区别

	finally 是配合 try catch 使用的，无论是否捕获异常，Java 都会执行 finally 块中的代码，经常将锁的释放，连接池的释放放在 finally 中执行
	
	finalize()是Object的protected方法，子类可以覆盖该方法以实现资源清理工作，GC在回收对象之前调用该方法，当对象变成(GC Roots)不可达时，GC会判断该对象是否覆盖了finalize方法，若未覆盖，则直接将其回收。否则，若对象未执行过finalize方法，将其放入F-Queue队列，由一低优先级线程执行该队列中对象的finalize方法。执行finalize方法完毕后，GC会再次判断该对象是否可达，若不可达，则进行回收，否则，对象“复活”

* Java 克隆

	```java
	public class Stu implements Cloneable {
		private String name;
		
		public Stu(String name) {
			this.name = name;
		}
		
		@Override
		public object clone() {
			Stu s = null;
			try {
				s = (Stu) super.clone();
			} catch(CloneNotSupportedException e) {
            e.printStackTrace();
        	}
        	return s;
		}
	}
	```

	* 直接相等

		```java
		Stu a = new Stu("a");
		Stu b = a;
		```
		
		这样 b 和 a 指向的是相同的引用
		
	* 浅拷贝

		```java
		Stu a = new Stu("a");
		Stu b = (Stu) a.clone();
		```
		
	* 深拷贝

		Stu 类中有成员变量是其他类
		
		```java
		class Address implements Cloneable {
			private String addr;
			
			Address(String addr) {
				this.addr = addr;
			}
			
			@Override
			public Object clone() {
				Address a = null;
				try {
					a = (Address) super.clone();
				} catch(CloneNotSupportedException e) {
	            e.printStackTrace();
	        	}
	        	return a;
			}
		}
		
		class Stu implements Cloneable {
			private String name;
			
			private Address addr;
			
			Stu(String name, Address addr) {
				this.name = name;
				this.addr = addr;
			}
			
			@Override
			public Object clone() {
				Stu s = null;
				try {
					s = (Stu) super.clone();
					s.addr = (Address) this.addr.clone();
				} catch(CloneNotSupportedException e) {
	          	e.printStackTrace();
	        	}
	        	return s;
			}
		}
		```
		
		如果类中有很多成员变量是其他类，这样写就会很繁琐，可以实现 Serializable 接口来实现深拷贝
		
		```java
		public class Inner implements Serializable{
			private static final long serialVersionUID = 872390113109L; //最好是显式声明ID
			public String name = "";
				
			public Inner(String name) {
				this.name = name;
			}
				
			@Override
			public String toString() {
				return "Inner的name值为：" + name;
			}
		}
		
		public class Outer implements Serializable{
			private static final long serialVersionUID = 369285298572941L;  //最好是显式声明ID
			public Inner inner;
			　//Discription:[深度复制方法,需要对象及对象所有的对象属性都实现序列化]　
			public Outer myclone() {
				Outer outer = null;
				try { // 将该对象序列化成流,因为写在流里的是对象的一个拷贝，而原对象仍然存在于JVM里面。所以利用这个特性可以实现对象的深拷贝
					ByteArrayOutputStream baos = new ByteArrayOutputStream();
					ObjectOutputStream oos = new ObjectOutputStream(baos);
					oos.writeObject(this); // 将流序列化成对象
					ByteArrayInputStream bais = new ByteArrayInputStream(baos.toByteArray());
					ObjectInputStream ois = new ObjectInputStream(bais);
					outer = (Outer) ois.readObject();
				} catch (IOException e) {
					e.printStackTrace();
				} catch (ClassNotFoundException e) {
					e.printStackTrace();
				}
				return outer;
			}
		}
		```

* Java 类的静态方法不能调用类的非静态方法，但是可以调用构造方法

* Java 中的 Arrays.copeOf 和 System.arraycopy 的区别

	* Arrays.copeOf

		```java
		int[] copied = Arrays.copyOf(arr, 10); //10 the the length of the new array
		System.out.println(Arrays.toString(copied));
		 
		copied = Arrays.copyOf(arr, 3);
System.out.println(Arrays.toString(copied));

		[1, 2, 3, 4, 5, 0, 0, 0, 0, 0]
		[1, 2, 3]
		```
	
	* System.arraycopy

		```java
		int[] arr = {1,2,3,4,5};
 
		int[] copied = new int[10];
		System.arraycopy(arr, 0, copied, 1, 5);//5 is the length to copy
		 
		System.out.println(Arrays.toString(copied));
		
		[0, 1, 2, 3, 4, 5, 0, 0, 0, 0]
		```
		
	* 两者的区别

		两者的区别在于，Arrays.copyOf 不仅仅只是拷贝数组中的元素，在拷贝元素时，会创建一个新的数组对象。而 System.arrayCopy 只拷贝已经存在数组元素
		
		如果我们看过 Arrays.copyOf 的源码就会知道，该方法的底层还是调用了 System.arrayCopy 方法
		
		```java
		public static int[] copyOf(int[] original, int newLength) 			{ 
   			int[] copy = new int[newLength]; 
   			System.arraycopy(original, 0, copy, 0, Math.min(original.length, newLength)); 
   			return copy; 
		}
		```

* Java 的 ThreadLocal

	ThreadLocal 用来做线程本地缓存
	
	```java
	public class ThreadLocalExample1 {
	    public static void main(String[] args) {
	        ThreadLocal threadLocal1 = new ThreadLocal();
	        ThreadLocal threadLocal2 = new ThreadLocal();
	        Thread thread1 = new Thread(() -> {
	            threadLocal1.set(1);
	            threadLocal2.set(1);
	        });
	        Thread thread2 = new Thread(() -> {
	            threadLocal1.set(2);
	            threadLocal2.set(2);
	        });
	        thread1.start();
	        thread2.start();
	    }
	}
	```
	
	每个 Thread 都有一个 ThreadLocal.ThreadLocalMap 对象
	
	```java
	ThreadLocal.ThreadLocalMap threadLocals = null;
	```
	
	当调用一个 ThreadLocal 的 set(T value) 方法时，先得到当前线程的 ThreadLocalMap 对象，然后将 ThreadLocal->value 键值对插入到该 Map 中
	
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
	
	get() 方法类似
	
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
	
	ThreadLocal 从理论上讲并不是用来解决多线程并发问题的，因为根本不存在多线程竞争，在一些场景 (尤其是使用线程池) 下，由于 ThreadLocal.ThreadLocalMap 的底层数据结构导致 ThreadLocal 有内存泄漏的情况，应该尽可能在每次使用 ThreadLocal 后手动调用 remove()，以避免出现 ThreadLocal 经典的内存泄漏甚至是造成自身业务混乱的风险

* [Java并发之AQS源码分析（一）](http://objcoding.com/2019/05/05/aqs-exclusive-lock/)

* Java Map 类集合 K/V 是否能存储 null

    ![map-null](/img/in-post/begin-java/map-null.png)

* [为什么阿里巴巴要求谨慎使用ArrayList中的subList方法](https://juejin.im/post/5d117ef7f265da1ba328d240?utm_source=gold_browser_extension)

* 创建线程或线程池的时候请指定有意义的线程名称，方便出错时回溯

    ```java
    public class TimeTaskThread extends Thread {
        public TimeTaskThread() {
            super.setName("TimerTaskThread");           
            ...
        }
    }
    ```

* 使用线程池的好处是减少在创建和销毁线程上所消耗的时间以及系统资源的开销，解决资源不足的问题。如果不使用线程池，有可能造成系统创建大量同类线程而导致消耗完内存或者"过度切换"的问题

* 线程池不允许使用 Executors 去创建，而是通过 ThreadPoolExecutor 的方式，这样的处理方式能够规避资源耗尽的风险

    FixedThreadPool 和 SingleThreadPool

    允许的请求队列长度为 Integer.MAX_VALUE，可能会堆积大量的请求，从而导致 OOM

    CachedThreadPool 和 ScheduledThreadPool

    允许的创建线程数量为 Integer.MAX_VALUE，可能会创建大量的线程，从而导致 OOM

* 避免 Random 实例被多线程使用，虽然共享该实例是线程安全的，但是会因为竞争统一 seed 导致性能的下降，Random 实例包括 java.util.Random 的实例或 Math.random() 的方式，在 JDK7 之后，可以直接使用 API ThreadLocalRandom

    ```java
    ThreadLocalRandom.current().nextInt()
    ```

* HashMap 在容量不够进行 resize 的时候由于高并发可能出现死链，导致 CPU 飙升，在开发过程中可以使用其他数据结构或加锁来规避此风险

* 在高并发场景中，避免使用"等于"判断作为中断或退出的条件

    如果并发控制没有处理好，容易产生等值判断被"击穿"的情况，使用大于或小于的区间判断条件来代替

* Java 内存模型
    
    Java 内存模型，其实是保证了 Java 程序在各种平台下对内存的访问都能得到一致效果的机制及规范。目的是解决由于多线程通过共享内存进行通信时，存在的原子性、可见性（缓存一致性）以及有序性问题

    除此之外，Java 内存模型还提供了一系列原语，封装了底层实现后，供开发者直接使用，如我们常见的 synchronized、volatile 以及并发包等

    * 内存间交互操作

        Java 内存模型定义了 8 个操作来完成主内存和工作内存的交互操作

        ![jmm](/img/in-post/begin-java/jmm.png)

        1. read：把一个变量的值从主内存传输到工作内存中
        2. load：在 read 之后执行，把 read 得到的值放入工作内存的变量副本中
        3. use：把工作内存中一个变量的值传递给执行引擎
        4. assign：把一个从执行引擎接收到的值赋给工作内存的变量
        5. store：把工作内存的一个变量的值传送到主内存中
        6. write：在 store 之后执行，把 store 得到的值放入主内存的变量中
        7. lock：作用于主内存的变量
        8. unlock：作用于主内存的变量

	* happen-before 原则
        
        1. 程序次序原则：一个线程内，按照程序代码顺序，书写在前面的操作先发生于书写在后面的操作
        2. volatile 规则：volatile 变量的写，先发生于读，这保证了 volatile 变量的可见性
        3. 锁规则：解锁（unlock） 必然发生在随后的加锁（lock）前
        4. 传递性：A先于B，B先于C，那么A必然先于C
        5. 线程的 start 方法先于他的每一个动作
        6. 线程的所有操作先于线程的终结
        7. 线程的中断（interrupt（））先于被中断的代码
        8. 对象的构造函数，结束先于 finalize 方法

	* 内存屏障（volatile）

        怎么保证有序性呢？实际上，在操作 volatile 关键字变量前后的汇编代码中，会有一个 lock 前缀，根据 intel IA32 手册，lock 的作用是 使得 本 CPU 的Cache 写入了内存，该写入动作也会引起别的CPU或者别的内核无效化其Cache，别的CPU需要重新获取Cache。这样就实现了可见性。可见底层还是使用的 CPU 的指令。
        
        如何实现有序性呢？同样是lock 指令，这个指令还相当于一个内存屏障（大多数现代计算机为了提高性能而采取乱序执行，这使得内存屏障成为必须。语义上，内存屏障之前的所有写操作都要写入内存；内存屏障之后的读操作都可以获得同步屏障之前的写操作的结果。因此，对于敏感的程序块，写操作之后、读操作之前可以插入内存屏障），指的是，重排序时不能把后面的指令重排序到内存屏障之前的位置。只有一个CPU访问内存时，并不需要内存屏障；但如果有两个或者更多CPU访问同一块内存，且其中有一个在观测另一个，就需要内存屏障来保证了。
        
        因此请不要随意使用 volatile 变量，这会导致 JIT 无法优化代码，并且会插入很多的内存屏障指令，降低性能

* [Java 序列化](https://www.hollischuang.com/archives/1140)

* Java 动态代理

	[Java 动态代理](https://juejin.im/post/5ad3e6b36fb9a028ba1fee6a)
	
	[从代理模式再出发！Proxy.newProxyInstance的秘密](https://blog.csdn.net/lovejj1994/article/details/78080124)
	
	动态代理内部其实是由两个静态代理组成的，实现了 InvocationHandler 接口的代理类内部有一个指向委托类的引用，然后 Proxy.newProxyInstance 内部通过反射创建了一个 \_Proxy 类（实现了委托类的接口），\_Proxy 内部有指向代理类的引用，\_Proxy 对委托类接口的调用都会转为对 InvocationHandler 中 invoke 方法的调用
    
    [Java Proxy和CGLIB动态代理原理](https://www.cnblogs.com/CarpenterLee/p/8241042.html)
    
* 线程安全：当多个线程访问一个对象的时候，如果不用考虑这些线程在运行时环境下的调度和交替执行，也不需要进行额外的同步，或者在调用方进行任何其他的协调操作，调用这个对象的行为都能得到正确的结果，那么这个对象就是线程安全的

* Java NIO 部分源码讲解

    * [NIO Demo](https://github.com/CyC2018/CS-Notes/blob/master/notes/Java%20IO.md#%E5%A5%97%E6%8E%A5%E5%AD%97-nio-%E5%AE%9E%E4%BE%8B)
    
    * [NIO源码分析](https://www.jianshu.com/p/4ad93f696fb2)
    
    * [sun.nio docs & code](http://www.docjar.com/src/api/sun/nio/ch/EPollSelectorImpl.java)
    
    讲解一下主要方法，`ServerSocketChannel.open()` 会返回一个 `ServerSocketChannelImpl` 实例，`Selector.open()` 会根据机器型号返回不同的实例，Mac 和 Linux 上会返回 `EPollSelectorImpl`，`ssc.socket` 使用适配器模式返回一个包裹 ssc 的 ServerSocket，serverSocket 执行 bind 等同于 ssc 执行 bind

    `ssc.register(selector, SelectionKey.OP_ACCEPT)` 会返回一个与 ssc 对应的 SelectionKey，方法内部会执行 `selector.register` 方法，selector 中有一个 map，保存着 channel 的 fd 到 SelectionKey 的映射，`selector.register` 方法会为 ssc 创建一个 `SelectionKeyImpl` 实例，然后将 <ssc.getFDVal(), SelectionKeyImpl> 的 pair 写入 map
    
    我们在调用 `ssc.register` 的时候，传入的是 SelectionKey.OP\_ACCEPT，在 SelectionKey 中，存在着 4 个 op 标识 —— OP\_READ、OP\_WRITE、OP\_CONNECT 和 OP\_ACCEPT，在 NIO 中，由于底层依赖的是 epoll，需要将传入的 op 转换为 epoll 能够理解的操作，例如 SelectionKey.READ -> PollArrayWrapper.POLLIN
    
    转换了类型之后，需要将 Channel 注册到 epoll 上去，这里将注册事件及对应的 Channel 保存到一个 updateList 中，到这里，初始化的工作全部都做好了
    
    于是，执行 `selector.select()` 方法，阻塞等待事件的到来，`select` 方法首先执行 `processDeregisterQueue`，该方法会删除所有 cancel 的 SelectionKey，然后执行 `pollWrapper.poll(timeout)`，poll 方法会遍历 updateList，remove 空闲的 key，然后执行 native 的 epollCtl 方法，然后执行 epollWait 方法，等待 epoll 返回可以操作的 Channel
    
    最后 NIO 将 epoll 的操作转换回用户定义的操作，例如 SelectionKey.READ，这样在代码中，使用 `key.isReadable()` 能够处理
    
    这大概就是 NIO 的执行过程，虽然很简单，但是还是能说明一些东西 
    
* Java volatile 小栗子

    ```java
    int a = 0;
    bool flag = false;

    public void write() {
        a = 2;  // 1
        flag = true;  // 2
    }

    public void multiply() {
        if (flag) {  // 3
            int ret = a * a;  // 4
        }
    }
    ```
    
    当 a 线程调用 write 方法，b 线程调用 multiply 方法的时候，👆的代码有两个问题

    * write 中可能会发生指令重排，`flag = true` 可能先于 `a = 2` 执行
    * `a = 2` 之后，a 的值没有从工作内存刷回主存，multiply 中的 a 还是 0    
    
    因此，这里应该给 flag 变量加上 volatile 关键字，这样根据 happen-before 原则，1 before 2，3 before 4，volatile 原则保证 2 before 3，传递性原则保证 1 before 4

* Java HashMap 默认的装载因子为什么是 0.75，为什么在 table Node 长度为 8 的时候转为红黑树，当 LOAD\_FACTOR 为 0.75 的时候，长度为 8 基本不可能（遵循泊松分布），时间和空间的权衡

    [为什么Map桶中个数超过8才转为红黑树](https://www.cnblogs.com/williamjie/p/11168133.html)

* Java JUC 包思维导图

    ![JUC](/img/in-post/begin-java/juc.jpeg)

* Java Synchronized 的局限性：

    * 当线程尝试获取锁的时候，如果获取不到会一直阻塞
    * 如果获取锁的线程进入休眠或者阻塞，除非当前线程异常，否则其他线程尝试获取锁必须一直等待

* 解决哈希冲突的常用方法：

    * 开放地址法：出现冲突时，以当前哈希值为基础，产生另一个哈希值
    * 再哈希法：同时构造多个不同的哈希函数，发生冲突就换一个哈希方法
    * 链地址法：将哈希地址相同的元素放在一个链表中，然后把这个链表的表头放在哈希表的对应位置
    * 建立公共溢出区：将哈希表分为基本表和溢出表两部分，凡是和基本表发生冲突的元素，一律填入溢出表
* [用通俗的语言说说lock和lockInterruptibly的区别](http://www.dewen.net.cn/q/9077)

* [Java OOM](https://github.com/TangBean/understanding-the-jvm/blob/master/Ch1-Java%E5%86%85%E5%AD%98%E7%AE%A1%E7%90%86%E6%9C%BA%E5%88%B6/01-OOM%E5%BC%82%E5%B8%B8.md)

* [JVM GC](https://github.com/TangBean/understanding-the-jvm/tree/master/Ch1-Java%E5%86%85%E5%AD%98%E7%AE%A1%E7%90%86%E6%9C%BA%E5%88%B6)

* [JVM G1](https://liuzhengyang.github.io/2017/06/07/garbage-first-collector/)

* Java 的反序列化的坑

    * 类里面一定要 serialVersionUID，否则旧数据会反序列化会失败
    * 一旦序列化保存到磁盘操作后，就不要修改类名了，否则旧数据会反序列化会失败
    
   可以尽量把对象转换成 JSON 保存更稳妥

* [Java 8 新特性](https://blog.csdn.net/yitian_66/article/details/81010434)

* Java 中对象一定在堆上分配内存吗
    
    Java 堆中主要保存了对象实例，但是，随着 JIT 编译期的发展与逃逸分析技术逐渐成熟, 栈上分配、标量替换优化技术将会导致一些微妙的变化，所有的对象都分配到堆上也渐渐变得不那么 "绝对" 了

    其实，在编译期间，JIT会对代码做很多优化。其中有一部分优化的目的就是减少内存堆分配压力，其中一种重要的技术叫做逃逸分析。

    如果JIT经过逃逸分析，发现有些对象没有逃逸出方法，那么有可能堆内存分配会被优化成栈内存分配

    ![memory-assign](/img/in-post/begin-java/memory-assign.png)

* Java Arrays.sort 源码

	* 老版本使用插入排序 + 归并排序，当数组中元素小于 7 的时候，使用插入排序，否则使用归并排序
	* 新版本，使用二分排序 + TimSort，当数组中元素小于 32 的时候，使用二分排序，这里的二分排序和常见的有一些不同，会先去得到递增/递减序列，再二分，TimSort 会将数组分成一段一段的（长度从 16 - 31），每一段使用前面说过的（先求递增／递减序列 + 二分）得到一堆有序的段，将段的长度写入一个数组，然后在👇条件下对段进行归并，关于归并方法和对一般的归并排序做出了简单的优化。假设两个 run 是 run1，run2 ，先用 gallopRight在 run1 里使用 binarySearch 查找run2 首元素 的位置k，那么 run1 中 k 前面的元素就是合并后最小的那些元素。然后，在run2 中查找run1 尾元素 的位置 len2，那么run2 中 len2 后面的那些元素就是合并后最大的那些元素。最后，根据len1 与len2 大小，调用mergeLo 或者 mergeHi 将剩余元素合并

		```java
		runLen[i - 3] > runLen[i - 2] + runLen[i - 1] 
		runLen[i - 2] > runLen[i - 1]
		```

* Java 中 a = a + 4 和 a += 4 的区别（a 是 byte 类型）

    a 是 byte 类型，4 默认是 int 类型，所以 a + 4 会转为 int 类型，不能赋值给 byte 类型的 a

    += 在 Java 里会自动转型，不过 4 换成 1111 就溢出了

* Java 字符串操作

    [Java String](https://www.cnblogs.com/yueshutong/p/9520691.html)

    [Java String 类拼接时的编译器优化](https://blog.csdn.net/OOC_ZC/article/details/83999471)

* Java 类中 serialversionuid 作用是什么

	[Java 类中 serialversionuid 作用是什么](https://www.cnblogs.com/duanxz/p/3511695.html)

* Java 8 新特性

    [Java 8 新特性](https://blog.csdn.net/javazyw/article/details/82733459)

* Maven

    * [最强Maven教程](https://zhuanlan.zhihu.com/p/73444114)
    * [Maven settings.xml](https://www.cnblogs.com/hongmoshui/p/10762272.html)  

* Java 中的四种引用

    [java的四种引用，强弱软虚，用到的场景](https://blog.csdn.net/google_huchun/article/details/54311659)
