---
layout:     post
title:      "numpy-pandas-base"
subtitle:   "numpy，pandas基础"
date:       2017-11-08 18:00:00
author:     "Mickey"
header-img: "img/post-bg-2015.jpg"
tags:
    - python
    - 数据分析
---

入门python之后，作为python的两大招牌，爬虫和数据分析是肯定要去尝试滴，爬虫的话，在我的[python爬虫基础](https://github.com/mickey0524/web-crawler)仓库中，有由简到繁的示例，下面在这篇博客中，记录一些numpy和pandas的基础，以便温故知新~

* numpy(from numpy import *)

	* a = array([1, 2, 3]) 创建一个np对象
	* a.dtype 查看数据类型
	* a.shape 一个元组，描述np对象每个维度的长度
	* a.size np对象元素个数
	* a.nbytes np对象消耗的字节数
	* a.dims np对象的维度数
	* sum(a[, axis = 0, 1]) 求和函数，0为列求和，1位行求和
	* a.min() 求最小值，同样可以带axis参数
	* a.mean() 求算数平均值，同样可以带axis参数
	* a.var() 求方差，同样可以带axis参数
	* a.std() 求标准差，同样可以带axis参数
	* a.round(decimals=1) 取整，可以带decimals参数选择保留几位小数
	* np.zeros((2, 3), dtype='int64') 创建一个全0的np对象，第一个参数为代表维度的元组，dtype可以选择类型，默认的类型是float64
	* np.ones() 参数和zeros一样
