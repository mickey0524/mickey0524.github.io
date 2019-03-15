---
layout:     post
title:      "interview"
subtitle:   "面试总结"
date:       2019-03-12 22:00:00
author:     "Mickey"
header-img: "img/post-bg-universe.jpg"
tags:
    - 面试
---

最近各家公司都开始暑期实习的招聘了，在这里总结一下，题目来自本人或同学的面试

* 腾讯（后台开发一面）

	1. 了解哈希嘛？说说哈希在密码学中的应用？

		将任意长度的二进制值串映射为固定长度的二进制值串，这个映射的规则就是哈希算法，而通过原始数据映射之后得到的二进制值串就是哈希值
		
		应用：文件校验，数字签名
		
	2. 哈希什么情况下会发生碰撞，怎么样选取哈希的方式能让碰撞几率降低

		其实根据鸽巢原理，哈希是一定会碰撞

		如何减少碰撞概率，我觉得应该可以从三个方面考虑的：
		1. 增大 映射空间/原空间 的大小这个很明显，映射空间/原空间 的比值大，碰撞的概率就可能小了。 前面我们也提到过，当这个比值大于等于1时，就可以实现无碰撞。
		2. 尽可能把原数据集均匀映射到较小空间简单举个例子说明。比如有10个数，映射到5个位置。如果每个位置均匀对应2个数，那么任意选2个数，碰撞概率为0.2.如果1个位置对应6个数，其余4个位置各对应1个数，那么任意选两个数，碰撞概率为0.36
		3. 结合原空间数据的数据特征我们在处理原空间的数据集时，数据集一般都会有特征，而且不会是原空间的全集。换句话说，我们要处理的数据一般只会是原空间所有数据的一部分，而且数据集有一定特征（数据元素出现的概率的不同）。
		
		一般哈希函数用质数取模这就是为了使得有特征的数据集也能均匀映射到映射空间。在这种情况下，有时hash函数还要结合数据特征，让出现概率较大的数据集有较小的碰撞概率，出现概率较小的数据集有较大的碰撞概率。这样，就可以减少整体数据集的碰撞概率。
	
	3. 说说平时用的排序算法的时间复杂度和空间复杂度
		
		![sort](/img/in-post/interview/sort.jpg)
		
		| 排序 | 是原地排序 | 是否稳定 | 最好 | 最坏 | 平均 |
		| ---- | -------- | ------ | --- | ---- | --- |
		| 冒泡 | 是 | 是 | O(n) | O(n^2) | O(n^2) |
		| 插入 | 是 | 是 | O(n) 从尾巴开始比较 | O(n^2) | O(n^2) |
		| 选择 | 是 | 否 | O(n^2) | O(n^2) | O(n^2) |
		| 归并 | 不是| 是 | O(nlogn) | O(nlogn) | O(nlogn)|
		| 快速 | 不是| 不是 | O(nlogn) | O(n^2) | O(nlogn) |
		| 堆 | 是 | 不是 | O(nlogn) | O(nlogn) | O(nlogn) |
		| 计数| 不是 | 是 | O(n + k) | O(n + k) | O(n + k) | 
		| 基数| 不是 | 是 | O(m * n) | O(m * n) | O(m * n) |
 		
	4. 写一个快排给我看

		```python
		def quick_sort(arr, l, r):
			if l < r:
				pivot = partition(arr, l, r)
				quick_sort(arr, l, pivot - 1)
				quick_sort(arr, pivot + 1, r)
			
		def partition(arr, l, r):
			base = arr[r]
			j = l
			
			for i in xrange(l, r):
				if arr[i] < base:
					arr[i], arr[j] = arr[j], arr[i]
					j += 1
			
			arr[j], arr[r] = arr[r], arr[j]
			
			return j
		```
		
	5. 进程间通信方式都有哪些？各自的优缺点是什么？
		
		* 管道
		* FIFO（命名管道）
		* 消息队列
		* 信号量
		* 共享内存
		* 套接字

		[进程通信(见我的博客)](https://github.com/mickey0524/web-development-knowledge/blob/master/docs/os.md)
		
	6. 进程有几种状态，怎么转换的。
		
		created，ready，running，terminated，waiting
		
		![process-state](/img/in-post/interview/process-state.png)
		
	7. 进程什么情况下会死锁，怎么避免。
		
		资源请求图中出现了环
		
		赋予资源的时候，银行家算法去检测
		
	8. 说说TCP，UDP各自的特点，并举一个现实中的例子

		* TCP 是面向连接的，可靠的连接，具有超时重传机制，具有流量控制和拥塞控制，单位是字节流，只能是一对一的（FTP，HTTP）
		* UDP 是无连接的，尽最大可能交付，单位是数据报文，支持一对一，一对多，多对一，多对多（DHCP）
	
	9. 说一下TCP三次握手和四次挥手，并描述一下三次握手四次挥手时服务器和客户端各自的状态
		
		[tcp的11种状态(见我的博客)](https://github.com/mickey0524/web-development-knowledge/blob/master/docs/network.md)
		
	10. TCP怎么保证可靠传输的

		TCP 自带超时重传的机制，当报文段在超时时间内没有被确认，TCP 会重传这个报文，超时时间略大于报文段发送确认的往返时间
			
	11. 讲一讲TCP拥塞控制
	
		慢开始，拥塞避免，快重传，快恢复
		
		[tcp拥塞控制(见我的博客)](https://github.com/mickey0524/web-development-knowledge/blob/master/docs/network.md)
		
	12. 两个链表，如何判断他们是否有交点

		1. 如果可以改 ListNode 的实例（例如 Python），直接遍历其中一个链表，打上 tag，再遍历另外一个即可
		2. 如果不能，也可以用 hash 表的方式存储，比较耗内存
		3. 最后，可以用长短链表的思想，先计算两个链表的长度，然后让长链表先走长度差的步速，然后两个链表每次一步走，相遇了就是有交点
		
	13. 讲一下点击url打开网页的过程都发生了什么
		
		[打开 Web 页面](https://github.com/CyC2018/CS-Notes/blob/master/docs/notes/%E8%AE%A1%E7%AE%97%E6%9C%BA%E7%BD%91%E7%BB%9C%20-%20%E5%BA%94%E7%94%A8%E5%B1%82.md#web-%E9%A1%B5%E9%9D%A2%E8%AF%B7%E6%B1%82%E8%BF%87%E7%A8%8B)
		
	14. 讲一下DNS服务器的实现
	
		[DNS服务器](https://github.com/CyC2018/CS-Notes/blob/master/docs/notes/%E8%AE%A1%E7%AE%97%E6%9C%BA%E7%BD%91%E7%BB%9C%20-%20%E5%BA%94%E7%94%A8%E5%B1%82.md#%E5%9F%9F%E5%90%8D%E7%B3%BB%E7%BB%9F)
	
	15. 你平时用的数据库是什么，讲讲MyISam和innoDB的区别
		
		* MyIsam

			* 高性能读取
			* 因为它保存了表的行数，当使用COUNT统计时不会扫描全表
			* 锁级别为表锁，表锁优点是开销小，加锁快；缺点是锁粒度大，发生锁冲动概率较高，容纳并发能力低，这个引擎适合查询为主的业务
			* 此引擎不支持事务，也不支持外键
			* INSERT和UPDATE操作需要锁定整个表
			* 它存储表的行数，于是SELECT COUNT(*) FROM TABLE时只需要直接读取已经保存好的值而不需要进行全表扫

		* InnoDB

			* 支持事务处理、ACID事务特性
			* 实现了SQL标准的四种隔离级别
			* 支持行级锁和外键约束
			* 可以利用事务日志进行数据恢复
			* 锁级别为行锁，行锁优点是适用于高并发的频繁表修改，高并发是性能优于 MyISAM。缺点是系统消耗较大
			* 索引不仅缓存自身，也缓存数据，相比 MyISAM 需要更大的内存
		
	16. 主键，唯一索引，联合索引区别。

		primary key
		unique key
		union key
		
	17. 如果一个表要做很多count操作，你应该用哪个引擎。
		
		MyIsam，不用扫表就能获取 count 值
		
	18. 如果一个表要频繁做很多insert操作，并且经常查询count，你应该用哪个引擎
	
		只要 insert 很频繁，使用 MyIsam 都不好，因为 MyIsam 的锁粒度是表锁。。
	
	19. 如果一个表要频繁做很多insert，update操作，并且经常会查询count，怎么实现好
		
		将 count 数量放在上层中，更新下层表的时候，利用 MySQL 事务同时去更新上层表
	
	20. 你平时linux用的多吗，常用命令有哪些？查看硬盘状态命令是什么？查看一个进程端口号占用的命令是啥。
	
		常用命令就不写了，毕竟 vim 一大堆
		
		磁盘状态 top/free
		
		进程端口号占用 lsof -i:进程号/ps aux | grep 进程号
	
	21. 写一下判断数学表达式是否正确的程序
		
		```python
		from collections import deque

		def solution(str):
		    str = str.replace(' ', '')
		    str_arr = str.split('=')
		    if len(str_arr) != 2:
		        return False
		
		    l, r = str_arr[0], str_arr[1]
		
		    op_stack = deque()
		    num_stack = deque()
		    ops = '+-*/'
		    i = 0
		    l_len = len(l)
		
		    while i < l_len:
		        ch = l[i]
		        if ch in ops:
		            op_stack.append(ch)
		            i += 1
		        elif '0' <= ch <= '9':
		            j = i + 1
		            while j < l_len and '0' <= l[j] <= '9':
		                j += 1
		            num = int(l[i:j])
		            if len(op_stack) > 0 and (op_stack[-1] == '*' or op_stack[-1] == '/'):
		                tmp = num_stack.pop()
		                res = tmp * num if op_stack[-1] == '*' else tmp / num
		                op_stack.pop()
		                num_stack.append(res)
		            else:
		                num_stack.append(num)
		
		            i = j
		        else:
		            return False
		
		    res = 0
		
		    while len(op_stack) != 0:
		        if len(num_stack) < 2:
		            return False
		        m = num_stack.popleft()
		        n = num_stack.popleft()
		        op = op_stack.popleft()
		        res = (m + n) if op == '+' else (m - n)
		        num_stack.appendleft(res)
		
		    return int(r) == res
		    
		if __name__ == '__main__':
    		print solution('1 + 2 * 3 = 7')
    		print solution('1 + 4 / 2 + 3 + 2 * 4 = 14')
    		print solution('1 + 8 - 4 / 2 + 8 * 9 - 3 = 76')
		```

		