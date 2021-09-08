<p align="center">
    <img width="280px" src="image/dongwu/dog/12.png" >
</p>

# Hive（十二）Hive分析窗口函数(二)NTILE,ROW_NUMBER,RANK,DENSE_RANK

## 概述

本文中介绍前几个序列函数，NTILE,ROW_NUMBER,RANK,DENSE_RANK，下面会一一解释各自的用途。

**注意： 序列函数不支持WINDOW子句。（\**ROWS BETWEEN\**）**

## 数据准备

### 数据格式

```
cookie1,2015-04-10,1
cookie1,2015-04-11,5
cookie1,2015-04-12,7
cookie1,2015-04-13,3
cookie1,2015-04-14,2
cookie1,2015-04-15,4
cookie1,2015-04-16,4
cookie2,2015-04-10,2
cookie2,2015-04-11,3
cookie2,2015-04-12,5
cookie2,2015-04-13,6
cookie2,2015-04-14,3
cookie2,2015-04-15,9
cookie2,2015-04-16,7
```

### 创建表

```
use cookie;
drop table if exists cookie2;
create table cookie2(cookieid string, createtime string, pv int) row format delimited fields terminated by ',';
load data local inpath "/home/hadoop/cookie2.txt" into table cookie2;
select * from cookie2;
```

![img](https://images2018.cnblogs.com/blog/1228818/201804/1228818-20180411190330410-382259848.png)

## NTILE

### 说明

NTILE(n)，用于将分组数据按照顺序切分成n片，返回当前切片值
NTILE不支持ROWS BETWEEN，比如 NTILE(2) OVER(PARTITION BY cookieid ORDER BY createtime ROWS BETWEEN 3 PRECEDING AND CURRENT ROW)
如果切片不均匀，默认增加第一个切片的分布

### 查询语句

```
select
  cookieid,
  createtime,
  pv,
  ntile(2) over (partition by cookieid order by createtime) as rn1, --分组内将数据分成2片
  ntile(3) over (partition by cookieid order by createtime) as rn2, --分组内将数据分成2片
  ntile(4) over (order by createtime) as rn3 --将所有数据分成4片
from cookie.cookie2 
order by cookieid,createtime;
```

### 查询结果

![img](https://images2018.cnblogs.com/blog/1228818/201804/1228818-20180411191813159-31132130.png)

### 比如，统计一个cookie，pv数最多的前1/3的天

#### 查询语句

```
select
  cookieid,
  createtime,
  pv,
  ntile(3) over (partition by cookieid order by pv desc ) as rn 
from cookie.cookie2;
```

#### 查询结果

![img](https://images2018.cnblogs.com/blog/1228818/201804/1228818-20180411192539356-1545943983.png)

--rn = 1 的记录，就是我们想要的结果

## ROW_NUMBER

### 说明

ROW_NUMBER() –从1开始，按照顺序，生成分组内记录的序列
–比如，按照pv降序排列，生成分组内每天的pv名次
ROW_NUMBER() 的应用场景非常多，再比如，获取分组内排序第一的记录;获取一个session中的第一条refer等。

### 分组排序

```
select
  cookieid,
  createtime,
  pv,
  row_number() over (partition by cookieid order by pv desc) as rn
from cookie.cookie2;
```

### 查询结果

![img](https://images2018.cnblogs.com/blog/1228818/201804/1228818-20180411193336998-1717025751.png)

-- 所以如果需要取每一组的前3名，只需要rn<=3即可，适合TopN

## RANK 和 DENSE_RANK

—RANK() 生成数据项在分组中的排名，排名相等会在名次中留下空位
—DENSE_RANK() 生成数据项在分组中的排名，排名相等会在名次中不会留下空位

### 查询语句

```
select
  cookieid,
  createtime,
  pv,
  rank() over (partition by cookieid order by pv desc) as rn1,
  dense_rank() over (partition by cookieid order by pv desc) as rn2,
  row_number() over (partition by cookieid order by pv desc) as rn3
from cookie.cookie2 
where cookieid='cookie1';
```

### 查询结果

 ![img](https://images2018.cnblogs.com/blog/1228818/201804/1228818-20180411193913711-1567210105.png)

## ROW_NUMBER、RANK和DENSE_RANK的区别

**row_number**： 按顺序编号，不留空位
**rank**： 按顺序编号，相同的值编相同号，留空位
**dense_rank**： 按顺序编号，相同的值编相同的号，不留空位