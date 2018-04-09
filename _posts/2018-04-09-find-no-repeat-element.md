---
layout:     post
title:      "find-no-repeat-element"
subtitle:   "寻找不重复的元素"
date:       2018-04-09 20:00:00
author:     "Mickey"
header-img: "img/post-bg-js-module.jpg"
tags:
    - 算法
    - Leetcode
---

这是一个系列的算法题，主要就是从一堆重复的数字中，寻找没有重复出现的数字

其实这种题目，暴力用一个hashmap去存储出现的次数，然后在遍历一遍hashmap就能找到想要的答案

但是，这类题目，用位运算来说非常方便

### Single Number

这是leetcode上的一道题，[题目地址](https://leetcode.com/problems/single-number/description/), 这是位运算在这类问题中最基本的用法

```js
Given an array of integers, every element appears twice except for one. Find that single one.
```

我们知道，二进制中两个相同的数字异或结果是0，两个不同的数字异或结果是1，那么，数组中，每个相同元素异或之后的结果都是0，然后0之间异或结果也是0，最后就是剩余的单个数字和0异或，因此，只需要将所有的元素通通异或一遍，这样就能找出单独的数字

```python
class Solution(object):
    def singleNumber(self, nums):
        """
        :type nums: List[int]
        :rtype: int
        """
        res = 0
        for num in nums:
          res ^= num
        
        return res
```

### Single Number II

这也是leetcode上的一道题，[题目地址](https://leetcode.com/problems/single-number-ii/description/)，这道题将二进制扩展到了多进制

```js
Given an array of integers, every element appears three times except for one, which appears exactly once. Find that single one.
```

这道题和前面一道题的区别在于，元素不止重复出现了2次，而是3次，其实3次，4次和5次用下面的方法没有任何区别，就是模几的不同罢了

```python
class Solution(object):
    def singleNumber(self, nums):
        """
        :type nums: List[int]
        :rtype: int
        """
        res = 0
        
        for i in xrange(32):
          bit = 1 << i
          cnt_sum = 0
          for num in nums:
            if num & bit:
              cnt_sum += 1
          cnt_sum %= 3
          cnt_sum <<= i
          res |= cnt_sum
        
        if res >= 2 ** 31:
          res -= 2 ** 32
        
        return res
```

### Single Number III

这是leetcode上 Single Number 系列的第三题，讲的是一堆数中除了两个数出现了一次，其他的数均出现了两次，题目要求我们找出这两个数字

```js
Given an array of numbers nums, in which exactly two elements appear only once and all the other elements appear exactly twice. Find the two elements that appear only once.

For example:

Given nums = [1, 2, 1, 3, 2, 5], return [3, 5].
```

这道题和前面不同的地方在于，存在两个单数，然而如果直接和第一道题一样的全部异或，会得到两个单数异或的结果，这样并不能区分开来，然后我们仔细想一想，得到两个数异或的结果，我们能知道这两个数在每一位上是否相同，通过这种方式，我们能够将这堆数分为两类，然后两堆数分别异或，就能得到最终的答案

```python
class Solution(object):
    def singleNumber(self, nums):
        """
        :type nums: List[int]
        :rtype: List[int]
        """
        if len(nums) == 0:
          return [0, 0]
        
        xor_sum = 0
        for num in nums:
          xor_sum ^= num
        
        diff_bit = 1
        while diff_bit & xor_sum == 0:
          diff_bit <<= 1
        
        res_1, res_2 = 0, 0
        for num in nums:
          if num & diff_bit == 0:
            res_1 ^= num
          else:
            res_2 ^= num
        
        return [res_1, res_2]
```

### 总结

通过这三道题，让我对位运算有了更深的理解～