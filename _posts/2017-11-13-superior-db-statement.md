---
layout:     post
title:      "superior-db-statement"
subtitle:   "天秀的db写法"
date:       2017-11-13 23:30:00
author:     "Mickey"
header-img: "img/post-bg-kuaidi.jpg"
tags:
    - 数据库
---

* 表中sex字段的数值为'm'或者'f'，将表格中的'm'替换为'f'，将'f'替换为'm'，👇两种写法都可以

	```
	UPDATE salary
	SET sex = IF(sex = 'm', 'f', 'm');
	
	UPDATE salary
	SET sex = (
		CASE WHEN
		sex = 'm'
		THEN 'f'
		ELSE 'm'
		END
	)
	```

* 查找Person表中所有重复的email，👇有三种写法

  ```
  select distinct Email
  from Person as p1
  where Email in 
              (select Email
               from Person
               where ID != p1.Id);
               
               
  
  select distinct P1.Email
  from Person as P1
  Inner join Person as P2
  on P1.Email = P2.Email
  where P1.ID != P2.ID;
  
  
  select Email
  from Person
  group by Email
  having Count(*) > 1         
  ```