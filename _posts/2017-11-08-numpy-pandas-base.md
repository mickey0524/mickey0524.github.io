---
layout:     post
title:      "numpy-pandas-base"
subtitle:   "numpy，pandas基础"
date:       2018-1-11 18:00:00
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
	* a.T 矩阵对象的转置
	* sum(a[, axis = 0, 1]) 求和函数，0为列求和，1位行求和
	* a.min() 求最小值，同样可以带axis参数
	* a.mean() 求算数平均值，同样可以带axis参数
	* a.median() 求中位数
	* a.var() 求方差，同样可以带axis参数
	* a.std() 求标准差，同样可以带axis参数
	* a.round(decimals=1) 取整，可以带decimals参数选择保留几位小数
	* np.zeros((2, 3), dtype='int64') 创建一个全0的np对象，第一个参数为代表维度的元组，dtype可以选择类型，默认的类型是float64
	* np.ones() 参数和zeros一样
    * np.arange(10, 20, 2) array([10, 12, 14, 16, 18]), 创建连续数组
    * np.reshape((row, col)) np.arange(12).reshape((3, 4)), 改变数据的形状
    * np.linspace(1, 10, 20) 开始端1，结束端10，且分割成20个数据，生成线段
    * np.dot(a, b) 矩阵a和矩阵b进行矩阵乘法，直接对np矩阵进行*是对矩阵中的每个元素进行乘法，这个需要记一下
    * np.clip(a, 5, 9) 将a中比5小的元素替换为5，比9大的元素替换为9
    * a.flatten() 获得a的一维数组形式，是copy的返回
    * a.flat 迭代器，类似于xrange
    * a.ravel() 获得a的一维数组形式，是view的返回
    * np.concatenate((a, a, a), axis = 0/1) 合并数组，axis控制方向，比hstack，vstack快
    * np.split(a, 2, axis=1) 纵向分割
    * np.split(a, 3, axis=0) 横向分割
    * 上面的分割api如果不能等数目分割，则会报错，np.array_split()则不会，参数和上面一样
    * b = a 浅复制
    * b = a.copy() 深复制
    * np.random.rand(a, b，c) 从[0, 1)中返回三维数组的随机数值 
    * np.random.randn() 参数和前面一样，从标准正态分布中返回随机数值
    * np.random.randint(low[, high, size, dtype])
    * np.random.random(size) 返回[0, 1)的size为size的数组

* pandas(import pandas as pd; import numpy as np)

	* pd.Series([1, 2, 3]) 创建一维Series
	* pd.DataFrame(np.random.randn(6, 4), index=np.arange(6), columns=['a', b', 'c', 'd']) 创建二维DateFrame，index是横轴标签，columns是纵轴标签
	* DataFrame的每列的数据类型可以是不一样的
	
		```
		df = pd.DataFrame({'A' : 1.,
                    'B' : pd.Timestamp('20130102'),
                    'C' : pd.Series(1,index=list(range(4)),dtype='float32'),
                    'D' : np.array([3] * 4,dtype='int32'),
                    'E' : pd.Categorical(["test","train","test","train"]),
                    'F' : 'foo'})	
		```
	
	* df.sort_index(axis=0/1, ascending=True/False) 按照横轴或纵轴进行排序，可以是升序或者降序
	* df.sort_values(by='B') 按照纵轴对应列的数值进行排序
	* df['a'] 选择一列的数据
	* df.loc[][] 按照标签进行选择
	* df.iloc[][] 按照位置/索引进行选择
	* df.dropna(axis=0/1, how='any'/'all') 对行/列操作，any代表只要存在NaN就drop掉，all代表全部是NaN才drop
	* df.fillna(value=0) 将df中的nan替换为0
	* pd.concat([df, df1, df2], axis=0/1, ignore\_index=True/False, join='outer/inner', join_axes=[df1.index]) 第一个参数代表要concat的df组成的list，axis代表是行扩展还是列扩展，ignore\_index代表扩展后坐标是否重置，join='outer/inner'代表碰到了df中不存在的坐标"全部显示，不存在坐标显示NaN/剔除，只保存公共的坐标，join\_axes代表遇到不存在的保留哪几个df
	* pd.append() 纵向合并，功能和concat有所重叠
	* pd.merge(df1, df2, on=['key'], how='outer/inner/left/right', suffixes=['\_boy', '_girl']) pandas中的merge和concat类似，我理解concat多用于合并，而merge和其中文意思一样，是用于数据合并，如果不特别制定，merge是按照columns进行的，df1，df2代表要merge的df，on代表合并的column，how和concat中的join功能类似，代表merge冲突的时候保留的方式，suffixes代表没merge的列中存在名字冲突的时候，列名后加上suffix以区分，未设置的话会自动加上'\_x'和'\_y'以区分
	* pd.merge(df1, df2, left_index=True, right_index=True, how='outer/inner') 代表按照index进行merge
