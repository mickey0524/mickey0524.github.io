---
layout:     post
title:      "redis-learn"
subtitle:   "redis的API简介"
date:       2017-11-18 23:30:00
author:     "Mickey"
header-img: "img/post-bg-js-module.jpg"
tags:
    - 数据库
---

最近在系统的过一遍redis的API，虽然在实际编程中这些API很少被用到，但是和mysql一样，基本的命令行操作还是要会的～于是总结了这篇博客，用以温故知新

* Redis keys命令

  * DEL key 删除这个键值对
  * EXISTS key 是否存在这个key值
  * EXPIRE key seconds 给key设置过期时间
  * KEYS pattern(正则) 给出所有符合pattern条件的key列表，KEYS * 代表给出所有的key
  * PERSIST key 删除key的过期时间
  * TTL key 给出key还有多少s就过期
  * TYPE key 返回 key 所储存的值的类型
  * RENAME key newkey 修改 key 的名称
  * RENAMENX key newkey 仅当 newkey 不存在时，将 key 改名为 newkey

* Redis 字符串(String)

  * SET KEY value 设置指定的KEY值为value
  * GET KEY 获取KEY的数值
  * MGET KEY1 [KEY2...] 一次性获取KEY1，KEY2...等的值
  * MSET key value [key value ...] 同时设置一个或多个 key-value 对
  * SETEX key seconds value 将值 value 关联到 key ，并将 key 的过期时间设为 seconds (以秒为单位)

* Redis 哈希(Hash)

  * HSET KEY field value 设置哈希表中key值为value
  * HGET KEY field 获取key值的哈希表的field字段的数值
  * HMSET key field1 value1 [field2 value2 ] 同时将多个 field-value (域-值)对设置到哈希表 key 中
  * HMGET key field1 [field2] 获取所有给定字段的值
  * HGETALL key 获取key中所有的field和value值
  * HDEL key field [field2] 删除哈希表中的字段
  * HEXISTS key field 查询哈希表中field字段是否存在

* Redis 列表(List)

  * LLEN key 获取列表的长度
  * LRANGE key start stop 获取列表指定索引的元素
  * LINDEX key index 通过索引获取列表中的元素
  * LINSERT key BEFORE|AFTER pivot value 在列表的元素前或者后插入元素
  * LPUSH key value1 [value2] 将一个或多个值插入到列表头部
  * RPUSH key value1 [value2] 在列表中添加一个或多个值
  * LPOP key 移出并获取列表的第一个元素
  * RPOP key 移除并获取列表最后一个元素
  * LREM key count value 移除列表元素
    * count > 0 : 从表头开始向表尾搜索，移除与 VALUE 相等的元素，数量为 COUNT
    * count < 0 : 从表尾开始向表头搜索，移除与 VALUE 相等的元素，数量为 COUNT 的绝对值。
    * count = 0 : 移除表中所有与 VALUE 相等的值。

* Redis 集合(Set)

  * SADD key member1 [member2] 向集合添加一个或多个成员
  * SCARD key 获取集合的成员数
  * SDIFF key1 [key2] 返回给定所有集合的差集
  * SINTER key1 [key2] 返回给定所有集合的交集
  * SISMEMBER key member 判断 member 元素是否是集合 key 的成员
  * SMEMBERS key 返回集合中的所有成员
  * SREM key member1 [member2] 移除集合中一个或多个成员
  * SUNION key1 [key2] 返回所有给定集合的并集

* Redis 有序集合(Sorted Set)

  * ZADD key score1 member1 [score2 member2] 向有序集合添加一个或多个成员，或者更新已存在成员的分数
  * ZCARD key 获取有序集合的成员数
  * ZCOUNT key min max 计算在有序集合中指定区间分数的成员数
  * ZLEXCOUNT key min max 计算有序集合中指定字典区间内成员数量 min = -/[b max = +/[f (-代表最小，+代表最大,[a代表从a开始且包括a，(a代表从a开始但是不包括a)
  * ZRANGE key start stop [WITHSCORES] 通过`索引`区间返回有序集合成指定区间内的成员(按照score从小到大排列，withscores代表将score一起打印出来)
  * ZRANGEBYLEX key start stop (start和stop的规则和ZLEXCOUNT一样)
  * ZRANGEBYSCORE key start stop [WITHSCORES] start/stop(-inf代表最小，+inf代表最大，(a代表大于a，a代表可以等于a)
  * ZRANK key member 返回集合中指定元素的索引
  * ZREM key member [member ...] 移除有序集合中的一个或多个成员
  * ZREMRANGEBYLEX key min max 移除有序集合中给定的字典区间的所有成员
  * ZREMRANGEBYRANK key start stop 移除有序集合中给定的排名区间的所有成员
  * ZREMRANGEBYSCORE key min max 移除有序集合中给定的分数区间的所有成员
  * ZREVRANGE key start stop [WITHSCORES] 返回有序集合中指定成员的排名，有序集成员按分数值递减(从大到小)排序，默认的话有序集合会按照从大到小来进行排列
  * ZSCORE key member 返回有序集中，成员的分数值

* Redis 事务

  Redis事务可以一次执行多个命令，并且带有以下两个重要的保证：

  * 事务是一个单独的隔离操作：事务中的所有命令都会序列化、按顺序地执行。事务在执行的过程中，不会被其他客户端发送来的命令请求所打断。
  * 事务是一个原子操作：事务中的命令要么全部被执行，要么全部都不执行。

  一个事务从开始到执行会经历以下三个阶段：

  * 开始事务
  * 命令入队
  * 执行事务

  一个事务以MULTI开始，然后将多个命令入队到事务中，最后由EXEC命令触发事务，一并执行事务中的所有命令

  ```
  redis 127.0.0.1:6379> MULTI
  OK
  
  redis 127.0.0.1:6379> SET book-name "Mastering C++ in 21 days"
  QUEUED
  
  redis 127.0.0.1:6379> GET book-name
  QUEUED
  
  redis 127.0.0.1:6379> SADD tag "C++" "Programming" "Mastering Series"
  QUEUED
  
  redis 127.0.0.1:6379> SMEMBERS tag
  QUEUED
  
  redis 127.0.0.1:6379> EXEC
  1) OK
  2) "Mastering C++ in 21 days"
  3) (integer) 3
  4) 1) "Mastering Series"
     2) "C++"
     3) "Programming"
  ```
