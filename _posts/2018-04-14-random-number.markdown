---
layout:     post
title:      "random-number"
subtitle:   "随机数问题"
date:       2018-04-14 13:30:00
author:     "Mickey"
header-img: "img/post-bg-universe.jpg"
tags:
    - 算法
---

前几天看到我司的一道面试题，给出一个rand3函数，能够等概率返回1、2、3，要求实现一个rand7函数，能够等概率的返回1-7

初看题目，有两种思路

* 调用rand3的结果除以3，再乘以7，这样得到的结果是7/3，14/3以及7，显然不符合题目要求
* 重复调用rand3的结果7次，再除以3，这样得到的结果为7/3～7，也不符合题目要求


反过来想想，如果是提供rand7函数，要求给出一个rand3函数，就很简单了，因为出现1-7的概率都是1/7，因此只需要在出现1-3的时候返回即可，这给我们提供了一种思路，如果两个rand3的结果相乘，会得倒1-9这九种结果，只需要将1-7的结果赋给rand7即可

```js
[1, 2, 3]
[4, 5, 6]
[7, 0, 0]
```

python实现如下

```python
def rand7():
    i, j = rand3(), rand3()
    if i == 2 and j >= 1:
        return rand7()
    return (i - 1) * 3 + j
```

上面这种方法在求rand9之前都是有效果的，但是超过9的随机数就没有办法表示了，于是，需要想新的方法

其实，还有一种方法，本质和上面的方法差不多，即根据

> 3 * (rand3() - 1) + rand3()

通过这个式子，同样能得到1-9九种可能，python代码如下所示

```python
def rand7():
    num = 3 * (rand3() - 1) + rand3()
    if num > 7:
        return rand7()
    return num % 7 + 1
```

最后给出一个通用性的函数，传入m, n即能获得对应的概率生成函数

```python
def universal_rand(m, n):
    """
    m: 传入的进制数
    n: 需要输出的进制数
    """
    tmp = m * (m - 1)
    max_sum = m + tmp
    level = 1
    
    while max_sum < n:
        tmp *= m
        max_sum += tmp
        level += 1

    threshold = (max_sum / n) * n

    def rand(threshold, m, n, level):
        """
        threshold: 丢弃数据的门槛
        m: 传入的进制数
        n: 需要输出的进制数
        level: 层数
        """
        num = randm()
        tmp = randm() - 1
        for _ in xrange(level):
            tmp *= m
            num += tmp
        if num > threshold:
            return rand(threshold, m, n, level)
        return num % n + 1

    return rand(threshold, m, n, level)
```
