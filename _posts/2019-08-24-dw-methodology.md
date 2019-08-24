---
layout:     post
title:      "dw-methodology"
subtitle:   "数仓方法论"
date:       2019-08-24 23:00:00
author:     "Mickey"
header-img: "img/post-bg-nextgen-web-pwa.jpg"
tags:
    - 大数据
---

最近面试的时候，面试官建议我去看看数据仓库的方法论，我思考了一下，觉得自己以往搭建数仓确实是凭借自己对数据的感觉的，缺乏理论的支持，于是决定看一下数据仓库工具箱这本书，这篇文章用来记录一下感悟

* 维度模型设计的四个步骤

    1. 选择业务过程：我认为这就是理解自己的业务是在做什么
    2. 声明粒度：精确定义某个事实表的每一行在表示什么
    3. 确定纬度：确定度量环境中所有可能出现的单值描述符
    4. 确定事实：确定过程的度量是什么

    ![dw_1](/img/in-post/dw-methodology/dw_1.png)