<p align="center">
    <img width="280px" src="image/dongwu/dog/11.png" >
</p>

# Hive（十一）Hive窗口分析函数（一）SUM、AVG、MIN、MAX

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
```

### 创建数据库及表

```
create database if not exists cookie;
use cookie;
drop table if exists cookie1;
create table cookie1(cookieid string, createtime string, pv int) row format delimited fields terminated by ',';
load data local inpath "/home/hadoop/cookie1.txt" into table cookie1;
select * from cookie1;
```

![img](https://images2018.cnblogs.com/blog/1228818/201804/1228818-20180410205530168-1924440479.png)

## SUM

### 查询语句

```
select 
   cookieid, 
   createtime, 
   pv, 
   sum(pv) over (partition by cookieid order by createtime rows between unbounded preceding and current row) as pv1, 
   sum(pv) over (partition by cookieid order by createtime) as pv2, 
   sum(pv) over (partition by cookieid) as pv3, 
   sum(pv) over (partition by cookieid order by createtime rows between 3 preceding and current row) as pv4, 
   sum(pv) over (partition by cookieid order by createtime rows between 3 preceding and 1 following) as pv5, 
   sum(pv) over (partition by cookieid order by createtime rows between current row and unbounded following) as pv6 
from cookie1;
```

### 查询结果

![img](https://images2018.cnblogs.com/blog/1228818/201804/1228818-20180410211647233-1245106876.png)

### 说明

```
pv1: 分组内从起点到当前行的pv累积，如，11号的pv1=10号的pv+11号的pv, 12号=10号+11号+12号
pv2: 同pv1
pv3: 分组内(cookie1)所有的pv累加
pv4: 分组内当前行+往前3行，如，11号=10号+11号， 12号=10号+11号+12号， 13号=10号+11号+12号+13号， 14号=11号+12号+13号+14号
pv5: 分组内当前行+往前3行+往后1行，如，14号=11号+12号+13号+14号+15号=5+7+3+2+4=21
pv6: 分组内当前行+往后所有行，如，13号=13号+14号+15号+16号=3+2+4+4=13，14号=14号+15号+16号=2+4+4=10
```

如果不指定ROWS BETWEEN,默认为从起点到当前行;
如果不指定ORDER BY，则将分组内所有值累加;
关键是理解ROWS BETWEEN含义,也叫做WINDOW子句：
PRECEDING：往前
FOLLOWING：往后
CURRENT ROW：当前行
UNBOUNDED：起点，

　　UNBOUNDED PRECEDING 表示从前面的起点，

　　UNBOUNDED FOLLOWING：表示到后面的终点
–其他AVG，MIN，MAX，和SUM用法一样。

## AVG

### 查询语句

```
select 
   cookieid, 
   createtime, 
   pv, 
   avg(pv) over (partition by cookieid order by createtime rows between unbounded preceding and current row) as pv1, -- 默认为从起点到当前行
   avg(pv) over (partition by cookieid order by createtime) as pv2, --从起点到当前行，结果同pv1
   avg(pv) over (partition by cookieid) as pv3, --分组内所有行
   avg(pv) over (partition by cookieid order by createtime rows between 3 preceding and current row) as pv4, --当前行+往前3行
   avg(pv) over (partition by cookieid order by createtime rows between 3 preceding and 1 following) as pv5, --当前行+往前3行+往后1行
   avg(pv) over (partition by cookieid order by createtime rows between current row and unbounded following) as pv6  --当前行+往后所有行
from cookie1;
```

### 查询结果

![img](https://images2018.cnblogs.com/blog/1228818/201804/1228818-20180410212726807-1076693371.png)

## MIN

### 查询语句

```
select 
   cookieid, 
   createtime, 
   pv, 
   min(pv) over (partition by cookieid order by createtime rows between unbounded preceding and current row) as pv1, -- 默认为从起点到当前行
   min(pv) over (partition by cookieid order by createtime) as pv2, --从起点到当前行，结果同pv1
   min(pv) over (partition by cookieid) as pv3, --分组内所有行
   min(pv) over (partition by cookieid order by createtime rows between 3 preceding and current row) as pv4, --当前行+往前3行
   min(pv) over (partition by cookieid order by createtime rows between 3 preceding and 1 following) as pv5, --当前行+往前3行+往后1行
   min(pv) over (partition by cookieid order by createtime rows between current row and unbounded following) as pv6  --当前行+往后所有行
from cookie1;
```

### 查询结果 

![img](https://images2018.cnblogs.com/blog/1228818/201804/1228818-20180410212927297-2042376017.png)

## MAX

### 查询语句

```
select 
   cookieid, 
   createtime, 
   pv, 
   max(pv) over (partition by cookieid order by createtime rows between unbounded preceding and current row) as pv1, -- 默认为从起点到当前行
   max(pv) over (partition by cookieid order by createtime) as pv2, --从起点到当前行，结果同pv1
   max(pv) over (partition by cookieid) as pv3, --分组内所有行
   max(pv) over (partition by cookieid order by createtime rows between 3 preceding and current row) as pv4, --当前行+往前3行
   max(pv) over (partition by cookieid order by createtime rows between 3 preceding and 1 following) as pv5, --当前行+往前3行+往后1行
   max(pv) over (partition by cookieid order by createtime rows between current row and unbounded following) as pv6  --当前行+往后所有行
from cookie1;
```

### 查询结果

![img](https://images2018.cnblogs.com/blog/1228818/201804/1228818-20180410213346492-1184970532.png)