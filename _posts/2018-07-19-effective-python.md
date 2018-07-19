---
layout:     post
title:      "effective python读后感"
subtitle:   "effective python读后感"
date:       2018-07-19 20:00:00
author:     "Mickey"
header-img: "img/post-bg-ios9-web.jpg"
tags:
    - python
---

这篇blog是[Effective Python](https://book.douban.com/subject/26709315/)的读后感

* 切片不光可以用于取数，也可以用于赋值，而且新的list不用和切片保持长度一致

  ```python
  a = [1, 2, 3]
  a[1:] = [2]
  print(a) # [1, 2]
  ```
* 在单次切片操作内，不要同时指定start、end和stride

  * 既有start和end，又有stride的切割操作，可能会令人非常费解
  * 尽量使用stride为正数，且不带start或end索引的切割操作。尽量避免用负数做stride
  * 如果实在要同时指定start、end和stride，考虑将其拆解为两条赋值语句，其中一条做范围切割，另一条做步进切割，或者使用itertools模块的islice

* 尽量使用推导表达式代替map和filter(可读性不高)，另外，字典与集也支持推导表达式

  ```python
  obj = { 'ghost': 1, 'solid': 2 }
  new_obj = { v: k for k, v in obj.iteritems() }
  my_set = { len(k) for k in new_obj.itervalues() }
  ```

* 用生成器表达式来改写数据量较大的列表推导

  ```python
  value = [len(x) for x in open('/tmp/my_file.txt')]
  print(value)

  >>>
  [100, 57, 15]
  ```

  上面的列表推导需要将全部数据读取到内存中，数据量大的时候，程序容易崩溃

  ```python
  value = (len(x) for x in open('/tmp/my_file.txt'))
  print(value)
  >>>
  <generator object <genexpr> at address>
  
  print next(value) # 100
  ```

  