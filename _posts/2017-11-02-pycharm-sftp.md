---
layout:     post
title:      "pycharm-sftp"
subtitle:   "pycharm本地文件同步远程开发机"
date:       2017-11-02 23:00:00
author:     "Mickey"
header-img: "img/post-bg-alitrip.jpg"
tags:
    - python
---

在使用python进行后端接口的开发的时候，将本地代码同步到远程开发机是很常见的需求，pycharm为我们提供了sftp同步文件的方法，下面来讲一个如何配置

* 打开pycharm，选择上方tools -> deployment -> configurations

![img](/img/in-post/pycharm-sftp/1.jpg)

* 输入服务器的地址，登陆username, password等

![img](/img/in-post/pycharm-sftp/2.jpg)

* 选择上方tools -> deployment -> 勾选automatic upload

* <strong>最重要的一点</strong>，本地的目录结构需要完全和服务器上的一样，可以先在一个地方配置好目录结构，然后使用scp进行拷贝，确保目录结构一样
