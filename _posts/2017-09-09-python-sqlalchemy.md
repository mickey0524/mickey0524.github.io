---
layout:     post
title:      "python-sqlalchemy-install"
subtitle:   "python sqlalchemy 安装"
date:       2017-09-09 23:00:00
author:     "Mickey"
header-img: "img/post-bg-alitrip.jpg"
tags:
    - python
---

最近在学习python sqlalchemy进行ORM操作的时候，遇到了一些安装方面的问题，在此记录一下～

1.使用pip安装sqlalchemy，安装完毕之后，运行写好的demo，发现如下错误

```
Traceback (most recent call last):
  File "orm-python.py", line 20, in <module>
    engine = create_engine('mysql+mysqlconnector://root:baihao0524@localhost:3306/baihao')
  File "/Users/baihao/Library/Python/2.7/lib/python/site-packages/sqlalchemy/engine/__init__.py", line 391, in create_engine
    return strategy.create(*args, **kwargs)
  File "/Users/baihao/Library/Python/2.7/lib/python/site-packages/sqlalchemy/engine/strategies.py", line 80, in create
    dbapi = dialect_cls.dbapi(**dbapi_args)
  File "/Users/baihao/Library/Python/2.7/lib/python/site-packages/sqlalchemy/dialects/mysql/mysqlconnector.py", line 107, in dbapi
    from mysql import connector
ImportError: No module named mysql
```

这是因为没有安装python连接数据库的包，python用于连接数据库的包有两个，分别为

mysql-connector-python：是MySQL官方的纯Python驱动；

MySQL-python：是封装了MySQL C驱动的Python驱动。

2.这里我们下载mysql-connector-python作为python连接mysql的驱动，然后发现用pip死活下载不下来这个包，无论是加上sudo，--user，甚至去git源下载再编译安装，统统不行，最后没办法，选择去[mysql-connector-python的下载地址](https://dev.mysql.com/downloads/connector/python/)下载最新版本的.dmg安装包进行安装

3.验证是否下载成功

```
~ python
>>> import mysql.connector as ms
>>> ms.__version__
'2.1.7'
```