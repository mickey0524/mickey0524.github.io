---
layout:     post
title:      "begin-python"
subtitle:   "python小白入门"
date:       2017-08-05 20:00:00
author:     "Mickey"
header-img: "img/post-bg-universe.jpg"
tags:
    - python
---

一直觉得前端工程师不能完全拘泥于前端开发，应该多关注关注后端领域，毕竟作为http的两端，缺一不可，233333，正好公司后端用的语言是python，加之想学习python数据抓取配合前端进行数据可视化的开发，于是义无反顾的开始学习啦，本博客的内容来自[《python核心编程》](https://book.douban.com/subject/3112503/)和[《廖雪峰python》](https://www.liaoxuefeng.com/wiki/001374738125095c955c1e6d8bb493182103fac9270762a000)，感谢🙏
<br>
<br>------------
<br>
<br><b>begin-python</b>
<br>
<br>下面列出一些python的知识点，总结一下，用来温故知新(本博客所用python版本为2.7)
<br>1. python中 / 为浮点除，//为地板除, /只要有一个参数为浮点数，那么结果就是浮点数，//无论参数如何返回的都是整数
想要让/ 和 //分工明确，可以和引入，`import division from __future__`，这样/返回的都是浮点数，//返回的都是整数

<br>2. python变量键入的写法和c有些相似

```js
print "my name is %s" % baihao
print "my name is %s, my age is %d" % (baihao, age)
```
%r代表不管什么都打印出来
如果你使用了非ASCII字符而且碰到了编码错误，记得在最顶端加一行

```js
#!/usr/bin/env python
# -*- coding: utf-8 -*-
```
第1行注释可以让这个hello.py文件直接在Unix/Linux/Mac上运行，第2行注释表示.py文件本身使用标准UTF-8编码
<br>3. 打印多行字符串，用两个```包裹起来即可

<br>4. python 中的list（列表）和 tuple（元组）list是可变的，tuple是不可变的，需要注意的是
a = (1)  这样a会被定义为1，一个元素的tuple写法为(1,)

<br>5. python的列表也支持slice操作，取一个列表的前三个 L[0:3]，tuple也是list，只是不可改变，tuple也可以使用切片

<br>6. python字典的迭代，d = {'a': 1, 'b': 2, 'c': 3}，for in 遍历的只是key，如果要遍历value for value in itervalues()
，两个都遍历的话 for k, v in iteritems()

<br>7. 列表的迭代方法为for in，这样没法知道元素的索引，如果想要获取index的话，可以像下面这样
```js
a = [1, 2, 3];
for k, v in enumerate(a):
  print k, v
```
<br>8. python 中strip()等于js中trim()

<br>9. map，reduce，filter，sorted都是高阶函数，区别是前三个的函数参数是第一个，sorted的函数参数是第二个

<br>10. lambda 匿名函数，简化代码量

<br>11. 在函数执行过程中动态增加功能的方式称为装饰器，decorator本质上也是高阶函数，接受需要动态添加功能的函数作为一个参数
<a href="https://www.liaoxuefeng.com/wiki/001374738125095c955c1e6d8bb493182103fac9270762a000/001386819879946007bbf6ad052463ab18034f0254bf355000" class=" wrap external" target="_blank" rel="nofollow noreferrer">初识装饰器</a>

装饰器的数学定义其实就是 (g · f)(x) = g(f(x))

```
@deco2
@deco1
def func(arg1, arg2, ...): pass
func = deco2(deco1(func))
```

<br>12. isinstance() < = > instanceof 同样用来判断类和引用类型（js中） type() < = > typeof  同样用来判断基本类型

<br>13. python可以通过hasattr(), setattr(), getattr()来操作对象实例，有点类似DOM了，同样和字典一样，通过get()操作对象可能不存在，可以返回一个默认值

<br>14. python作为一门动态语言，能够在程序运行过程中对类和实例对象绑定方法，贼强，这里需要从types里引入MethodType

Class.way = MethodType(way,None, Student) 类

instance.way = MethodType(way, instance, Student) 实例

```
from types import MethodType

class Stu(object):
	pass

def set_name(self, name):
	self.name = name

def set_age(self, age):
	self.age = age

Stu.set_name = MethodType(set_name, None, Stu) # 为class动态添加方法

bob = Stu()

bob.set_age = MethodType(set_age, bob, Stu) # 为实例动态添加方法
```

<br>15. python类可以通过设置__slots__来限制类能被设置的属性，需要注意的是子类继承父类如果不定义__slots__属性的话，是没有限制的，子类如果定义了的话，它的可编辑属性为自己和父亲的slots中的值之和

```
class Stu(object):
	__slots__ = ('name', 'age')
	
>>> s = Student() # 创建新的实例
>>> s.name = 'Michael' # 绑定属性'name'
>>> s.age = 25 # 绑定属性'age'
>>> s.score = 99 # 绑定属性'score'
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
AttributeError: 'Student' object has no attribute 'score'
```

<br>16. 通过set和get来操作类的属性有简写方法，通过python的装饰器

众所周知，python中，可以通过instance.property的方式直接访问属性，但是这样没法对新的数据做数值的判断，于是有了如下的装饰器(默认不能直接访问实例._property的内部属性，不然property毫无意义)

```js
  class Student(object):
    @property
    def birth(self):
      return self._birth
    @birth.setter
    def birth(self, value):
      self._birth = value
    @property
    def age(self):
      return 2014 - self._birth
```
@property，@属性.setter，这样实例就能和操作属性一样操作方法了，s.birth，s.birth = 'XXX'

<br>17. python里面\_\_XXX\_\_这样命名的属性都是有hin大作用的，前面我们已经知道了\_\_slots\_\_，在python里面print 实例或者直接输出实例是
<\_\_main\_\_.Student object at 0x109afb310>，不好看，可以在class 里面定义\_\_str\_\_和\_\_repr\_\_，\_\_str\_\_定义print 返回的语句，\_\_repr\_\_定义在命令行直接输入显示的语句，偷懒的写法是\_\_repr\_\_ = \_
\_str\_\_，这样只定义\_\_str\_\_就行了
\_\_getattr\_\_：当python在类属性中寻找不到的话，python解释器会自动调用\_\_getattr\_\_方法，我们可以在\_\_getattr\_\_中做一些限制
\_\_call\_\_: 实例初始化后，如果调用自身则访问该函数

```js
class Student(object):
  pass
s = Student()
s() => 调用__call__方法
```
可以通过callable()方法来判断一个类是否包含\_\_call\_\_方法

<br>18. 大多数编程语言都有用于捕获代码中发生的错误的方法，JavaScript中的try catch之类的，python也不例外，python中对应的代码为try except finally

```js
  try:
  	res = 10 / 0;
  	print res;
  except BaseException, e:
  	print e;
  finally:
  	print 'finally'
```
BaseException是所有异常的父类，python的错误捕获有类似冒泡的性质，比如main中调用了boo，boo调用了two，two函数中出现了错误，在main中依然能够捕捉到，另一个性质是，异常捕获有先后之分，如果在其他exception前调用了except BaseException，那么后面的补获语句将永远不会生效，这里给出python2.7的异常类间的继承关系，<a href="https://docs.python.org/2/library/exceptions.html#exception-hierarchy" target="_blank">python异常类继承大全</a>

<br>19. 在python中I/O可以这样写

```
  try:
  	f = open('path/to/file', 'r');
  	print f.read();
  except:
  	...
  finally:
    f.close();
```
感觉十分的麻烦，于是python提供了with语句来简化IO的写法，比如上述代码可以简化为

```
  with open('path/to/file', 'r') as f:
  	print f.read();
```
with语句自然会调用close()，这样就不用在finally中手动调用close()来关闭文件流

<br>20. python中对JSON的转换

在现在的web开发中，JSON应该是前后端通信的标准了233，python提供了非常完善的JSON处理方法，模块就叫做json

json.dumps()将python对象转为JSON对象

json.loads()将JSON对象转为python对象

当然，更多的时候，我们更愿意用class来表示一个对象，可以理解为MVC中的model层吧，这个时候，你要是直接用上面讲的方法，妥妥的给你报错，这时候，需要对dumps方法和loads方法配制一下

```js
  def student2dict(std):
  	return {
     'name': std.name,
     'age': std.age,
     'score': std.score
  	};
  	
  json.dumps(s, default=student2dict)
  
  def dict2student(d):
  	return Student(d['name'], d['age'], d['score']);
  	
  json.loads(jsonobject, object_hook=dict2student)
```

<br>21. 在python中有模块和包之分，模块的话就是一个简单的.py文件，包的话必须包含一个__init__.py文件，模块的引入方式为直接import，包的话，可以import全部的包，也可以from package import module

<br>22. python由于GIL锁的原因，多线程不能使用多核，十分鸡肋，因而python多用多进程模型进行并发操作，一般来说使用的都是multiprocessing 包中的Pool来创建一个python进程池

```
from multiprocess import Pool

p = Pool()

task = p.apply_async(f,  args=(参数,))

p.close()
p.join()

result = task.get()
```

一般在p.close()之后调用get方法获取并发操作的返回值，避免阻塞

<br>23. python List 向list末尾插入元素的方法是append方法，有一个简便的写法

```
a = [1, 2, 3]
a += 4,
a // [1, 2, 3, 4]
```

<br>24. python scrapy中不能定义名字为model的模块，会有冲突，定义成models即可

<br>25. python中str.find(substr)类似于js中的indexOf，然而，判断一个子字符串是否存在于父亲字符串中最快的方式为substr in str，list中的index()方法在字串不存在的时候会报错，不推荐使用

<br>26. python内置函数zip的用法，zip() 函数用于将可迭代的对象作为参数，将对象中对应的元素打包成一个个元组，然后返回由这些元组组成的列表。
如果各个迭代器的元素个数不一致，则返回列表长度与最短的对象相同，利用 * 号操作符，可以将元组解压为列表

```js
a, b = [1, 2, 3], [4, 5, 6]
c = zip(a, b) # [(1, 4), (2, 5), (3, 6)]
d = zip(*c) [(1, 2, 3), (4, 5, 6)]

l是一个递增的数组，快速请求最小的元素间隔
min_interval = min([abs(a - b) for a, b in zip(l, l[1:])])
```

<br>26. python中的最小数和最大数，类似于java中的Interger.MAX_VALUE或者js中的Number.MAX_SAFE_INTEGER，float('inf')(最大值)，float('-inf')(最小值)

<br>27. python中sorted函数配合lambda的用法:

```
a = [[1, 2], [3, 4], [5, 6]]
sorted(a, key = lambda x: x[1]) # 按照数组的第二个元素的大小排序
sorted(a, key = lambda x: x[1], reverse = True) # 按照数组的第二个元素的大小的反序大小排序
```

<br>28. python set 的操作

```
x, y = set(['a', 'p', 's', 'm']), set(['a', 'h', 'm'])

x & y = set('a', 'm') # 集合的交操作
x | y = set('a', 'p', 's', 'm', 'h') # 集合的并操作
x - y = set('p', 's') # 集合的差操作 
```

<br>29. python 3 < 4 and 4 < 5 可以简写为 3 < 4 < 5，简直不要太优美

<br>30. python函数是引用传值，修改传递的对象会影响形参

<br>31. python对象的深浅拷贝

```js
import copy

a = [1, 2, [3, 4]]
b = copy.copy(a) # 浅拷贝
c = copy.deepcopy(a) # 深拷贝
```

<br>32. python中一行写不下的东西可以用反斜杠\分成多行来表示

```
 # check conditions
    if (weather_is_hot == 1) and \
    (shark_warnings == 0):
    	send_goto_beach_mesg_to_pager()
```

<br>33. python合理的模块布局

```
#!/usr/bin/env python      起始行
"this is a test module"   模块文档
import sys, os            模块引入
debug = True              全局变量定义
class FooClass (object):  类定义
	pass
def test():               函数定义    
	foo = FooClass()
	if debug:
		print 'run test()'

if __name__ == '__main__': main方法
	test()
```

<br>34. python中判断两个变量是否是一个对象的方法

`obj1 is obj2` obj1和obj2是同一个对象

`obj1 is not obj2` obj1和obj2不是同一个对象

<br>35. 在python中，当一个对象生成的时候，python会给该对象绑上一个计算器，当该对象被引用的时候，计数器+1，当引用的对象销毁的时候，计数器-1，当计数器为0的时候，python垃圾回收器就回将该对象占用的内存释放，这就是python大概的内存管理机制

<br>36. 在python中检测变量类型有两种方法 type() 和 isinstance()，在python2.2之后对类型和类的统一导致 isinstance()使用的越来越多，由于int既是类型也是类，使用isinstance(variable, [type1, type2])显得更为方便，如果要使用type()方法的话，可以按照如下进行一些优化

```
import types
type(a) == types.IntType # 这是最容易想到的

type(a) is types.IntType # 其实没必要去比较数值是否相同，这里可以调用is去判断是否为相同的对象

from types import IntType # 这样写可以避免每次使用IntType的时候都去types模块内查找
```
<br>37. python中字符串和二进制串相互转换的方法

```
bin(int('256', 10)) # 0b100000000

str(int('0b100000000', 2)) # 256

''.join([bin(int(x)).replace('0b', '') for x in '999']) # 100110011001
```

<br>38. python中ASCII和字符串的转换

```
ord('a') # 97
chr(97) # 'a'
```

<br>39. python的随机数模块

```
from random import *
randrange(1, 5) # 返回1-5之内的随机一个整数
uniform(1, 5) # 返回1-5之内的随机一个浮点数
random() # 和js中的random一样，返回0-1之间的一个浮点数
choice() # 返回序列中的随机一个数值
```

<br>40. python中遍历字符串，依次减少最后一个字符的输出，简便写法

```
s = 'abcde'
s[:None] = 'abcde' # 厉害了
for i in [None] + range(-1, -len(s), -1):
	print s[:i]
	
依次输出abcde, abcd, abc, ab, a
```

<br>41. python中的string模块

```
import string
string.ascii_uppercase   # ABCDEDF...XYZ
string.ascii_lowercase   # abcdefg...xyz 
string.ascii_letters     # ABCD...XYZabcd...xyz
string.digits            # 0123456789
string.upper()           # 转为大写
string.lower()           # 转为小写
```

<br>42. python字符串中一些内建函数

```
string.count(substr, begin, end) # 计算字符串中下表为begin到end的范围内，substr出现的次数，如果没有指定的话，即为0 - len(string)
string.startswith() # 检查字符串是否以XXX开始
string.endswith() # 检查字符串是否以XXX结束
string.replace(a, b) # 将字符串中的a替换成b
string.upper() # 将字符串替换为大写形式
string.lower() # 将字符串替换为小写形式
```

<br>43. python不允许修改字符串中间的一个字符，如果需要修改的话，需要用切片创建一个新字符串

```
s = 'asd'
s[2] = 'S'
Trackback(innermost last):
File "<stdin>", line 1, in ? AttributeError: __setitem__

s = s[:2] + 'S' # asS
```

<br>44. python可以用一个很方便的内建方法fromkeys()来创建一个“默认”字典，字典中的元素具有相同的值（如果没有给出，默认为None）

```
dict = {}.fromkeys(('x', 'y'), -1)
dict # {'x': -1, 'y': -1}
```

<br>45. python中字典的大小比较比较鸡肋，因为python的字典为了查找的速度，key仅仅和hash值有关，但是还是记录一下python字典的比较方法

python字典比较首先进行字典长度的比较，然后是key值的比较，最后是value值的比较，比较规则和序列的一样，不做过多记录

<br>46. python字典的内建函数

```
a = {'a': 1, 'b': 2}
a.get(key, default=None) # 获取字典中key的value，如果key不存在，则返回default值，如果没有给出default，则返回None
a.setdefault(key, default) # 获取字典中key的value，如果key不存在，则设置key的value为default并且放回该值，如果key存在的话，返回value值
```

<br>47. python的不可变集合frosenset(存在hash，可以作为字典的key值)和可变集合set，set中的add类似list的append，set中的update类似list的extend

```
a = [1, 2]
a.extend('asd') # [1, 2, 'a', 's', 'd']
a.append('asd') # [1, 2, 'a', 's', 'd', 'asd']

a = set([1, 2])
a.add('asd') # set([1, 2, 'asd'])
a.update('asd') # set([1, 2, 'asd', 'a', 's', 'd'])
```

<br>48. python中else语句的其他用法

在C语言中，你不会在条件语句范围外发现else语句，但python不同，你可以在while和for循环中使用else语句，在循环中使用时，else子句只在循环完成后执行，也就是说break语句也会跳过else块

<br>49. python在使用for...in...迭代器遍历字典的时候，不能使用del语句删除字典中的项，因为一个序列的迭代器只是记录你当前到达第多少个元素，所以如果你在迭代的时候改变了元素，更新会立即反映到你所迭代的条目上，但是使用字典的keys()方法是可以的，因为keys()返回一个独立于字典的列表。而迭代器是与实际对象绑定在一起的，它将不会继续执行下去。

<br>50. python使用生成器的一个例子

```
rows = [1, 2, 3, 17]

def cols():
	yield 56
	yield 2
	yield 1
	
x_product_pairs = ((i, j) for i in rows for j in cols())

for pair in x_product_pairs:
	print pair
	
(1, 56)...
```

<br>51. python除了可以进行显示的参数调用，还可以传入列表和字典进行传参，传入字典的时候，可以不按照顺序，举个栗子:

```
def print_stu(name, age):
	print name, age

print_stu(*['bob', 21]) # bob, 21
print_stu(**{'name': 'bob', 'age': 21}) # bob, 21
```

<br>52. python也可以在函数声明中定义接受可变数量的参数（我觉得js es6的三点运算符...就是和python学的2333）

```
def print_stu(name, *tuple, **obj):
	pass

print_stu('bob', 21, 32, arg1 = 'a', arg2 = 'b')
tuple = (21, 32)
obj = { 'arg1': 'a', 'arg2': 'b' }

```

<br>53. python中局部作用域如果想影响全局作用域的变量，可以使用global关键字

```
is_this_global = 'xyz'
def foo():
	global is_this_global
	is_this_global = 'def'
	
>>> foo()
>>> print is_this_global
def
```

<br>54. python中，推荐使用如下风格的import语句模块导入顺序

1. Python 标准库模块
2. Python 第三方模块
3. 应用程序自定义模块

然后用一个空行分割这三类模块的导入语句，这将确保模块使用固定的习惯导入，有助于减少每个模块需要的import语句数目

<br>55. python中，class中的`__init__()`方法和`__del__()`方法类似于构造函数和解构函数（可能有同学会说，`__new__()`才是构造函数，`__new__()`函数返回一个self实例，提供给`__init__()`初始化），需要注意的是，如果类存在一个继承的非object的基类，在`__del__()`中需要首先调用`Parent.__del__()`，下面给出一个最简单的`__init__()`和`__del__()`的用法，记录该class实例化了多少个实例

```js
class InstCc(object):
	count = 0
	def __init__(self):
		InstCc.count += 1
	def __del__(self):
		InstCc.count -= 1
	def howMany(self):
		return InstCc.count

a = InstCc()
b = InstCc()
b.howMany() 2
a.howMany() 2
del b
a.howMany() 1
del a
InstCc.count 0
```

<br>56. python中类属性如果是不可变的，那么通过实例修改类属性不会对类属性造成影响，相当于在实例的__dict__属性中创建了一个新的字段，然而，如果通过实例修改的类属性是可变的话，比如字典，那么，修改实例属性会同步修改类属性，所以尽量不要通过实例修改类属性，操作类的静态数据的时候，使用Class.property

<br>57. python中class存在静态方法（staticmethod）和类方法（classmethod)，类方法定义的时候需要传入一个参数，一般是cls，可以理解为创建实例中的self，这个cls就代指类对象，可以通过cls获得类的属性，比如cls.__name__，下面给出一个栗子

```
  1 #!/usr/bin/env python
  2
  3 class Stu:
  4     count = 1
  5
  6     @staticmethod
  7     def how_many():
  8         print Stu.count
  9
 10     @classmethod
 11     def how_much(cls):
 12         print cls.count
 13
 14 if __name__ == '__main__':
 15     stu = Stu()
 16     stu.how_many()
 17     stu.how_much()
```

<br>58. python类的继承，子类可以调用父亲类中的所有方法，但是如果在子类中需要覆盖掉父亲类的方法，例如__init__方法，可以在子类中显示的调用父亲类的方法

```
class P(object):	def __init__(self):		print "calling P's constructor"
		
class C(P):
	def __init__(self):
		P.__init__(self) / super(P, self).__init__() # 需要注意的是，super方法只有基类为新类才能有，如果基类为老类 class P: 这种类型的，只能用第一种方法显示的调用父亲类的重名方法
```

<br>59. 类、实例的一些内建函数

issubclass(child, parent) python2.3之后parent可以为一个父亲类组成的元组

isinstance(example, class) 和issubclass一样，class可以为一个元组

*attr()系列函数可以在各种对象下工作，不限于类(class)和实例(instance)，但是在类和实例里面，使用的非常多，于是在这里介绍一下

hasattr(obj, attr)

setattr(obj, attr)

getattr(obj, attr[, 默认值])

delattr(obj, attr)

vars(obj = None) 返回obj的属性及其的一个字典；如果没有给出obj，vars()显示局部名字空间字典(属性及其值)，也就是locals()

在类中间方法中返回一个类，一般不建议直接使用类名，这样修改的时候需要逐个查找修改，推荐使用`self.__class__`方法

<br>60. python except 捕获异常既可以分开操作，也可以作为一个元组合在一起捕获

```
try:
	A
except MyException: B
else: C(try块中的代码完全正确执行完毕，没有发生异常，调用else中的代码)
finally: D(无论如何，都会调用的代码)

try:
	try-body
except TypeError, e:
	except-body
except ValueError, e:
	except-body

try:
	try-body
except (TypeError, ValueError), e:
	except-body
```

<br>61. with语句的使用

python提供with语句进一步的透明程序中发生的细节，让程序员更加注重于代码的实现，用打开文件作为例子介绍一下with

```
with open('path', 'r') as f:
	for eachlines in f:
		# ...do stuff with eachLine or f...
```

<br>62. python中，在windows环境下，换行符为'\r\n'，在mac环境下，换行符为'\n'，这在跨平台读取文件的时候，就会存在一些问题，解决方法如下所示:

* 如果不是txt文件，建议用wb和rb来读写。通过二进制读写，不会有换行问题。
* 如果需要明文内容，请用rU来读取（强烈推荐），即U通用换行模式（Universal new line mode）。该模式会把所有的换行符（\r \n \r\n）替换为\n。
* os.linesep，可以获得当前操作系统的换行符
* os.sep 用来分隔文件路径名的字符串
* os.pathsep 用来分隔文件路径的字符串
* os.curdir 当前工作目录的字符串名称
* os.pardir 当前工作目录的父目录字符串的名称

<br>63. python中，通过open或者file操作文件，当使用输入方法如read()或者readlines()从文件中读取行的时候，python并不会删除行结束符，同理，输出方法，write()或者writelines()也不会自动加入行结束符，这都是留给程序员自己解决的	

```js
f = open('myFile', 'r')
data = [line.strip() for line in f.readlines()]
f.close()
```

<br>64. python中通过sys.argv属性提供了对命令行参数的访问，sys.argv是一个列表，sys.argc是一个number，也就是len(sys.argv)

<br>65. python中关于正则表达式的操作

```
import re

pattern = re.compile(r'[abc]', re.S(让.可以匹配\n)) 预编译，增加速度

re.match(pattern, string) 从头开始匹配，找到一个符合正则表达式的匹配串
re.search(pattern, string) 随便从什么地方匹配，找到一个符合正则表达式的匹配串，一般来说用search不用match
re.group() / re.group(0) 返回search或match
匹配到的串
re.group(1) 返回第一个字匹配
re.groups() 返回子匹配组成的元组
re.findall(pattern, string) 返回字符串中所有的匹配，不限于找到第一个
re.sub(pattern, newstring, string) 类似于js中的replace
re.spilt(pattern, string) 分割字符串，返回list，速度比string的split快得多
```

<br>66. python自定义类的一些方法[自定义python类](https://www.liaoxuefeng.com/wiki/001374738125095c955c1e6d8bb493182103fac9270762a000/0013946328809098c1be08a2c7e4319bd60269f62be04fa000)

\_\_iter\_\_: 如果一个类想被用于for ... in循环，类似list或tuple那样，就必须实现一个\_\_iter\_\_()方法，该方法返回一个迭代对象，然后，Python的for循环就会不断调用该迭代对象的next()方法拿到循环的下一个值，直到遇到StopIteration错误时退出循环 

\_\_getitem\_\_: 要表现得像list那样按照下标取出元素

\_\_getattr\_\_: 当调用不存在的属性时，比如score，Python解释器会试图调用\_\_getattr\_\_(self, 'score')来尝试获得属性，这样，我们就有机会返回score的值

\_\_call\_\_: 直接调用实例() instance = Class() instance() => 访问 \_\_call\_\_()

<br>67. 常用内建模块 -- collections

1. namedtuple: namedtuple是一个函数，它用来创建一个自定义的tuple对象，并且规定了tuple元素的个数，并可以用属性而不是索引来引用tuple的某个元素

	```
	from collections import namedtuple
	
	Point = namedtuple('Point', ['x', 'y'])
	p = Point(1, 2)
	p.x // 1
	p.y // 2
	```
	
2. deque: 使用list存储数据时，按索引访问元素很快，但是插入和删除元素就很慢了，因为list是线性存储，数据量大的时候，插入和删除效率很低。deque是为了高效实现插入和删除操作的双向列表，适合用于队列和栈

	```
	>>> from collections import deque
	>>> q = deque(['a', 'b', 'c'])
	>>> q.append('x')
	>>> q.appendleft('y')
	>>> q
	deque(['y', 'a', 'b', 'c', 'x'])
	```

3. OrderedDict: 字典的key值排序是按照hash来的，如果想按照插入的顺序来排序，可以采用OrderedDict

	```
	>>> from collections import OrderedDict
	>>> d = dict([('a', 1), ('b', 2), ('c', 3)])
	>>> d # dict的Key是无序的
	{'a': 1, 'c': 3, 'b': 2}
	>>> od = OrderedDict([('a', 1), ('b', 2), ('c', 3)])
	>>> od # OrderedDict的Key是有序的
	OrderedDict([('a', 1), ('b', 2), ('c', 3)])
	```
4. Counter: Counter是一个简单的计数器，例如，统计字符出现的个数

	```
	>>> from collections import Counter
	>>> Counter('programmer')
	Counter({'r': 3, 'm': 2, 'a': 1, 'e': 1, 'g': 1, 'o': 1, 'p': 1})
	```

5. defaultdict: 给字典设置初始值，可以减少代码的书写量

	```
	>>> from collections import defaultdict
	>>> a = defaultdict(int)
	>>> a['a'] += 1
	>>> a['a'] # 1
	>>> a = defaultdict(str)
	>>> a['a'] += 'a'
	>>> a['a'] # 'a'
	```

<br>68. 常用内建模块 -- base64

* base64.b64encode()
* base64.b64decode()

由于标准的Base64编码后可能出现字符+和/，在URL中就不能直接作为参数，所以又有一种"url safe"的base64编码，其实就是把字符+和/分别变成-和_

* base64.urlsafe_b64encode()
* base64.urlsafe_b64decode()

<br>69. 常用内建模块 -- hashlib

其实就是 MD5, SHA1 之类的加密算法

```
import hashlib

md5 = hashlib.md5()
md5.update('how to use md5 in ')
md5.update('python hashlib?')

sha1 = hashlib.sha1()
sha1.update('how to use sha1 in ')
sha1.update('python hashlib?')
```

<br>70. python中快速建立集合的方法，`{()}`这样就建立了一个集合

<br>71. python sqlalchemy 的常用API

[sqlalchemy的API](http://docs.sqlalchemy.org/en/latest/orm/query.html)

* query(Table) 查询
* one() 返回一个确定的item，如果返回多个会报错，没有返回也会报错
* first() 返回当前查询排行第一的结果，如果没有实体则返回None
* order_by(Table.Field) 按照一个字段的数值从小到大的返回list
* order_by(Table.Field.desc()) 反序
* filter(Table.Field == num) 过滤函数，相当于sql中的where
* from sqlalchemy.sql import func, func中有max，min，count等函数，可以用于查询，query(func.max(Table.Field))
* scalar() 返回第一个元素的第一个结果（这就是和first的区别），如果没有行存在，返回None。如果返回多行，则引发MultipleResultsFound，session.query(Item.id, Item.name).scalar() == 1，只返回id，经常和func共同使用
