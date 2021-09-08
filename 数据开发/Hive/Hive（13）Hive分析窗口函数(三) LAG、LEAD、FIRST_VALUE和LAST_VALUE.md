<p align="center">
    <img width="280px" src="image/dongwu/dog/13.png" >
</p>

# Hive（十三）Hive分析窗口函数(三) LAG、LEAD、FIRST_VALUE和LAST_VALUE

## 数据准备

### 数据格式

cookie4.txt

```
cookie1,2015-04-10 10:00:02,url2
cookie1,2015-04-10 10:00:00,url1
cookie1,2015-04-10 10:03:04,1url3
cookie1,2015-04-10 10:50:05,url6
cookie1,2015-04-10 11:00:00,url7
cookie1,2015-04-10 10:10:00,url4
cookie1,2015-04-10 10:50:01,url5
cookie2,2015-04-10 10:00:02,url22
cookie2,2015-04-10 10:00:00,url11
cookie2,2015-04-10 10:03:04,1url33
cookie2,2015-04-10 10:50:05,url66
cookie2,2015-04-10 11:00:00,url77
cookie2,2015-04-10 10:10:00,url44
cookie2,2015-04-10 10:50:01,url55
```

### 创建表

```
use cookie;
drop table if exists cookie4;
create table cookie4(cookieid string, createtime string, url string) 
row format delimited fields terminated by ',';
load data local inpath "/home/hadoop/cookie4.txt" into table cookie4;
select * from cookie4;
```

![img](https://images2018.cnblogs.com/blog/1228818/201804/1228818-20180411201621843-342659508.png)

## LAG

### 说明

LAG(col,n,DEFAULT) 用于统计窗口内往上第n行值

> 第一个参数为列名，
> 第二个参数为往上第n行（可选，默认为1），
> 第三个参数为默认值（当往上第n行为NULL时候，取默认值，如不指定，则为NULL）

### 查询语句

```
select 
  cookieid, 
  createtime, 
  url, 
  row_number() over (partition by cookieid order by createtime) as rn, 
  LAG(createtime,1,'1970-01-01 00:00:00') over (partition by cookieid order by createtime) as last_1_time, 
  LAG(createtime,2) over (partition by cookieid order by createtime) as last_2_time 
from cookie.cookie4;
```

### 查询结果

![img](https://images2018.cnblogs.com/blog/1228818/201804/1228818-20180411202336839-169636398.png)

### 结果说明

```
last_1_time: 指定了往上第1行的值，default为'1970-01-01 00:00:00'  
　　　　　　　　cookie1第一行，往上1行为NULL,因此取默认值 1970-01-01 00:00:00
　　　　　　　　cookie1第三行，往上1行值为第二行值，2015-04-10 10:00:02
　　　　　　　　cookie1第六行，往上1行值为第五行值，2015-04-10 10:50:01
last_2_time: 指定了往上第2行的值，为指定默认值
　　　　　　　　cookie1第一行，往上2行为NULL
　　　　　　　　cookie1第二行，往上2行为NULL
　　　　　　　　cookie1第四行，往上2行为第二行值，2015-04-10 10:00:02
　　　　　　　　cookie1第七行，往上2行为第五行值，2015-04-10 10:50:01
```

## LEAD

### 说明

与LAG相反

LEAD(col,n,DEFAULT) 用于统计窗口内往下第n行值

> 第一个参数为列名，
> 第二个参数为往下第n行（可选，默认为1），
> 第三个参数为默认值（当往下第n行为NULL时候，取默认值，如不指定，则为NULL）

### 查询语句

```
select 
  cookieid, 
  createtime, 
  url, 
  row_number() over (partition by cookieid order by createtime) as rn, 
  LEAD(createtime,1,'1970-01-01 00:00:00') over (partition by cookieid order by createtime) as next_1_time, 
  LEAD(createtime,2) over (partition by cookieid order by createtime) as next_2_time 
from cookie.cookie4;
```

### 查询结果

### ![img](https://images2018.cnblogs.com/blog/1228818/201804/1228818-20180411203316368-518018530.png)

### 结果说明

--逻辑与LAG一样，只不过LAG是往上，LEAD是往下。

## FIRST_VALUE

### 说明

取分组内排序后，截止到当前行，第一个值

### 查询语句

```
select 
  cookieid, 
  createtime, 
  url, 
  row_number() over (partition by cookieid order by createtime) as rn, 
  first_value(url) over (partition by cookieid order by createtime) as first1 
from cookie.cookie4;
```

### 查询结果

![img](https://images2018.cnblogs.com/blog/1228818/201804/1228818-20180411204128827-493017140.png)

## LAST_VALUE

### 说明

取分组内排序后，截止到当前行，最后一个值

### 查询语句

```
select 
  cookieid, 
  createtime, 
  url, 
  row_number() over (partition by cookieid order by createtime) as rn, 
  last_value(url) over (partition by cookieid order by createtime) as last1 
from cookie.cookie4;
```

### 查询结果

### ![img](https://images2018.cnblogs.com/blog/1228818/201804/1228818-20180411205221871-1107163476.png)

### 如果不指定ORDER BY，则默认按照记录在文件中的偏移量进行排序，会出现错误的结果

![img](https://images2018.cnblogs.com/blog/1228818/201804/1228818-20180411205841092-1288201073.png)

### **如果想要取分组内排序后最后一个值，则需要变通一下**

#### **查询语句**

```
select 
  cookieid, 
  createtime, 
  url, 
  row_number() over (partition by cookieid order by createtime) as rn,
  LAST_VALUE(url) over (partition by cookieid order by createtime) as last1,
  FIRST_VALUE(url) over (partition by cookieid order by createtime desc) as last2 
from cookie.cookie4 
order by cookieid,createtime;
```

#### 查询结果

 ![img](https://images2018.cnblogs.com/blog/1228818/201804/1228818-20180411211123369-588038997.png)

**提示：在使用分析函数的过程中，要特别注意ORDER BY子句，用的不恰当，统计出的结果就不是你所期望的。**

