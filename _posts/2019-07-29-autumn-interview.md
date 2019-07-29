---
layout:     post
title:      "autumn interview"
subtitle:   "秋招面试总结"
date:       2019-07-29 22:00:00
author:     "Mickey"
header-img: "img/post-bg-universe.jpg"
tags:
    - 面试
---

秋招也要陆陆续续的开始了，今天我面试了第一家公司，所以这篇博客来记录一下

* 快手 —— 数据研发工程师

    因为之前暑期三面全过，因此这次提前批，算是直通终面吧，一轮长达一个半小时的技术面之后，进行了 hr 面试

    * 一个 uid，p_date 的 hive 表，计算 7 留

        ```
        select
            t1.p_date as p_date,
            count(distinct t2.uid) / count(distinct t1.uid) as 7_retain
        from
            t t1 left join t t2 on
            t1.uid = t2.uid and
            datediff(t2.p_date, t1.p_date) >= 1 and
            datediff(t2.p_date, t1.p_date) <= 7
        group by
            t1.p_date;
        ```

    * 一个 referer 的 hive 表，计算 top 10 的 referer

        ```
        select
            referer,
            count(*) as num
        from
            t
        group by
            referer
        order by
            num desc
        limit 10;
        ```

    * 一个 referer，url 的 hive 表，计算 top 10 的跳出 url（需要用 url join referer）

        ```
        select
            t1.url,
            count(*) as num
        from
            t t1 join t t2 on
            t1.url = t2.referer
        group by
            t1.url
        order by
            num desc
        limit 10;
        ```

    * flink 源码相关

        感兴趣的同学可以参考 [flink 流式处理源码](https://github.com/mickey0524/flink-streaming-source-analysis)

    * redis 如何持久化

        AOF + RDB