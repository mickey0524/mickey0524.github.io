---
layout:     post
title:      "numpy-pandas-matplotlib-base"
subtitle:   "numpy，pandas，matplotlib基础"
date:       2018-1-29 18:00:00
author:     "Mickey"
header-img: "img/post-bg-2015.jpg"
tags:
    - python
    - 数据分析
---

入门python之后，作为python的两大招牌，爬虫和数据分析是肯定要去尝试滴，爬虫的话，在我的[python爬虫基础](https://github.com/mickey0524/web-crawler)仓库中，有由简到繁的示例，下面在这篇博客中，记录一些numpy和pandas的基础，以便温故知新~

* numpy(from numpy import \*)

	* a = array([1, 2, 3]) 创建一个np对象
	* a.dtype 查看数据类型
    * a.astype('float64') 改变numpy数组的类型
	* a.shape 一个元组，描述np对象每个维度的长度
	* a.size np对象元素个数
	* a.nbytes np对象消耗的字节数
	* a.ndim np对象的维度数
	* a.T 矩阵对象的转置
	* np.sum(a[, axis = 0, 1]) 求和函数，0为列求和，1位行求和，可以用sum()处理bool数组，计算值为True的数量
    * np.argmax()/np.argmin() 求当前矩阵的最值的第一个索引，如果是bool数组，就是找第一个值为True的索引
	* a.min() 求最小值，同样可以带axis参数
	* a.mean() 求算数平均值，同样可以带axis参数
	* np.median(a) 求中位数
	* a.var() 求方差，同样可以带axis参数
	* a.std() 求标准差，同样可以带axis参数
    * a.cumsum() 求累计和
    * a.cumprod() 求累计积
	* a.round(decimals=1) 取整，可以带decimals参数选择保留几位小数
	* np.zeros((2, 3), dtype='int64') 创建一个全0的np对象，第一个参数为代表维度的元组，dtype可以选择类型，默认的类型是float64
	* np.ones() 参数和zeros一样
    * np.empty((2, 3)) 一般用于创建一个2\*3的空白np矩阵 
    * np.eye(n) 创建一个n\*n对角线为1.的数组
    * np.arange(10, 20, 2) array([10, 12, 14, 16, 18]), 创建连续数组
    * np.reshape((row, col)) np.arange(12).reshape((3, 4)), 改变数据的形状
    * np.linspace(1, 10, 20) 开始端1，结束端10，且分割成20个数据，生成线段
    * np.dot(a, b) 矩阵a和矩阵b进行矩阵乘法，直接对np矩阵进行\*是对矩阵中的每个元素进行乘法，这个需要记一下
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
    * np.random.normal(loc, scale, size=None) randn(size) 等于 normal(loc=0, scale=1, size)
    * np.random.randint(low[, high, size, dtype])
    * np.random.random(size) 返回[0, 1)的size为size的数组
    * numpy中bool比较与操作是 & 或操作是 \|，不是python中and和or，需要记一下
    * 利用bool型索引选取数组中的数据，总是创建数据的副本，而不是切片创建的view，性能上不如view
    * numpy的花式切片，创建的也是数据的副本
    
        ```
        a = np.arange(32).reshape(4, 8)
        a[[3, 1], [2, 4]] # 选出来的是由(3, 2)和(1, 4)元素组成的一位数组
        a[[3, 1]][:, [2, 4]] # 这样选择出来的才是矩形
        a[np.ix_([3, 1], [2, 4])] # 效果和上面一样
        ```
    * np.meshgrid() 接受两个一维数组，返回两个二维数组，代表这两个一维数组的所有点对1
        
        ```
        
        In [12]: p = np.arange(-5, 5)

		In [13]: p
		Out[13]: array([-5, -4, -3, -2, -1,  0,  1,  2,  3,  4])
		
		In [14]: x, y = np.meshgrid(p ,p)
		
		In [15]: x
		Out[15]:
		array([[-5, -4, -3, -2, -1,  0,  1,  2,  3,  4],
		       [-5, -4, -3, -2, -1,  0,  1,  2,  3,  4],
		       [-5, -4, -3, -2, -1,  0,  1,  2,  3,  4],
		       [-5, -4, -3, -2, -1,  0,  1,  2,  3,  4],
		       [-5, -4, -3, -2, -1,  0,  1,  2,  3,  4],
		       [-5, -4, -3, -2, -1,  0,  1,  2,  3,  4],
		       [-5, -4, -3, -2, -1,  0,  1,  2,  3,  4],
		       [-5, -4, -3, -2, -1,  0,  1,  2,  3,  4],
		       [-5, -4, -3, -2, -1,  0,  1,  2,  3,  4],
		       [-5, -4, -3, -2, -1,  0,  1,  2,  3,  4]])
		
		In [16]: y
		Out[16]:
		array([[-5, -5, -5, -5, -5, -5, -5, -5, -5, -5],
		       [-4, -4, -4, -4, -4, -4, -4, -4, -4, -4],
		       [-3, -3, -3, -3, -3, -3, -3, -3, -3, -3],
		       [-2, -2, -2, -2, -2, -2, -2, -2, -2, -2],
		       [-1, -1, -1, -1, -1, -1, -1, -1, -1, -1],
		       [ 0,  0,  0,  0,  0,  0,  0,  0,  0,  0],
		       [ 1,  1,  1,  1,  1,  1,  1,  1,  1,  1],
		       [ 2,  2,  2,  2,  2,  2,  2,  2,  2,  2],
		       [ 3,  3,  3,  3,  3,  3,  3,  3,  3,  3],
		       [ 4,  4,  4,  4,  4,  4,  4,  4,  4,  4]])
       
        ```

    * np.where(cond, xarr, yarr) np.where函数是三元表达式x if condition else y的矢量化版本，当cond中的值为True时，选取xarr的值，否则从yarr中选取，例如np.where(arr > 0, 2, -2)将arr中的正数替换为2，负数替换为-2
    * np.unique(a) 去重，并返回有序结果
    * np.in1d(a, b) 返回一个bool数组，表示a中的元素是否在b中存在
    * np.intersect1d(a, b)/np.union1d(a, b)/setdiff1d(a, b)/setxorid(a, b) 两个集合的交，并，差，对称差操作

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
    * df.isnull() / df.notnull() / df.isna() 用于检测缺失数据
	* df.fillna(value=0) 将df中的nan替换为0
	* df.fillna({'b': 0.5, 'c': 2}) 对df中'b'列的nan替换为0.5，'c'列的nan替换为2
	* df.fillna默认会返回新对象，但也可以对现有对象进行修改
		
		```
		_ = df.fillna(0, inplace=True)
		```
	
	* df.fillna(method="ffill/bfill", limit=2) method代表前向补充，limit代表最多补全多少个
		 
	* pd.concat([df, df1, df2], axis=0/1, ignore\_index=True/False, join='outer/inner', join_axes=[df1.index]) 第一个参数代表要concat的df组成的list，axis代表是行扩展还是列扩展，ignore\_index代表扩展后坐标是否重置，join='outer/inner'代表碰到了df中不存在的坐标"全部显示，不存在坐标显示NaN/剔除，只保存公共的坐标，join\_axes代表遇到不存在的保留哪几个df
	* pd.append() 纵向合并，功能和concat有所重叠
	* pd.merge(df1, df2, on=['key'], how='outer/inner/left/right', suffixes=['\_boy', '\_girl']) pandas中的merge和concat类似，我理解concat多用于合并，而merge和其中文意思一样，是用于数据合并，如果不特别制定，merge是按照columns进行的，df1，df2代表要merge的df，on代表合并的column，how和concat中的join功能类似，代表merge冲突的时候保留的方式，suffixes代表没merge的列中存在名字冲突的时候，列名后加上suffix以区分，未设置的话会自动加上'\_x'和'\_y'以区分
	* pd.merge(df1, df2, left\_index=True, right\_index=True, how='outer/inner') 
    * pandas对象一个重要方法是reindex，其作用是创建一个适应新索引的新对象 obj.reindex(newIndex, fill\_value=0, method="ffill/bfill")，newIndex代表新的索引，fill\_value代表原先不存在的索引用该值填充，ffill代表前向填充，bfill代表后向填充
    * df.drop(['a', 'b'], axis=1) 删除结构中的行/列
    * df.apply(func) 将函数应用到由各列或行所形成的一维数组上

    	```
    	f = lambda x: x.max() - x.min()
    	df = DateFrame(np.random.randn(4, 3), columns=list('abd'))
    	df.apply(f, axis=0/1)
    	```

    * df.applymap(f) df中元素级别的操作
    * series.map() series中元素级别的操作
    * df与series在列上操作，series与df有相同的columns，直接+/-即可，df的每一行对应的columns都会减去对应的数值，如果在行上操作，需要使用算数函数，df.sub(series, axis=0)
    * series.unique() 去重
    * series.value_counts() 计算各值出现的频率
    * series.isin(['b', 'c']) 返回一个bool数组，指代相应位置的元素是否出现在后面的arr中
    * Series和DataFrame可以存在层次索引
    
    	```
    	series = Series([1, 2, 3, 4], index=[[0, 0, 1, 1], [0, 1, 2, 3]])
    	```
    
    * groupby()操作，可以对数据进行分类，然后执行value_counts(), max(), mean()等操作 
    * pd.get_dummies是将拥有不同值的变量转换为0/1的的数值
        
        ```
        xiaoming=pd.DataFrame([1,2,3],index=['yellow','red','blue'],columns=['hat'])
        print(xiaoming)
        hat_ranks=pd.get_dummies(xiaoming['hat'],prefix='hat')
        print(hat_ranks.head())
        
                hat
        yellow    1
        red       2
        blue      3
                hat_1  hat_2  hat_3
        yellow      1      0      0
        red         0      1      0
        blue        0      0      1
        ```

    * DateFrame中修改一列的属性`df['A'] = df['A'].astype('int64')`  
    * df.append([])会报`list indx out of range`的异常
  
* matplotlib(import matplotlib.pyplot as plt)

	* fig, axes = plt.subplots(nrows=2, ncols=3, sharex(所有subplot应该使用相同的X轴刻度), sharey(所有subplot应该使用相同的Y轴刻度)) 创建一个figure，同时返回一个2\*3的axes数组的引用
	* plt.subplots_adjust(left, bottom, right, top, wspace, hspace)，其中wspace和hspace用于控制宽度和高度的百分比
	* ax.plot(x, y, 'ko--')，k代表颜色，--代表虚线，o代表实心点，完整的应该是，plt.plot(x, y, color='k', linestyle='dashed', marker='o', label='one')
	* ax.legend(loc='best') 选择最佳位置进行线的标示（添加图例），`需要plot中的label参数`
	* ax.set_xlim(['1/1/2007', '1/1/2011']) 设置x轴范围
	* ax.set_xticks([0, 250...]) 修改x轴的刻度值
	* ax.set_xticklabels(['one', 'two'...], rotation=30, fontsize='small') 将其他值映射为标签
	* ax.set_title('') 为图标设置标题
	* ax.set_xlabel('Stages') 为x轴设置一个名称
	* Series.plot(label'标签', ax='要在其上进行绘制的matplotlib subplot对象', style='ko--', alpha='0.3'不透明度, kind='line/bar(柱状图)/barh(水平柱状图)/kde(曲线密度图)', rot='旋转刻度标签', xticks='用作X轴刻度的值', yticks='用作Y轴刻度的值', xlim='X轴的界限，如[0, 10]', ylim='y轴的界限')
	* Series.hist(bins=100) 直方密度图
	* plt.scatter(df['A'], df['C']) 简单散步图
	* pd.plotting.scatter_matrix(df, diagonal='kde') 一个df的相关列的关系，对角线可以看趋势，不设diagonal默认为密度直方图
