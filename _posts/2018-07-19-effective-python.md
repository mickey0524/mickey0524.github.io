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

* python中要少用多重继承，如果要复用不同class的方法，建议使用mix-in(个人理解就是不写\_\_init\_\_的class类)

	```python
	class Mixin1(object):
   		def printName(self):
     		print self.name
   
   class Mixin2(object):
   		def printAge(self):
   			print self.age
   	
   	class Stu(Mixin1, Mixin2):
   		def __init__(self, name, age):
   			self.name = name
   			self.age = age
	```
	
* python中应该多用public和protected属性，少用private属性，因为就算是private也不是完全封闭的，一样有方法可以在class外部调用

	```python
	class Stu(object):
		def __init__(self, name):
			self.__name = name
	
	student = Stu('mickey')
	
	student.__name # 'Stu' object has no attribute '__name'
	student._Stu__name # 'mickey'
	```
	
	如上所示，\_\_name在student中变成了\_Stu\_\_name，只要按照格式，private数据都能获取

* python中操作public属性不用和java一样写getter，setter函数，当需要一些处理逻辑的时候，可以用@property关键字，但是@property有一个问题，不能复用处理逻辑，有多少个参数，就需要写多少套@property和@variable_name.setter，在python中，可以用自定义`__set__`和`__get__`来解决这个问题

	```python
	class Grade(object):
		def __init__(self):
			self._value = 0
		def __get__(self, instance, instance_type):
			return self._value
		def __set__(self, instance, value):
			self._value = value
	
	class Exam(object):
		# class attributes
		math_grade = Grade()
		writing_grade = Grade()
	
	exam = Exam()
	exam.writing_grade = 40
	```
	
	当访问python实例的属性时，若属性在实例中不存在，python会将代码转换为
	
	```python
	Exam.__dict__['writing_grade'].__set__(exam, 40)
	```
	
	而获取属性的时候，python也会进行相应的转译
	
	```python
	Exam.__dict__['writing_grade'].__get__(exam, Exam)
	```
	
	之所以会有这样的转译，关键在于object类的`__getattribute__`方法，简单来说，如果实例对应的属性，那么Python就会转向Exam类
	
	这样处理的时候，还有一个问题，类属性在只会在初始的时候实例化一次，这样Exam类的实例都会共用一个数值，可以按如下方式更改，用instance做key存不同的value
	
	```python
	class Grade(object):
		def __init__(self):
			self._values = {}
		
		def __get__(self, instance, instance_type);
			if instance is None: return self
			self._value.get(instance, 0);
		
		def __set__(self, instance, value):
			self._value[instance] = value
	```
	
	这样还是会有一个问题，会导致内存泄漏，当Exam类实例越来越多的时候，_value字典维护的key越来越多且不会释放，使用weakref模块的WeakKeyDictionary可以解决这个问题
	
	```python
	class Grade(object):
		def __init__(self):
			self.values = WeakKeyDictionary()
	```	
	
* python的	`__getattribute__`和`__setattr__`在python每次访问/设置属性的时候，都会调用，因此，如果要在这两个方法中访问实例属性的时候，应该通过`super()`来做，避免无限递归

	```python
	class DictionaryDB(object):
		def __init__(self, data):
			self._data = data
		
		def __getattribute__(self, name):
			data_dict = super().__getattribute__('_data')
			return data_dict[name]
	```
	
	而`__getattr__`方法只有在实例属性和类属性不存在的时候，才会调用
	
