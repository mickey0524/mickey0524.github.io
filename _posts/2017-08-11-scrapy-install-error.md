---
layout:     post
title:      "scrapy-install-error"
subtitle:   "scrapy安装失败"
date:       2017-08-11 14:00:00
author:     "Mickey"
header-img: "img/post-bg-os-metro.jpg"
tags:
    - python
---

笔者最近沉迷python爬虫，经过一系列基础练习之后，终于到了使用框架的阶段，笔者选择了scrapy框架，这篇文章只是记录一下安装所踩的坑，所以不介绍scrapy的优点了，有需要者自行谷歌，知乎～

1.博主用的是mac，window没试过哈，首先安装pip

2.然后通过pip安装scrapy

`sudo pip install scrapy`

3.一般情况下，都能正常安装，然后在命令行输入scrapy

```
TLSVersion.TLSv1_1: SSL.OP_NO_TLSv1_1,
AttributeError: 'module' object has no attribute 'OP_NO_TLSv1_1'
```

解决方法是如下：

3.1 确认安装twisted

`pip freeze|grep Twisted`

3.2 更新pyOpenSSL

`pip install --upgrade pyOpenSSl`

安装过程中提示 PemissionDenied, 记得加上sudo即可

`sudo pip install --upgrade pyOpenSSl`

如果加上sudo依旧提示 Operation not permitted, 那就再加上--user选项

`sudo pip install --upgrade --user pyOpenSSl`

4.如何提示Pip安装依赖于six的库失败，参考下面这篇博客

[[pip]Pip安装依赖于six的库失败的解决方法](http://www.jianshu.com/p/45fb07007ddc)