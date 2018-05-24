---
layout:     post
title:      "Number of Matching Subsequences"
subtitle:   "子序列总数"
date:       2018-05-24 23:00:00
author:     "Mickey"
header-img: "img/post-bg-js-module.jpg"
tags:
    - 算法
    - Leetcode
---

今天是自己的生日，先对自己说一声生日快乐吧🎂，希望新的一岁天天开心，身体健康，码力增强

许久未写算法题，今天开了一道有关子序列的题目，题目给出一个长字符串S以及一个字符串数组words，求words中有多少元素是S串的子序列，[题目链接](https://leetcode.com/problems/number-of-matching-subsequences/description/)

```python
Given string S and a dictionary of words words, find the number of words[i] that is a subsequence of S.

Example :
Input: 
S = "abcde"
words = ["a", "bb", "acd", "ace"]
Output: 3
Explanation: There are three words in words that are a subsequence of S: "a", "acd", "ace".
```

## 暴力遍历

这道题暴力遍历很容易想到，就不介绍思路了，代码如下所示:

```python
class Solution(object):
    def numMatchingSubseq(self, S, words):
        """
        :type S: str
        :type words: List[str]
        :rtype: int
        """
        res, length = 0, len(S)
        
        for word in words:
          s_idx = 0
          w_idx = 0
          word_len = len(word)
          while s_idx < length and w_idx < word_len:
            if S[s_idx] == word[w_idx]:
              w_idx += 1
            s_idx += 1
          if w_idx == word_len:
            res += 1
        
        return res
          
```

## Trie树

显而易见，这种题目，暴力遍历很容易超时，于是我想到了Trie的方法，将word中的一个个字符用Trie树的节点来进行存储，如果当前word的ch在树中存储过的话，直接往下方子节点走即可，建立节点的时候，需要初始化传入一个idx，代表当前ch位于S中的哪个下标，这样，在Trie中不存在而需要遍历查找的时候，直接用当前node的idx作为启始索引即可

```python
class TrieTree(object):
  def __init__(self, idx):
    self.children = [None] * 26
    self.idx = idx

class Solution(object):
    def numMatchingSubseq(self, S, words):
        """
        :type S: str
        :type words: List[str]
        :rtype: int
        """
        length = len(S)
        res = 0
        root = TrieTree(-1)
        
        for word in words:
          node = root
          for ch_idx, ch in enumerate(word):
            num = ord(ch) - 97
            is_found = False
            if node.children[num]:
              node = node.children[num]
              is_found = True
            else:
              cnt_idx = node.idx
              for i in xrange(cnt_idx + 1, length):
                if S[i] == ch:
                  node.children[num] = TrieTree(i)
                  node = node.children[num]
                  is_found = True
                  break
            if not is_found:
              break
            if ch_idx == len(word) - 1:
              res += 1
          
        return res
```

## 数组存储的方法

思路依旧很简单，先遍历一遍S字符串，生成obj对象，key，value分别为小写字母和在S中出现的下标正序组成的list，对words中的word进行遍历的时候，只需要比较当前idx和数组中idx的大小，就可以知道是否为子序列

```python
from collections import defaultdict

class Solution(object):
    def numMatchingSubseq(self, S, words):
        """
        :type S: str
        :type words: List[str]
        :rtype: int
        """
        obj, res = defaultdict(list), 0
        for idx, ch in enumerate(S):
          obj[ch] += idx,
        
        for word in words:
          cnt_idx = -1
          for ch_idx, ch in enumerate(word):
            is_found = False
            for idx in obj[ch]:
              if idx > cnt_idx:
                cnt_idx = idx
                is_found = True
                break
            if not is_found:
              break
            if ch_idx == len(word) - 1:
              res += 1
        
        return res
```
