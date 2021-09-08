<p align="center">
    <img width="280px" src="image/dongwu/1/k9.png" >
</p>

# Hive（九）Hive SQL面试题

## 一、求单月访问次数和总访问次数

### 1、数据说明

#### 数据字段说明

```
用户名，月份，访问次数
```

#### 数据格式

```
A,2015-01,5
A,2015-01,15
B,2015-01,5
A,2015-01,8
B,2015-01,25
A,2015-01,5
A,2015-02,4
A,2015-02,6
B,2015-02,10
B,2015-02,5
A,2015-03,16
A,2015-03,22
B,2015-03,23
B,2015-03,10
B,2015-03,1
```

### 2、数据准备

#### （1）创建表

```
use myhive;
create external table if not exists t_access(
uname string comment '用户名',
umonth string comment '月份',
ucount int comment '访问次数'
) comment '用户访问表' 
row format delimited fields terminated by "," 
location "/hive/t_access"; 
```

#### （2）导入数据

```
load data local inpath "/home/hadoop/access.txt" into table t_access;
```

#### （3）验证数据

```
select * from t_access;
```

![img](https://images2018.cnblogs.com/blog/1228818/201804/1228818-20180408191422095-618397996.png)

### 3、结果需求

现要求出：
每个用户截止到每月为止的最大单月访问次数和累计到该月的总访问次数，结果数据格式如下

![img](https://images2018.cnblogs.com/blog/1228818/201804/1228818-20180408191641383-2108507539.png)

### 4、需求分析

此结果需要根据用户+月份进行分组

#### （1）先求出当月访问次数

```
--求当月访问次数
create table tmp_access(
name string,
mon string,
num int
); 

insert into table tmp_access 
select uname,umonth,sum(ucount)
 from t_access t group by t.uname,t.umonth;select * from tmp_access;
```

![img](https://images2018.cnblogs.com/blog/1228818/201804/1228818-20180408212426965-852872067.png)

#### （2）tmp_access进行自连接视图

```
create view tmp_view as 
select a.name anme,a.mon amon,a.num anum,b.name bname,b.mon bmon,b.num bnum from tmp_access a join tmp_access b 
on a.name=b.name;

select * from tmp_view;
```

![img](https://images2018.cnblogs.com/blog/1228818/201804/1228818-20180408213008759-536376094.png)

#### （3）进行比较统计

```
select anme,amon,anum,max(bnum) as max_access,sum(bnum) as sum_access 
from tmp_view 
where amon>=bmon 
group by anme,amon,anum;
```

![img](https://images2018.cnblogs.com/blog/1228818/201804/1228818-20180408213548473-1116107765.png)

## 二、学生课程成绩

###  1、说明

```
use myhive;
CREATE TABLE `course` (
  `id` int,
  `sid` int ,
  `course` string,
  `score` int 
) ;
```

```
// 插入数据
// 字段解释：id, 学号， 课程， 成绩
INSERT INTO `course` VALUES (1, 1, 'yuwen', 43);
INSERT INTO `course` VALUES (2, 1, 'shuxue', 55);
INSERT INTO `course` VALUES (3, 2, 'yuwen', 77);
INSERT INTO `course` VALUES (4, 2, 'shuxue', 88);
INSERT INTO `course` VALUES (5, 3, 'yuwen', 98);
INSERT INTO `course` VALUES (6, 3, 'shuxue', 65);
```

![img](https://images2018.cnblogs.com/blog/1228818/201804/1228818-20180408215422080-584424294.png)

### 2、需求

求：所有数学课程成绩 大于 语文课程成绩的学生的学号

#### 1、使用case...when...将不同的课程名称转换成不同的列

```
create view tmp_course_view as
select sid, case course when "shuxue" then score else 0 end  as shuxue,  
case course when "yuwen" then score else 0 end  as yuwen from course;  

select * from tmp_course_view;
```

![img](https://images2018.cnblogs.com/blog/1228818/201804/1228818-20180408215759602-1208331286.png)

#### 2、以sid分组合并取各成绩最大值

```
create view tmp_course_view1 as
select aa.sid, max(aa.shuxue) as shuxue, max(aa.yuwen) as yuwen from tmp_course_view aa group by sid;  

select * from tmp_course_view1;
```

![img](https://images2018.cnblogs.com/blog/1228818/201804/1228818-20180408220127658-1596886525.png)

#### 3、比较结果

```
select * from tmp_course_view1 where shuxue > yuwen;
```

![img](https://images2018.cnblogs.com/blog/1228818/201804/1228818-20180408220333411-2122468849.png)

## 三、求每一年最大气温的那一天 + 温度

###  1、说明

数据格式

```
2010012325
```

具体数据

![img](https://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif) View Code

数据解释

```
2010012325表示在2010年01月23日的气温为25度
```

### 2、 需求

比如：2010012325表示在2010年01月23日的气温为25度。现在要求使用hive，计算每一年出现过的最大气温的日期+温度。
要计算出每一年的最大气温。我用
select substr(data,1,4),max(substr(data,9,2)) from table2 group by substr(data,1,4);
出来的是 年份 + 温度 这两列数据例如 2015 99

但是如果我是想select 的是：具体每一年最大气温的那一天 + 温度 。例如 20150109 99
请问该怎么执行hive语句。。
group by 只需要substr(data,1,4)，
但是select substr(data,1,8)，又不在group by 的范围内。
是我陷入了思维死角。一直想不出所以然。。求大神指点一下。
在select 如果所需要的。不在group by的条件里。这种情况如何去分析？

### 3、解析

#### （1）创建一个临时表tmp_weather，将数据切分

```
create table tmp_weather as 
select substr(data,1,4) years,substr(data,5,2) months,substr(data,7,2) days,substr(data,9,2) temp from weather;
select * from tmp_weather;
```

![img](https://images2018.cnblogs.com/blog/1228818/201804/1228818-20180409100419037-1519099272.png)

#### （2）创建一个临时表tmp_year_weather

```
create table tmp_year_weather as 
select substr(data,1,4) years,max(substr(data,9,2)) max_temp from weather group by substr(data,1,4);
select * from tmp_year_weather;
```

![img](https://images2018.cnblogs.com/blog/1228818/201804/1228818-20180409100254081-790110773.png)

#### （3）将2个临时表进行连接查询

```
select * from tmp_year_weather a join tmp_weather b on a.years=b.years and a.max_temp=b.temp;
```

![img](https://images2018.cnblogs.com/blog/1228818/201804/1228818-20180409100655695-981143896.png)

## 四、求学生选课情况

## 1、数据说明

#### （1）数据格式

```
id course 
1,a 
1,b 
1,c 
1,e 
2,a 
2,c 
2,d 
2,f 
3,a 
3,b 
3,c 
3,e
```

#### （2）字段含义

表示有id为1,2,3的学生选修了课程a,b,c,d,e,f中其中几门。

## 2、数据准备

#### （1）建表t_course

```
create table t_course(id int,course string)
row format delimited fields terminated by ",";
```

![img](https://images2018.cnblogs.com/blog/1228818/201804/1228818-20180409101747464-781937248.png)

#### （2）导入数据

```
load data local inpath "/home/hadoop/course/course.txt" into table t_course;
```

![img](https://images2018.cnblogs.com/blog/1228818/201804/1228818-20180409101834214-242067281.png)

## 3、需求

编写Hive的HQL语句来实现以下结果：表中的1表示选修，表中的0表示未选修

```
id    a    b    c    d    e    f
1     1    1    1    0    1    0
2     1    0    1    1    0    1
3     1    1    1    0    1    0
```

## 4、解析

第一步：

```
select collect_set(course) as courses from id_course;
```

第二步：

```
set hive.strict.checks.cartesian.product=false;

create table id_courses as select t1.id as id,t1.course as id_courses,t2.course courses 
from 
( select id as id,collect_set(course) as course from id_course group by id ) t1 
join 
(select collect_set(course) as course from id_course) t2;
```

> 启用严格模式：hive.mapred.mode = strict // Deprecated
> hive.strict.checks.large.query = true
> 该设置会禁用：1. 不指定分页的orderby
> 　　　　　　  2. 对分区表不指定分区进行查询 
> 　　　　　　  3. 和数据量无关，只是一个查询模式
>
> hive.strict.checks.type.safety = true
> 严格类型安全，该属性不允许以下操作：1. bigint和string之间的比较
> 　　　　　　　　　　　　　　　　　　2. bigint和double之间的比较
>
> hive.strict.checks.cartesian.product = true
> 该属性不允许笛卡尔积操作

第三步：得出最终结果：
思路：
拿出course字段中的每一个元素在id_courses中进行判断，看是否存在。

```
select id,
case when array_contains(id_courses, courses[0]) then 1 else 0 end as a,
case when array_contains(id_courses, courses[1]) then 1 else 0 end as b,
case when array_contains(id_courses, courses[2]) then 1 else 0 end as c,
case when array_contains(id_courses, courses[3]) then 1 else 0 end as d,
case when array_contains(id_courses, courses[4]) then 1 else 0 end as e,
case when array_contains(id_courses, courses[5]) then 1 else 0 end as f 
from id_courses;
```

## 五、求月销售额和总销售额

### 1、数据说明

#### （1）数据格式

```
a,01,150
a,01,200
b,01,1000
b,01,800
c,01,250
c,01,220
b,01,6000
a,02,2000
a,02,3000
b,02,1000
b,02,1500
c,02,350
c,02,280
a,03,350
a,03,250
```

#### （2）字段含义

店铺，月份，金额

### 2、数据准备

#### （1）创建数据库表t_store

```
use class;
create table t_store(
name string,
months int,
money int
) 
row format delimited fields terminated by ",";
```

#### （2）导入数据

```
load data local inpath "/home/hadoop/store.txt" into table t_store;
```

### 3、需求

编写Hive的HQL语句求出每个店铺的当月销售额和累计到当月的总销售额

### 4、解析

（1）按照商店名称和月份进行分组统计

```
create table tmp_store1 as 
select name,months,sum(money) as money from t_store group by name,months;

select * from tmp_store1;
```

![img](https://images2018.cnblogs.com/blog/1228818/201804/1228818-20180409113229256-890349636.png)

（2）对tmp_store1 表里面的数据进行自连接

```
create table tmp_store2 as 
select a.name aname,a.months amonths,a.money amoney,b.name bname,b.months bmonths,b.money bmoney from tmp_store1 a 
join tmp_store1 b on a.name=b.name order by aname,amonths;

select * from tmp_store2;
```

![img](https://images2018.cnblogs.com/blog/1228818/201804/1228818-20180409113405020-775197507.png)

（3）比较统计

```
select aname,amonths,amoney,sum(bmoney) as total from tmp_store2 where amonths >= bmonths group by aname,amonths,amoney;
```

![img](https://images2018.cnblogs.com/blog/1228818/201804/1228818-20180409113709570-551757814.png)