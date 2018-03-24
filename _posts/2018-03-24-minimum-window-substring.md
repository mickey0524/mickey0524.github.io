---
layout:     post
title:      "Minimun-window-substring"
subtitle:   "最小窗口子字符串"
date:       2018-03-24 15:30:00
author:     "Mickey"
header-img: "img/post-sample-image.jpg"
tags:
    - 算法
    - Leetcode
---

记录一道两指针的滑动窗口的题目，难度在leetcode上是hard

```js
Given a string S and a string T, find the minimum window in S which will contain all the characters in T in complexity O(n).

For example,
S = "ADOBECODEBANC"
T = "ABC"
Minimum window is "BANC".

Note:
If there is no such window in S that covers all characters in T, return the empty string "".

If there are multiple such windows, you are guaranteed that there will always be only one unique minimum window in S.
```

题目非常简单，就是求包含T串所有元素的最短子串，如果不存在的话，则返回空

下面分析一下，要求包含T串的所有元素，首先需要统计T串中各元素的个数，因此遍历一遍T字符串，得到t\_obj的字典，然后用satisfy\_num指代T中不同字符的种类，同时我们定义一个deque用于保存S遍历过程中，存在于T中的字符的索引

准备工作做好之后，我们开始遍历S串，当遇到T中的字符key的时候，我们将t\_obj[key]减一，同时将索引append进path，当t\_obj[key]为0的时候，代表key这个字符个数已经够了，我们将satisfy\_num减一，当satisfy\_num为0的时候，我们就找到了一个包含T中所有字符的字串，然后我们开始消重，因为当前我们找到的子串不一定是最短的，可能包含多余的字符，于是我们比较t_obj[s[head]] < 0，通过这种方法来消重，最后进行长度比较，获得结果

通过这种索引跳转的方法，减少了很多次无用比较，在leetcode击败了93%的代码，感觉还不错

```python
from collections import deque

class Solution(object):
    def minWindow(self, s, t):
        """
        :type s: str
        :type t: str
        :rtype: str
        """
        length_s, length_t = len(s), len(t)
        if length_s < length_t or length_s == 0:
          return ''
        if length_t == 0:
          return s[0]
        
        t_obj = {}
        for i in xrange(length_t):
          key = t[i]
          cnt_value = t_obj.get(key, 0)
          t_obj[key] = cnt_value + 1
        
        head, tail, path, res, satisfy_num = 0, 0, deque(), '', len(t_obj.keys())
        
        while tail < length_s:
          k = s[tail]
          if k in t_obj:
            t_obj[k] -= 1
            path.append(tail)
            if t_obj[k] == 0:
              satisfy_num -= 1
              if satisfy_num == 0:
                head = path[0]
                while t_obj[s[head]] < 0:
                  index = path.popleft()
                  t_obj[s[index]] += 1
                  head = path[0]
                if res == '' or len(res) > (tail - head + 1):
                  res = s[head:tail + 1]
                satisfy_num = 1
                t_obj[s[head]] = 1
                path.popleft()
                head = head + 1 if len(path) == 0 else path[0]
                
          tail += 1
          
        return res
      
```