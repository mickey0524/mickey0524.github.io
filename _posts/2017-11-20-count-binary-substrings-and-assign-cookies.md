---
layout:     post
title:      "count-binary-substrings-and-assign-cookies"
subtitle:   "计算二进制子串和分配饼干"
date:       2017-11-20 20:00:00
author:     "Mickey"
header-img: "img/post-bg-kuaidi.jpg"
tags:
    - 算法
    - Leetcode
---

这篇博客记录两道非常简单的算法题目，然而我在实现的时候都想复杂了，于是记录一下

* 给定一个仅由0和1组成的字符串，求该输入串存在多少个0，1数目相等且0和1连续出现的子串

	Example：
	
	```
	Input: "00110011"
	Output: 6
	Explanation: There are 6 substrings that have equal number of consecutive 1's and 0's: "0011", "01", "1100", "10", "0011", and "01".
	
	Notice that some of these substrings repeat and are counted the number of times they occur.
	
	Also, "00110011" is not a valid substring because all the 0's (and 1's) are not grouped together.
	```
	
	由于输入串只包含0和1，因此题目分析起来很简单，从👆的例子可以看出，连续0和连续1组成的子串中符合要求的个数为min(num(0), num(1))，因此可以将输入串先分成只含0或1子串的数组，然后再两两比较，得出结论，下面给出源代码
	
	```python
	class Solution(object):
   	  def countBinarySubstrings(self, s):
        """
        :type s: str
        :rtype: int
        """
        s = map(len, s.replace('01', '0 1').replace('10', '1 0').split())
        return sum([min(a, b) for a, b in zip(s, s[1:])])
	```

* 第二题是一道简单的贪心法的题目，题目给出两个数组，第一个数组代表每个小孩想要多重的饼干，另外一个数组代表大人有的饼干的重量，问最多能满足多少小孩子的要求

	Example:
	
	```
	Input: [1,2,3], [1,1]
	
	Output: 1
	
	Explanation: You have 3 children and 2 cookies. The greed factors of 3 children are 1, 2, 3. 
	And even though you have 2 cookies, since their size is both 1, you could only make the child whose greed factor is 1 content.
	You need to output 1.
	```
	
	这道题也很简单，先给两个数组都排序，使之按照从小到大排列，然后再依次遍历，先满足要求小的孩子，再依次往上走，最基本的贪心
	
	```
	class Solution(object):
     def findContentChildren(self, g, s):
        """
        :type g: List[int]
        :type s: List[int]
        :rtype: int
        """
        g.sort()
        s.sort()
        
        childi, cookiei = 0, 0
        
        while childi < len(g) and cookiei < len(s):
            if g[childi] <= s[cookiei]:
                childi += 1
            cookiei += 1
            
        return childi
	```

