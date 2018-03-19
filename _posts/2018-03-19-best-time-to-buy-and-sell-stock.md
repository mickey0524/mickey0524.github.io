---
layout:     post
title:      "Best Time to Buy and Sell Stock III"
subtitle:   "交易股票的最佳时机"
date:       2018-03-14 20:30:00
author:     "Mickey"
header-img: "img/post-bg-kuaidi.jpg"
tags:
    - 算法
    - Leetcode
---

记录一道动态规划(DP)的题目，这是一个系列的题目，v1和v2版本的题目较为简单，故没有记录

```
Say you have an array for which the ith element is the price of a given stock on day i.

Design an algorithm to find the maximum profit. You may complete at most two transactions.

Note:
You may not engage in multiple transactions at the same time (ie, you must sell the stock before you buy again).
```
	
由题意得，可以进行两次股票交易（1次包括买入和卖出），那么就可以将整个数组分为左右两个部分，以中间的某一天为分界线，左右两个数组分别交易所赚金额之和的最大值就是我们要的题解，其实有点归并的意思，因此自左向右，自右向左遍历两遍数组，得倒left和right两个结果数组，在遍历一遍取相加的最大值即可
	
```python
class Solution(object):
    def maxProfit(self, prices):
        """
        :type prices: List[int]
        :rtype: int
        """
        length = len(prices)
        if length < 2:
          return 0
        left, right = [0] * length, [0] * length
        
        min_num = prices[0]
        for i in xrange(1, length):
          min_num = min(min_num, prices[i])
          left[i] = max(left[i - 1], prices[i] - min_num)
        
        max_num = prices[length - 1]
        for i in xrange(length - 2, -1, -1):
          max_num = max(max_num, prices[i])
          right[i] = max(right[i + 1], max_num - prices[i])
          
        res = float('-inf')
        for i in xrange(length):
          res = max(res, left[i] + right[i])
        
        return res
        
```

