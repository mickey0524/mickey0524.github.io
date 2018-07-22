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

* python2.7的zip函数并不是生成一个generator，而是遍历完两个list得道元组的组合，可以使用itertools模块的izip方法代替，另外zip方法是以两个list中length小的为基准，如果要以length大的为基准的话，可以使用itertools中的izip_longest函数

* python在闭包函数中无法访问上级函数的基础变量，只能通过list，dict，set或某个类的实例

  ```python
  def sort_priority(numbers, group):
    found = False
    def helper(x):
      if x in group:
        found = True
        return (0, x)
      return (1, x)
    numbers.sort(key=helper)
    return found
  ```

  上述函数，在闭包函数中found设为True，然而修改的不是外层函数的变量，而是在闭包函数的作用域中新定义了一个变量

  ```python
  def sort_priority(numbers, group):
    found = [False]
    def helper(x):
      if x in group:
        found[0] = True
        return (0, x)
      return (1, x)
    numbers.sort(key=helper)
    return found[0]
  ```

* 考虑写生成器来改写直接返回列表的函数

	👇这个函数的功能是得到一个由text中每个单词首字母的索引组成的list
	
	```python
	def index_words(text):
		result = []
		if text:
			result += 0,
		for index, letter in enumerate(text):
			if letter == ' ':
				result += (index + 1),
		return result
	```
	
	可以改成迭代器
	
	```python
	def index_words_iter(text):
		if text:
			yield 0
		for index, letter in enumerate(text):
			if letter == ' ':
				yield index + 1
	```

* 同一个迭代器，在遍历过一遍之后，遍历第二遍是得不到结果的，奇怪的是，也不会报StopIteration的异常，需要记一下，因此如果要多次调用相同的迭代器，需要生成新的迭代器，或者是写一个class，实现`__iter__`方法，返回迭代器

	```python
	def iter():
		yield 1
		yield 2
		yield 3
	
	def traverse_iter(nums):
		print sum(nums)
		
		for num in nums:
			print num
	
	traverse_iter(iter())
	
	# 6
	# 
	```

	可以看到上面函数，第二次遍历的时候就取不到任何东西了
	
	```python
	class MyIter(object):
		def __init__(self):
			pass
		def __iter__(self):
			yield 1
			yield 2
			yield 3
	
	traverse_iter(MyIter())
	
	# 6
	# 1
	# 2
	# 3
	```
	
	小tips，iter()方法可以得到一个collection的迭代器，可以通过iter(nums) is iter(nums)来判断实参是容器还是迭代器，如下
	
	```python
	def traverse_iter(nums):
		if iter(nums) is iter(nums):
			raise TypeError('nums must be a container')
		print sum(nums)
		
		for num in nums:
			print num
	```

* python中函数的参数是可以赋予默认值的，当默认值不为静态数值，而是[], {}或者iife的时候，就会出现各函数调用的时候访问的是同一address的默认值，对于这种情况，我们应该使用None作为默认值，然后在代码中去赋予默认值

	```python
	def decode(data, default={}):
		try:
			return json.loads(data)
		except ValueError:
			return default
	
	foo = decode('bad data')
	foo['stuff'] = 5
	bar = decode('also bad')
	bar['meep'] = 1
	
	# foo {'stuff': 5, 'meep': 1}
 	# bar {'stuff': 5, 'meep': 1}
	```
	
	应该按如下书写
	
	```python
	def decode(data, default=None):
		default = {} if default is None else default
		try:
			return json.loads(data)
		except ValueError:
			return default
	
	foo = decode('bad data')
	foo['stuff'] = 5
	bar = decode('also bad')
	bar['meep'] = 1
	
	# foo {'stuff': 5}
	# bar {'meep': 1}
	```

* python中大多数场景用dict和tuple都可以cover，但是，当场景越来越复杂的时候(比如字典套字典，超过2个元素的元组)，这个时候就需要考虑使用类来抽象数据结构了；如果容器中包含简单而又不可变的数据，那么可以先使用namedtuple来表示