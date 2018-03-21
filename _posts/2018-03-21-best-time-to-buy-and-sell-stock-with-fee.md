---
layout:     post
title:      "Best Time to Buy and Sell Stock with Transaction Fee"
subtitle:   "付费交易股票的最佳时机"
date:       2018-03-21 20:30:00
author:     "Mickey"
header-img: "img/post-bg-kuaidi.jpg"
tags:
    - 算法
    - Leetcode
---

一道贪心法的问题，解法实在巧妙，记录一下

```js
Your are given an array of integers prices, for which the i-th element is the price of a given stock on day i; and a non-negative integer fee representing a transaction fee.

You may complete as many transactions as you like, but you need to pay the transaction fee for each transaction. You may not buy more than 1 share of a stock at a time (ie. you must sell the stock share before you buy again.)

Return the maximum profit you can make.

Example 1:
Input: prices = [1, 3, 2, 8, 4, 9], fee = 2
Output: 8
Explanation: The maximum profit can be achieved by:
Buying at prices[0] = 1
Selling at prices[3] = 8
Buying at prices[4] = 4
Selling at prices[5] = 9
The total profit is ((8 - 1) - 2) + ((9 - 4) - 2) = 8.
Note:

0 < prices.length <= 50000.
0 < prices[i] < 50000.
0 <= fee < 50000.
```

由题易得，还是买卖股票的问题，那么思路也不会有太大的区别，不外乎，逢低买入，逢高出售，这道题目多了一个交易费的概念，就是卖出一次股票，需要支付fee的费用。我解决这道题目的思路是，记录当前遍历到的最小值，当发现能够盈利的时候，也就是prices[i] - min_num - fee > 0的时候，计算一次盈利，有的同学可能会问，那万一后面有更高的价格，现在卖出不是亏了，为了杜绝亏了的情况，计算盈利之后，我们将min_num = prices[i] - fee，这样，如果后面更高的价格 - 2 * fee还能大于0，加起来便是

```python
class Solution(object):
    def maxProfit(self, prices, fee):
        """
        :type prices: List[int]
        :type fee: int
        :rtype: int
        """
        length = len(prices)
        if length == 1:
          return 0
        
        res, min_num = 0, prices[0]
        for i in xrange(1, length):
          if prices[i] < min_num:
            min_num = prices[i]
          else:
            profit = prices[i] - min_num - fee
            if profit > 0:
              res += profit
              min_num = prices[i] - fee
        
        
        return res
```