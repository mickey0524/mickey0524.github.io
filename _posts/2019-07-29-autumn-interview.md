---
layout:     post
title:      "autumn interview"
subtitle:   "秋招面试总结"
date:       2019-07-29 22:00:00
author:     "Mickey"
header-img: "img/post-bg-universe.jpg"
tags:
    - 面试
---

秋招也要陆陆续续的开始了，今天我面试了第一家公司，所以这篇博客来记录一下

* 快手 —— 数据研发工程师（offer）

    因为之前暑期三面全过，因此这次提前批，算是直通终面吧，一轮长达一个半小时的技术面之后，进行了 hr 面试

    * 一个 uid，p_date 的 hive 表，计算 7 留

        ```
        select
            t1.p_date as p_date,
            count(distinct t2.uid) / count(distinct t1.uid) as 7_retain
        from
            t t1 left join t t2 on
            t1.uid = t2.uid and
            datediff(t2.p_date, t1.p_date) >= 1 and
            datediff(t2.p_date, t1.p_date) <= 7
        group by
            t1.p_date;
        ```

    * 一个 referer 的 hive 表，计算 top 10 的 referer

        ```
        select
            referer,
            count(*) as num
        from
            t
        group by
            referer
        order by
            num desc
        limit 10;
        ```

    * 一个 referer，url 的 hive 表，计算 top 10 的跳出 url（需要用 url join referer）

        ```
        select
            t1.url,
            count(*) as num
        from
            t t1 join t t2 on
            t1.url = t2.referer
        group by
            t1.url
        order by
            num desc
        limit 10;
        ```

    * flink 源码相关

        感兴趣的同学可以参考 [flink 流式处理源码](https://github.com/mickey0524/flink-streaming-source-analysis)

    * redis 如何持久化

        AOF + RDB
    
* 阿里 - 数据研发工程师

    * 一面

        * 深入理解 JVM，从头开始背完事了

        * 双线程交替打印字符串

            ```java
            class Thread1 implements Runnable {
                private static final String s = "This is a coding test";
            
                private Semaphore self;
                private boolean first;
            
                public Thread1(Seamphore self, Seamphore next, boolean first) {
                    this.self = self;
                    this.first = first;
                }
            
                public void run() {
                    int length = s.length();
                    int idx = first ? 0 : 1;
                    
                    while (idx < length) {
                        semaphore.acquire();
                        System.out.print(s.charAt(idx));
                        idx += 2;
                        next.release();
                    }
                }
            }
            ```
        
        * n 线程交替打印字符串

            ```java
            class Thread2 implements Runnable {
                private static final String s = "This is a coding test";
                
                private Semaphore self;
                private Semaphore next;
                private int index;
                private int threadNum;

                public Thread2(Semaphore self, Semaphore next, int index, int threadNum) {
                    this.self = self;
                    this.next = next;
                    this.index = index;
                    this.threadNum = threadNum;
                }

                public void run() {
                    int length = s.length();
                    while (index < length) {
                        self.acquire();
                        System.out.print(s.charAt(index));
                        this.index = index + this.threadNum;
                        next.release();
                    }
                }
            }
            ```
    
    * 二面

        * 常见的排序算法，讲出算法复杂度和稳定性

            ![sort](/img/in-post/autumn-interview/sort.jpg)
        
        * 讲一下快速排序的实现

            ```python
            def quick_sort(self, arr, head, tail):
            """
            快排
            type arr: list 待排序的数组
            type head: int 起始索引
            type tail: int 终止索引
            rtype: list
            """

            def partition(arr, head, tail):
                """
                获取快排中用于分割数组的元素的索引
                type arr: list 待排序的数组
                type head: int 起始索引
                type tail: int 终止索引
                rtype: int
                """
                base_num = arr[tail]
                i = head

                for j in xrange(head, tail):
                    if arr[j] < base_num:
                        arr[i], arr[j] = arr[j], arr[i]
                        i += 1

                arr[tail], arr[i] = arr[i], arr[tail]

                return i

            if head < tail:
                middle = partition(arr, head, tail)
                self.quick_sort(arr, head, middle - 1)
                self.quick_sort(arr, middle + 1, tail)
            ```
        
        * hash 表解决 hash 冲突的方法

            1. 开放地址法
            2. 再哈希法
            3. 拉链法
            4. 公共缓存区
        
        * HashMap 的实现方式

            [HashMap 源码分析](https://mickey0524.github.io/2019/06/18/java-hash-map-source/)
        
        * 平衡树

            平衡树是的左右两个子树的高度差的绝对值不超过 1，并且左右两个子树都是一棵平衡二叉树

            常见的平衡树：2 - 3 搜索树，AVL，红黑树
        
        * 操作系统进程和线程

            [进程和线程](https://github.com/mickey0524/web-development-knowledge/blob/master/docs/os.md)

        * JVM 的内存模型

            程序计数器，java 虚拟机栈，本地方法栈，堆，方法区按个解释

        * 进程间通信的方式

            [进程间通信](https://github.com/mickey0524/web-development-knowledge/blob/master/docs/os.md)

        * 常用的 Linux 命令

            ls，cd，top，lsof，netstat，grep，awk，sed
        
        * 讲一下实习经历

            自行发挥

        * 为什么要从 Web 开发转岗大数据

            自行发挥

        * 讲一下 NSQ 的实现

            [NSQ 源码分析](https://github.com/mickey0524/nsq-analysis)

        * JVM GC 算法

            [GC 算法](https://github.com/CyC2018/CS-Notes/blob/master/notes/Java%20%E8%99%9A%E6%8B%9F%E6%9C%BA.md#%E5%9E%83%E5%9C%BE%E6%94%B6%E9%9B%86%E7%AE%97%E6%B3%95)

        * 接口延迟太高，如何排查

            1. 多尝试，是不是稳定复现
            2. 查看是否限流
            3. 查看是不是 Redis key 失效了
            4. 数据库是不是在刷新 insert buffer
            5. SQL 是否没有建索引
            6. Java 频繁触发 full gc，查看是否存在内存泄漏/内存溢出

        * 如何搭建一个数据仓库

            分层搭建
            
            1. dim 层 - 维度层
            2. dwd 层 - 事实层
            3. dwa 层 - 聚合层
            4. app 层 - 应用层
