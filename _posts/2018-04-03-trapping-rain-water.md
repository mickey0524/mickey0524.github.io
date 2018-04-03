---
layout:     post
title:      "Trapping Rain Water"
subtitle:   "接最多的雨水"
date:       2018-04-03 18:00:00
author:     "Mickey"
header-img: "img/post-bg-e2e-ux.jpg"
tags:
    - 算法
    - Leetcode
---

今天看面经，看到一道接雨水的题目，记录一下~

```js
Given n non-negative integers representing an elevation map where the width of each bar is 1, compute how much water it is able to trap after raining.

For example, 
Given [0,1,0,2,1,0,1,3,2,1,2,1], return 6.
```

最开始看到这道题的时候，我第一反应是使用stack来存储，如果当前高度比前一个高度低的话，就进栈，反之，出栈，但是这样在进栈的临界条件的判断上会出现问题，后来细想了一下，这道题其实也可以用动态规划来做，记录节点左右的高点，取最低值，然后计算即可，还是相当巧妙的

```python
class Solution(object):
    def trap(self, height):
        """
        :type height: List[int]
        :rtype: int
        """
        length = len(height)
        if length <= 2:
          return 0
        
        left_max, right_max = [0] * length, [0] * length
        left_max[0] = height[0]
        for i in xrange(1, length):
          left_max[i] = max(left_max[i - 1], height[i])
        
        right_max[-1] = height[-1]
        for i in xrange(length - 2, -1, -1):
          right_max[i] = max(right_max[i + 1], height[i])
        
        res = 0
        for i in xrange(1, length - 1):
          res += max(0, min(left_max[i - 1], right_max[i + 1]) - height[i])
        
        return res
          
```


