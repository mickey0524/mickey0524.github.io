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

    7.29 直通终面 + hr面，8.9 oc

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

    8.6 一面，8.7 二面，8.12 三面，8.14 四面

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
    
    * 三面

        * 介绍一下实习

            自行发挥

        * 介绍一下自己如何学习的

            自行发挥

        * Nsq 源码介绍一下

            [我的 Nsq 源码解析](https://github.com/mickey0524/nsq-analysis)

        * Flink 源码介绍一下

            [我的 Flink 源码解析](https://github.com/mickey0524/flink-streaming-source-analysis)

        * Vue 源码介绍一下

            [我的 Vue 源码解析](https://mickey0524.github.io/2018/04/21/vue-source-code-analysic-v1/)

        * 之后的打算，计划

            自行发挥
        
    * 四面

        * 介绍一下实习

            自行发挥

        * HBase 架构

            [我整理的 HBase](https://github.com/mickey0524/big-data-knowledge#hbase)
        
        * Flink 介绍一下

            [我的 Flink 源码解析](https://github.com/mickey0524/flink-streaming-source-analysis)
        
        * 进程间通信的方式

            [进程间通信](https://github.com/mickey0524/web-development-knowledge/blob/master/docs/os.md)
        
        * TCP 11 种状态

            [TCP](https://github.com/mickey0524/web-development-knowledge/blob/master/docs/network.md)
        
        * ArrayBlockingQueue 源码

            [BlockingQueue 源码](https://mickey0524.github.io/2019/07/01/blocking-queue-source/)
        
        * 1000 个苹果，放到 10 个篮子，如何分配，能够得到 1 - 1000 中任意的苹果数目

            二进制分法，1，2，4...

        * 为什么要出来面试

            自行发挥
    
    * hr 面

        * 大学如何学习的

            自行发挥

        * 保研还是考验

            保研
        
        * 比较一下头条和阿里两个公司

            自行发挥

        * 未来什么规划

            自行发挥

        * 有没有女朋友

            自行发挥


* 百度 - 大数据开发工程师

    8.10 一面 - 三面

    * 一面

        * HDFS 原理

            [我整理的 HDFS](https://github.com/mickey0524/big-data-knowledge#hdfs)

        * YARN 原理

            [我整理的 YARN](https://github.com/mickey0524/big-data-knowledge#yarn)

        * MR 原理

            [我整理的 MR](https://github.com/mickey0524/big-data-knowledge#mapreduce)

        * Spark 原理

            [我整理的 Spark](https://github.com/mickey0524/big-data-knowledge#mapreduce)

        * 线程、进程

            线程是 CPU 调度的最小单位，进程是内存分配的最小单位

        * CPU 如何进行时间片轮转

            硬件层面，通过时间中断实现的

        * 红黑树

            [红黑树](https://github.com/CyC2018/CS-Notes/blob/master/docs/notes/%E7%AE%97%E6%B3%95%20-%20%E7%AC%A6%E5%8F%B7%E8%A1%A8.md#%E7%BA%A2%E9%BB%91%E6%A0%91)
        
        * Java 反射，动态代理

            [我的 Java 总结](https://mickey0524.github.io/2019/03/21/begin-java/)
        
        * 快速排序

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

        * 二分查找

            ```python
            def binary_search(arr, target):
                l, r, m = 0, len(arr) - 1, -1

                while l <= r:
                    m = l + (r - l) / 2
                    if target == arr[m]:
                        return True
                    elif target > arr[m]:
                        l = m + 1
                    else:
                        r = m - 1

                return False
            ```     
        
        * 设备连接路由表，路由表对外的 IP 是相同的，如何区分是哪个设备

            路由表中存在映射，端口号 -> 真实 ip
        
        * Hive 中的元数据存储

            首先需要搞清楚，Hive 中数据存储的位置，元数据（对数据的描述，包括表，表的列及其它各种属性）是存储在 MySQL 等数据库中的，因为这些数据要不断的更新、修改，不适合存储在 HDFS 中

        * 判断链表有环的快慢指针的证明

            [证明](https://blog.csdn.net/jnxxhzz/article/details/82773112)

    * 二面

        * HBase 架构

            [我整理的 HBase](https://github.com/mickey0524/big-data-knowledge#hbase)

        * MySQL B+ 树，MySQL 四种隔离级别，MVCC

            [我整理的 MySQL](https://github.com/mickey0524/web-development-knowledge/blob/master/docs/db.md)

        * MySQL 查询计划生成以及优化

            [MySQL 查询优化](https://www.cnblogs.com/williamjie/p/9132390.html)

        * Redis 数据淘汰策略

            [Redis 数据淘汰](https://github.com/CyC2018/CS-Notes/blob/master/notes/Redis.md#%E4%B8%83%E6%95%B0%E6%8D%AE%E6%B7%98%E6%B1%B0%E7%AD%96%E7%95%A5)
        
        * Redis 持久化

            RDB + AOF
        
        * Nsq 介绍一下

            [我的 Nsq 源码解析](https://github.com/mickey0524/nsq-analysis)

        * Flink 介绍一下

            [我的 Flink 源码解析](https://github.com/mickey0524/flink-streaming-source-analysis)

    * 三面

        三面聊人生，理想，以及面试官对百度云未来的展望，面试官介绍了国内的云市场，以及其他厂商的不足，学习了很多

* 头条 - 大数据工程师，转正加面

    8.11 转正加面

    * 介绍自己做的事情

        自行发挥

    * 介绍为什么要从 Web 转岗大数据

        自行发挥
    
    * Flink 原理，越详细越好

        [我的 Flink 源码解析](https://github.com/mickey0524/flink-streaming-source-analysis)

    * 算法题

        Leetcode 第四题，自行 google

* 猿辅导 - 数据研发工程师

    8.12 一面 - 三面

    * 一面

        * 两个 字符串，str1 和 str2，计算 str1 - str2 的值，不能转为整型进行计算

            ```python
            def solution(str1, str2):
                if str1 == str2:
                    return '0'

                len1, len2 = len(str1), len(str2)
                symbol = 1

                if len1 == len2:
                    for i in xrange(len1):
                        if str1[i] < str2[i]:
                            symbol = -1
                            str1, str2 = str2, str1
                            break

                elif len1 < len2:
                    symbol = -1
                    len1, len2, str1, str2 = len2, len1, str2, str1

                carry = False
                res = []

                for i in xrange(max(len1, len2)):
                    ch1 = int(str1[len1 - 1 - i])
                    if carry:
                        ch1 -= 1

                    if i < len2:
                        ch2 = int(str2[len2 - 1 - i])
                        ch1 -= ch2

                    if ch1 < 0:
                        ch1 += 10
                        carry = True
                    else:
                        carry = False

                    res += str(ch1),

                ans = res[::-1]
                while ans and ans[0] == '0':
                    ans.pop(0)

                return ''.join(ans) if symbol == 1 else '-' + ''.join(ans)
            ```

        * 最长摆动序列，Leetcode 376

            ```python
            def wiggleMaxLength(self, nums):
                """
                :type nums: List[int]
                :rtype: int
                """
                length = len(nums)
                
                if length < 2:
                    return length
                
                dp = [[0] * 2 for _ in xrange(length)]
                dp[0] = [1, 1]
                
                for i in xrange(1, length):
                    if nums[i] > nums[i - 1]:
                        dp[i][0] += dp[i - 1][1] + 1
                        dp[i][1] = dp[i - 1][1]
                    elif nums[i] < nums[i - 1]:
                        dp[i][1] += dp[i - 1][0] + 1
                        dp[i][0] = dp[i - 1][0]
                    else:
                        dp[i] = dp[i - 1]
                
                return max(dp[-1])
            ```

    * 二面

        * 实习和项目经历

            自行发挥

    * 三面

        * 零钱问题，Leetcode 518

            ```python
            def solution(coins, target):
                dp = [0] * (target + 1)
                dp[0] = 1

                for c in coins:
                    for i in xrange(c, target + 1):
                        dp[i] += dp[i - c]
                
                return dp[target]
            ```

        * Flink 原理

            [我的 Flink 源码解析](https://github.com/mickey0524/flink-streaming-source-analysis)
        
        * 对大数据的理解

            大数据类的工作可以分为三种，inf 方向，数据平台方向，以及数据研发方向

        * 如何实时同步 MySQL -> HDFS

            MySQL binlog -> Kafka -> 服务消费 Kafka，将更新写到 HDFS 上，然后根据 timestamp 来区分新数据
