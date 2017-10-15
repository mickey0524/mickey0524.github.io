---
layout:     post
title:      "target-sum"
subtitle:   "计算目标和"
date:       2017-10-15 23:00:00
author:     "Mickey"
header-img: "img/post-bg-e2e-ux.jpg"
tags:
    - 算法
    - Leetcode
---

首先给出这道题的地址[target-sum](https://leetcode.com/problems/target-sum/description/)

这道题的题干如下所示：

You are given a list of non-negative integers, a1, a2, ..., an, and a target, S. Now you have 2 symbols + and -. For each integer, you should choose one from + and - as its new symbol.

Find out how many ways to assign symbols to make sum of integers equal to target S.

```
Input: nums is [1, 1, 1, 1, 1], S is 3. 
Output: 5
Explanation: 

-1+1+1+1+1 = 3
+1-1+1+1+1 = 3
+1+1-1+1+1 = 3
+1+1+1-1+1 = 3
+1+1+1+1-1 = 3

There are 5 ways to assign symbols to make the sum of nums be target 3.
```

题目的意思很清晰，就是给出一个由非负数组成的list，通过在每个数字的前面添加+号或者-号，最终计算得到target，求有多少种符号添加的方式

下面给出解题思路，我认为这是一道动态规划的题目，以第一个数字(用s代指)为例，如果取了-号的话，那么后面的数字计算所得应该为target + s才行，反之，如果取了+号的话，那么后面的数字计算所得应该为target - s，ok，想到这个之后，题目就迎刃而解了，func(index, target) = func(index + 1, target - list[0]) + func(index + 1, target + list[0])

源码如下所示：

```
class Solution(object):
    def findTargetSumWays(self, nums, S):
        """
        :type nums: List[int]
        :type S: int
        :rtype: int
        """
        def reversive(cur_index, s):
            if (cur_index, s) not in cache:
                num = 0
                if cur_index == len(nums):
                    if s == 0:
                        num = 1
                else:
                    num = reversive(cur_index + 1, s + nums[cur_index]) + reversive(cur_index + 1, s - nums[cur_index])
                cache[(cur_index, s)] = num
            return cache[(cur_index, s)] 
                            
        cache = {}
        return reversive(0, S)
        
        
        
```