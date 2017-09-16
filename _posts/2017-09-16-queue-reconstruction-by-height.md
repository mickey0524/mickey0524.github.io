---
layout:     post
title:      "queue-reconstruction-by-height"
subtitle:   "按照高度重新排列队列"
date:       2017-09-16 22:00:00
author:     "Mickey"
header-img: "img/post-bg-kuaidi.jpg"
tags:
    - 算法
    - Leetcode
---

以前做leetcode的时候，没有做任何学习笔记，导致很多当时灵光一闪写下的代码后来自己都不记得了，现在决定将一些题目比较好的解法记录下来，这样以后复习的时候也比较轻松

好的，话不多说，这道题目的题干如下所示:

Suppose you have a random list of people standing in a queue. Each person is described by a pair of integers (h, k), where h is the height of the person and k is the number of people in front of this person who have a height greater than or equal to h. Write an algorithm to reconstruct the queue.

Note:
The number of people is less than 1,100.

Example

Input:
[[7,0], [4,4], [7,1], [5,0], [6,1], [5,2]]

Output:
[[5,0], [7,0], [5,2], [6,1], [4,4], [7,1]]

在题目中，每一个人由一个包含两个元素的list[h, k]表示，其中h代表这个人的身高，k代表重新排序之后站在该人前面且身高大于等于这个人身高的人的个数，题目需要我们给出重新排序之后的队列

分析一下，从输入的数据，我们可以得到每一个人的身高，那身高最高的几个人在队列中的排序应该和k值相同，例如，例子中的[7, 0]和[7, 1]，同理，身高为其他数值的人可以自己先排好序，然后向已经存在的队列中进行插入

下面给出python实现的源码

```
class Solution(object):
    def reconstructQueue(self, people):
        """
        :type people: List[List[int]]
        :rtype: List[List[int]]
        """
        if not people:
            return [];
        
        people_obj, height, res  = {}, [], []
        
        for i in xrange(len(people)):
            p = people[i]
            if p[0] in people_obj:
                people_obj[p[0]] += (p[1], i),
            else:
                people_obj[p[0]] = [(p[1], i)]
                height += p[0],
        
        height.sort()
        
        for i in height[::-1]:
            people_obj[i].sort()
            for p in people_obj[i]:
                res.insert(p[0], people[p[1]])
        
        return res
             
            
        
```
