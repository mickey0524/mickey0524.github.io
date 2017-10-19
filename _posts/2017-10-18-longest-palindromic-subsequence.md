---
layout:     post
title:      "Longest Palindromic Subsequence"
subtitle:   "最长回文子序列"
date:       2017-10-18 23:00:00
author:     "Mickey"
header-img: "img/home-bg-o.jpg"
tags:
    - 算法
    - Leetcode
---

首先给出这道题目的地址[longest-palindromic-subsequence](https://leetcode.com/problems/longest-palindromic-subsequence/)

这道题的题干如下所示:

Given a string s, find the longest palindromic subsequence's length in s. You may assume that the maximum length of s is 1000.

```
Input: "bbbab"
Output: 4
One possible longest palindromic subsequence is "bbbb".
```

题目的意思非常清晰，就是给出一个字符串，求该字符串中存在的最长回文子序列，这里需要注意是子序列，不是子串，因此回文串不用连续

* 刚看到这道题目的时候，我觉得这道题可以理解为求s(以下用s代指输入串)和s的逆串的最长公共子序列，代码如下所示

```
class Solution(object):
    def longestPalindromeSubseq(self, s):
        """
        :type s: str
        :rtype: int
        """
        s_len = len(s)
        
        if s_len <= 1:
            return s_len
        
        flag = [[0 for i in xrange(s_len + 1)] for i in xrange(s_len + 1)]
        
        for i in xrange(1, s_len + 1):
            head = s[i - 1]
            for j in xrange(1, s_len + 1):
                tail = s[s_len - j]
                flag[i][j] = max(flag[i - 1][j], flag[i][j - 1])
                if head == tail:
                    flag[i][j] = max(flag[i][j], 1 + flag[i - 1][j - 1])
                    
        return flag[s_len][s_len]
```

我觉得思路上是没有问题的，但是在第67个测试用例的时候TLE了

* 于是重新思考了一遍，觉得没有必要创建一个n^2的数组，这样增加了空间复杂度，原理和上面差不多dp[i][j]代指s[i:j]的最长公共子序列长度，当s[i] == s[j]的时候，dp[i][j] = dp[i + 1][j - 1] + 2，否则，dp[i][j] = max(dp[i + 1][j], dp[i][j - 1])，代码如下所示：<b>注意这段代码里，使用的flag数组的大小，是n * 2，而不是n ^ 2，因为数组当前元素需要和数组中前一个或者后一个元素进行比较，所以需要2个位置，当i不变，j变大的时候，其实后面的flag[i][j]可以覆盖前面的，这就是空间复杂度减小的原因</b>

```
class Solution(object):
    def longestPalindromeSubseq(self, s):
        """
        :type s: str
        :rtype: int
        """
        s_len = len(s)
        
        if s_len <= 1:
            return s_len
        
        flag = [[1] * 2 for i in xrange(s_len)]
        
        for i in xrange(1, s_len):
            for j in reversed(xrange(0, i)):
                if s[i] == s[j]:
                    flag[j][i % 2] = 2 + flag[j + 1][(i - 1) % 2] if i - 1 >= j + 1 else 2
                else:
                    flag[j][i % 2] = max(flag[j][(i - 1) % 2], flag[j + 1][i % 2])
        
        return flag[0][(s_len - 1) % 2]
```

这种思路在第74个用例的时候TLE了

* 在两次TLE之后，决定向其他方向思考一下，换一换思路，这道题可以分解为先找一对首尾相同的字符，然后以这两个字符的index为界限，继续往下寻找，代码如下所示

```
class Solution(object):
    """
    :type s: str
    :rtype: int
    """
    def longestPalindromeSubseq(self, s):
        d = {}
        def f(s):
            if s not in d:
                maxL = 0    
                for c in set(s):
                    i, j = s.find(c), s.rfind(c)
                    maxL = max(maxL, 1 if i == j else 2 + f(s[i + 1 : j]))
                d[s] = maxL
            return d[s]
        return f(s)
```

这次终于AC了~