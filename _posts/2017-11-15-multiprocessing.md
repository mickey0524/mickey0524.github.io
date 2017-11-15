---
layout:     post
title:      "multiprocessing"
subtitle:   "正确使用 multiprocessing 的姿势"
date:       2017-11-15 23:00:00
author:     "Mickey"
header-img: "img/post-bg-alitrip.jpg"
tags:
    - python
---

Multiprocessing 是python内置的用于操作多进程的库，由于python有GIL锁的限制，python不能很好的利用多线程，因此，并发操作多用Multiprocessing来实现

我们使用进程池 multiprocessing.pool 来自动管理进程任务。通过以下语句初始化pool:

```python
multiprocessing.freeze_support() # Windows平台要加上这句，避免 RuntimeError
pool = multiprocessing.Pool()
```

假设我们要并行执行的任务是以下函数：

```python
def task(pid):
	# do something
	return result
```

然后在主函数调用：

```python
results = []
total_task = 1000
worker_num = 50
for i in xrange(worker_num):
	result = pool.apply_async(task, args=(i,))
	results += result,
```

`pool.apply_async`采用异步方式调用task，`pool.apply`则是同步方式调用。同步方式意味着下一个 task 需要等待上一个 task 完成后才能开始运行，这显然不是我们想要的功能，所以采用异步方式连续地提交任务。在上面的语句中，我们提交了 4 个任务，假设我的 CPU 是 4 核，那么我的每个核运行一个任务。如果我提交多于 4 个任务，那么每个核就需要同时运行 2 个以上的任务，这回带来任务切换成本，降低了效率。所以我们设置的并行任务数最好等于 CPU 核心数， CPU 核可以通过下面语句得到：

```python
cpus = multiprocessing.cpu_count()
```

接下来我们使用`result.get()`来获取task的返回值：

```python
for result in results:
	print(result.get())
```

在这里不免有人要疑问，为什么不直接在 for 循环中直接 result.get()呢？这是因为pool.apply_async之后的语句都是阻塞执行的，调用 result.get() 会等待上一个任务执行完之后才会分配下一个任务。事实上，获取返回值的过程最好放在进程池回收之后进行，避免阻塞后面的语句。

最后我们使用一下语句回收进程池：

```python
pool.close()
pool.join()
```

最后，给出一份完整的代码

```python
#!/usr/bin/env python

import multiprocessing
import time
from random import *

def task(pid):
    interval = randrange(1, 5)
    time.sleep(interval)
    print pid

def main():
    multiprocessing.freeze_support()
    pool = multiprocessing.Pool()
    cpus = multiprocessing.cpu_count()
    results = []

    #for i in xrange(0, 20):
	#result = pool.apply_async(task, args=(i,))

    pool.map(task, [i for i in xrange(20)])

    pool.close()
    pool.join()


if __name__ == '__main__':
    main()
```