* python中的元类

	* 利用元类来验证子类

		```python
		class Meta(type):
			def __new__(meta, name, bases, class_dict):
				print '__new__'
				return type.__new__(meta, name, bases, class_dict)
		
		class Line(object):
			__metaclass__ = Meta
			def __init__(self):
				pass
		
		>>> __new__
		```
	
		python在把子类的class语句处理完毕之后，就会调用其元类的__new__方法，上述代码中
		
		```python
		meta = '__main__.Meta'
		name = 'Line'
		bases = (object,)
		class_dict = {...}类属性
		```
	
	* 元类可以用于在子类生成后进行注册，因为`type.__new__(meta, name, bases, class_dict)`生成的就是子类，可以对class进行一些操作
	* 描述符与元类能够有效的组合起来，以便对某种行为做出修饰，或在程序运行时探查相关信息

		在前面介绍`__set__`和`__get__`方便复用`@property`逻辑的时候，我们在`Grade`类中定义了一个`self.values`，以`instance`为key存储不同实例的属性，同时为了避免内存泄漏，我们引入了`WeakKeyDictionary`，下面我们来看一下将实例的属性存储在实例内的方法
		
		```python
		class Grade(object):
			def __init__(self, name):
				self.name = name
				self.internal_name = '_' + name
			def __set__(self, instance, value):
				setattr(instance, self.internal_name, value)
			def __get__(self, instance, instance_type):
				return getattr(instance, self.internal_name, '')
			
		class Exam(object):
			math_grade = Grade('math_grade')
			writing_grade = Grade('writing_grade')
		```
		
		这样比使用`WeakKeyDictionary`方便了许多，但是，在`Exam`中初始化`Grade`属性的时候，要将`key`的名字书写两边，略显繁琐，不够优雅，可以采用元类来优化这一点，因为在元类的`__new__`的`class_dict`中，是可以拿到类的属性的
		
		```
		class Meta(type):
			def __new__(meta, name, bases, class_dict):
				for k, v in class_dict.iteritems():
					if isinstance(v, Grade):
						v.name = k
						v.internal_name = '_' + k
				cls = type.__new__(meta, name, bases, class_dict)
				return cls
		
		class Grade(object):
			def __init__(self):
				pass
				
		class Exam(object):
			math_grade = Grade()
			writing_grade = Grade()
		```
		
		这样，就不用在Grade中再繁琐的初始化一遍，不过个人觉得最好写注释，不然不好理解
	
* python多线程的意义

	因为受到全局解释器锁(GIL)的限制，所以多条Python线程不能在多个CPU核心上面平行地执行字节码，但是python的多线程功能依旧很有用，它可以轻松地模拟出同一时刻执行多项任务的效果(俗称并发，注意不是并行)，通过Python线程，我们也可以并行的执行多个系统调用，这使得程序能够在执行阻塞式I/O操作的同时，执行一些运算操作，由于Python线程在执行系统调用的时候会释放GIL，并且一直要等到执行完毕才会重新获取它，所以GIL是不会影响系统调用的，下面看一个栗子
	
	```python
	import select
	import time
	
	from threading import Thread
	
	def slow_systemcall():
		select.select([], [], [], 0.1)
		
	start = time()
	for _ in xrange(5):
		slow_systemcall()
	end = time()
	print("Took %.3f seconds" % (end - start))
	
	>>> Took 0.503 seconds
	
	start = time()
	threads = []
	for _ in xrange(5):
		thread = Thread(target=slow_systemcall)
		thread.start()
		threads += thread,
	
	for thread in threads:
		thread.join()
	
	end = time()
	print("Took %.3f seconds" % (end - start))
	
	>>> Took 0.102 seconds
	```
	
* 多线程中的Lock

	python的GIL锁虽然实现了在同一时间只允许一个python线程工作，但是还是需要锁来避免非原子操作带来的错误，`value += offset`其实不是一个原子操作，在多线程中，如果不加锁，这个语句就会出问题
	

	```python
	from threading import Lock
	
	lock = Lock()
	
	# way1
	
	lock.acquire()
	...
	lock.release()
	
	# way2
	with lock:
		...
	```
	
* python2.7中，`multiprocessing`和`concurrent.futures`中的`ProcessPoolExexutor`(底层是`multiprocessing`)来利用多进程并行计算，但是`multiprocessing`的开销较大，原因是因为在主进程和子进程之间，存在序列化和反序列化的操作(字节码 <-> python对象)

* python中的装饰器非常有用，可以用于在程序运行前后执行操作，例如`metric`打点，但是用装饰器wrapper一层之后，python会修改原先函数的内部属性，对调试器和对象序列化等需要使用内省机制的工具来说，会严重干扰，于是，python内置的`functools`模块提供了`wraps`函数(也是一个装饰器，装饰在内置的wrapper函数上)

	```python
	from functools import wraps
	
	def trace(func):
		@wraps(func)
		def wrapper(*args, **kwargs):
			# ...
	return wrapper
	
	@trace
	def true_func(param):
		# ...
	```
	
