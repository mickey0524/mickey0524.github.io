---
layout:     post
title:      "python-time"
subtitle:   "python之time基础"
date:       2017-11-17 11:30:00
author:     "Mickey"
header-img: "img/post-bg-2015.jpg"
tags:
    - python
---

本文旨在记录python操作时间日期的方法，已备温故知新

```
      _       _       _   _
     | |     | |     | | (_)
   __| | __ _| |_ ___| |_ _ _ __ ___   ___
  / _` |/ _` | __/ _ \ __| | '_ ` _ \ / _ \
 | (_| | (_| | ||  __/ |_| | | | | | |  __/
  \__,_|\__,_|\__\___|\__|_|_| |_| |_|\___|
```

以`datetime`为中心，起点或中转，转化为目标对象，涵盖了大多数业务场景中需要的日期转换处理

步骤：

```
1. 掌握几种对象及其关系(date，datetime，string，timetuple，timestamp)
2. 了解每类对象的基本操作方法
3. 通过转化关系转化
```

## 涉及对象

### 1. datetime

```
>>> import datetime
>>> now = datetime.datetime.now()
>>> now
datetime.datetime(2015, 1, 12, 23, 9, 12, 946118)
```

### 2. timestamp

```
>>> import time
>>> time.time()
1510889000.69805
```

### 3. time tuple

```
>>> import time
>>> time.localtime()
time.struct_time(tm_year=2017, tm_mon=11, tm_mday=17, tm_hour=11, tm_min=24, tm_sec=5, tm_wday=4, tm_yday=321, tm_isdst=-1)
```

### 4. string

```
>>> from datetime import datetime
>>> datetime.now().strftime('%Y-%m-%d %H:%M:%S')
'2017-11-17 11:27:04'
```
### 5. date

```
>>> from datetime import datetime
>>> datetime.now().date()
datetime.date(2017, 11, 17)
```

## datetime基本操作

### 1. 获取当前datetime

```
>>> from datetime import datetime
>>> datetime.now()
datetime.datetime(2017, 11, 17, 11, 29, 44, 748590)
```

### 2. 获取当天date

```
>>> from datetime import datetime
>>> datetime.now().date()
datetime.date(2017, 11, 17)
```

### 3. 获取明天/前N天

```
>>> from datetime import datetime, timedelta
>>> datetime.now().date() + timedelta(days=1)
datetime.date(2017, 11, 18)
>>> datetime.now() + timedelta(days=1)
datetime.datetime(2017, 11, 18, 13, 39, 17, 495497)
```

### 4. 获取当天开始和结束时间(00:00:00 23:59:59)

```
>>> from datetime import datetime, time
>>> datetime.combine(datetime.now().date(), time.min)
datetime.datetime(2017, 11, 17, 0, 0)
>> datetime.combine(datetime.now().date(), time.max)
datetime.datetime(2017, 11, 17, 23, 59, 59, 999999)
```

### 5. 获取两个datetime的时间差

```
>>> from datetime import datetime
>>> (datetime(2017, 11, 17, 15, 0, 0) - datetime.now()).total_seconds()
4561.230405
```

## 关系转换

### 1. datetime <=> string

datetime -> string

```
>>> from datetime import datetime
>>> datetime.now().strftime('%Y-%m-%d %H:%M:%S')
'2017-11-17 13:46:28'
```

string -> datetime

```
>>> from datetime import datetime
>>> datetime.strptime('2017-11-17 13:46:28', '%Y-%m-%d %H:%M:%S')
datetime.datetime(2017, 11, 17, 13, 46, 28)
```

### 2. datetime <=> timetuple

datetime -> timetuple

```
>>> from datetime import datetime
>>> datetime.now().timetuple()
time.struct_time(tm_year=2017, tm_mon=11, tm_mday=17, tm_hour=13, tm_min=52, tm_sec=58, tm_wday=4, tm_yday=321, tm_isdst=-1)
```

timetuple -> datetime

```
>>> from datetime import datetime
>>> import time
>>> datetime.fromtimestamp(time.mktime(timetuple))
```

### 3. datetime <=> date

datetime -> date

```
>>> from datetime import datetime
>>> datetime.now().date()
datetime.date(2017, 11, 17)
```

date -> datetime

```
>>> from datetime import datetime, time
>>> today = datetime.now().date()
>>> datetime.combine(today, time())
>>> datetime.combine(today, time.max/min)
```

### 4. datetime <=> timestamp

datetime -> timestamp

```
>>> from datetime import datetime
>>> import time
>>> time.mktime(datetime.now().timetuple())
```

timestamp -> datetime

```
>>> from datetime import datetime
>>> datetime.fromtimestamp(1510889000.69805)
datetime.datetime(2017, 11, 17, 11, 23, 20, 698050)
```
