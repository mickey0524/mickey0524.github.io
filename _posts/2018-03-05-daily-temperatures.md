---
layout:     post
title:      "daily-temperatures"
subtitle:   "日常温度计算"
date:       2018-03-05 23:00:00
author:     "Mickey"
header-img: "img/post-bg-js-module.jpg"
tags:
    - 算法
    - Leetcode
---

这篇博客记录一道stack的算法题目，第一次submit的时候ttl了，于是记录一下

给定一个整数数组，找到每个数与顺序往后数第一个大于自己的数的索引差，不存在的话，返回0

```
Example：
	
Given a list of daily temperatures, produce a list that, for each day in the input, tells you how many days you would have to wait until a warmer temperature. If there is no future day for which this is possible, put 0 instead.

For example, given the list temperatures = [73, 74, 75, 71, 69, 72, 76, 73], your output should be [1, 1, 4, 2, 1, 1, 0, 0].
```
	
这道题暴力来做的话，就是遍历每个数直到数组末尾，最差的情况下，复杂度为O(n^2)
	
```python
class Solution(object):
  def dailyTemperatures(self, temperatures):
    """
    :type temperatures: List[int]
    :rtype: List[int]
    """
    t_len = len(temperatures)
    res =  [0] * t_len
    
    for i in xrange(t_len):
      for j in xrange(i + 1, t_len):
        if temperatures[j] > temperatures[i]:
          res[i] = j - i
          break
    
    return res
```
	
上面这种做法，submit后超时，仔细分析一下，这道题用stack的思路来做非常简单，如果当前数大于stack顶部的数值，则出栈，否则，则压栈，使用collections的deque来操作进出栈，提高效率

```python
class Solution(object):
  def dailyTemperatures(self, temperatures):
    """
    :type temperatures: List[int]
    :rtype: List[int]
    """
    t_len = len(temperatures)
    stack, res = collections.deque([]), [0] * t_len
    
    for i in xrange(t_len):
      if len(stack) == 0 or temperatures[i] <= stack[-1][0]:
        stack.append((temperatures[i], i))
        continue
      value, index = stack.pop()
      res[index] = i - index
      while len(stack) > 0 and temperatures[i] > stack[-1][0]:
        value, index = stack.pop()
        res[index] = i - index
      stack.append((temperatures[i], i))
    
    return res
```