* python中的with语句非常好用，多用于`threading.Lock`和`文件open`中，python内置的`contextlib`模块提供了一个装饰器叫做`contextmanager`，可以让很多自定义操作使用with语句

	```python
	import logging
	from contextlib import contextmanager
	
	def my_function():
		logger.debug('Some debug data')
		logger.error('Error log here')
		logger.debug('More debug data')
	
	@contextmanager
	def debug_logging(level):
		logger = logging.getLogger()
		old_level = logger.getEffectiveLevel()
		logger.setLevel(level)
		try:
			yield
		finally:
			logger.setLevel(old_level)
	
	with debug_logging(logging.DEBUG):
		print('inside')
		my_function
	print('after')
	my_function()
	
	>>>
	inside:
	some debug data
	error log here
	more debug data
	after:
	error log here
	```
	
	yield表达式所在的地方，就是with块中的语句所要展开执行的地方，另外，我们知道`with open(...) as hander`可以通过with语句获得文件的操作符，`contextmanager`同样也可以，只要在`yield`表达式那里抛出一个对象，就能通过`as`关键字得到
	
	```python
	import logging
	from contextlib import contextmanager
	
	@contextmanager
	def debug_logging(level, logger_name):
		logger = logging.getLogger(logger_name)
		old_level = logger.getEffectiveLevel()
		logger.setLevel(level)
		try:
			yield logger
		finally:
			logger.setLevel(old_level)
	
	with debug_logging(logging.DEBUG, 'my-log') as logger:
		logger.debug('debug info')
		logging.debug('no print')
	```
	
* python中存在两个处理时间的内置模块，time和datetime，python建议尽量使用datetime，因为time模块需要依赖操作系统而运作，该模块的实际行为，取决于底层的c函数如何与宿主操作系统相交互。这种工作方式，使得time模块的功能不够稳定，另外，有关python时间模块，可以参考之前整理的一篇博客[python之time基础](https://mickey0524.github.io/2017/11/17/python-time/)，里面涵盖了大部分业务上的常用操作，包括时区之间相应的转换操作

* 在重视精确度的场合，应该使用decimal

	```python
	rate = 1.45
	seconds = 3 * 60 + 42
	cost = rate * seconds / 60
	print round(cost, 2) # 5.36
	
	rate = Decimal('1.45')
	seconds = Decimal('222')
	cost = rate * seconds / Decimal('60')
	rounded = cost.quantize(Decimal('0.01'), rounding=ROUND_UP)
	print rounded # 5.37
	```

* python系统执行import语句的详细机制

	引入模块的时候，python会按照**深度优先**的顺序执行下列操作
	
	* 在由`sys.path`所指定的路径中，搜寻待引入的模块
	* 从模块中加载代码，并保证这段代码能够正确编译
	* 创建与该模块相对应的空对象
	* 把这个空的模块对象，添加到`sys.modules`里面
	* 运行模块对象中的代码，以定义其内容

* python中的循环引用

	由上一条tip易得，python引入模块先是一个空对象，最后才会去执行模块中的代码，这有可能会造成循环引用
	
	```python
	# dialog.py
	import app
	
	class Dialog(object):
		def __init__(self, save_dir):
			self.save_dir = save_dir
		# ...
	
	save_dialog = Dialog(app.prefs.get('save_dis'))
	
	def show():
		# ...
	```
	
	```python
	# app.py
	
	class Prefs(object):
		# ...
		def get(self, name):
			# ...
	
	prefs = Prefs()
	dialog.show()
	```
	
	像上面这两段代码，就会造成循环引用，解决循环引用的方法
	
	* 把导致两个模块互相依赖的那部分代码，重构为单独的模块，并把它放在依赖树的底层(最佳)
	* 动态引用(最简单，不符合PEP8风格)
	* 先引入、再配置、最后运行(创建，初始化分离，语义不好)

		```python
		# dialog.py
		import app
		
		class Dialog(object):
			# ...
		
		save_dialog = Dialog()
		
		def show():
			# ...
		
		def configure():
			save_dialog.save_dir = app.prefs.get('save_dir')
		```
		
		```python
		# app.py
	
		class Prefs(object):
			# ...
		
		prefs = Prefs()
		def configure():
			# ...
		```
		
		```python
		# main.py
		
		import app
		import dialog
		
		app.configure()
		dialog.configure()
		
		dialog.show()
		```
		
* 在python中打印调试的时候，`print`打印出来的是一个字符串，不能很清晰的显示类型，可以打印`repr(var)`，在类中，同样可以定义`__repr__`方法

	```python
	a, b = 4, '4'
	print a, b # 4, 4
	print repr(a), repr(b) # 4, '4'
	```
	
	其实`repr`和`%r`是一样的