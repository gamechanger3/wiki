<p align="center">
    <img width="280px" src="image/dongwu/dog/10.png" >
</p>

# Hive（十）Hive SQL练习之影评案例

## 案例说明

现有如此三份数据：
1、users.dat 数据格式为： 2::M::56::16::70072，

共有6040条数据
对应字段为：UserID BigInt, Gender String, Age Int, Occupation String, Zipcode String
对应字段中文解释：用户id，性别，年龄，职业，邮政编码

2、movies.dat 数据格式为： 2::Jumanji (1995)::Adventure|Children's|Fantasy，

共有3883条数据
对应字段为：MovieID BigInt, Title String, Genres String
对应字段中文解释：电影ID，电影名字，电影类型

3、ratings.dat 数据格式为： 1::1193::5::978300760，

共有1000209条数据
对应字段为：UserID BigInt, MovieID BigInt, Rating Double, Timestamped String
对应字段中文解释：用户ID，电影ID，评分，评分时间戳

题目要求

　　数据要求：
　　　　（1）写shell脚本清洗数据。（hive不支持解析多字节的分隔符，也就是说hive只能解析':', 不支持解析'::'，所以用普通方式建表来使用是行不通的，要求对数据做一次简单清洗）
　　　　（2）使用Hive能解析的方式进行

　　Hive要求：
　　　　（1）正确建表，导入数据（三张表，三份数据），并验证是否正确

　　　　（2）求被评分次数最多的10部电影，并给出评分次数（电影名，评分次数）

　　　　（3）分别求男性，女性当中评分最高的10部电影（性别，电影名，影评分）

　　　　（4）求movieid = 2116这部电影各年龄段（因为年龄就只有7个，就按这个7个分就好了）的平均影评（年龄段，影评分）

　　　　（5）求最喜欢看电影（影评次数最多）的那位女性评最高分的10部电影的平均影评分（观影者，电影名，影评分）

　　　　（6）求好片（评分>=4.0）最多的那个年份的最好看的10部电影

　　　　（7）求1997年上映的电影中，评分最高的10部Comedy类电影

　　　　（8）该影评库中各种类型电影中评价最高的5部电影（类型，电影名，平均影评分）

　　　　（9）各年评分最高的电影类型（年份，类型，影评分）

　　　　（10）每个地区最高评分的电影名，把结果存入HDFS（地区，电影名，影评分）

## 数据下载

[https://files.cnblogs.com/files/qingyunzong/hive%E5%BD%B1%E8%AF%84%E6%A1%88%E4%BE%8B.zip](https://files.cnblogs.com/files/qingyunzong/hive影评案例.zip)

## 解析

之前已经使用MapReduce程序将3张表格进行合并，所以只需要将合并之后的表格导入对应的表中进行查询即可。

### 1、正确建表，导入数据（三张表，三份数据），并验证是否正确

#### （1）分析需求

需要创建一个数据库movie，在movie数据库中创建3张表，t_user，t_movie，t_rating

> **t_user**:userid bigint,sex string,age int,occupation string,zipcode string
> **t_movie**:movieid bigint,moviename string,movietype string
> **t_rating**:userid bigint,movieid bigint,rate double,times string

原始数据是以::进行切分的，所以需要使用能解析多字节分隔符的Serde即可

使用RegexSerde

需要两个参数：
input.regex = "(.*)::(.*)::(.*)"
output.format.string = "%1$s %2$s %3$s"

#### （2）创建数据库

```
drop database if exists movie;
create database if not exists movie;
use movie;
```

#### （3）创建t_user表

```
create table t_user(
userid bigint,
sex string,
age int,
occupation string,
zipcode string) 
row format serde 'org.apache.hadoop.hive.serde2.RegexSerDe' 
with serdeproperties('input.regex'='(.*)::(.*)::(.*)::(.*)::(.*)','output.format.string'='%1$s %2$s %3$s %4$s %5$s')
stored as textfile;
```

#### （4）创建t_movie表

```
use movie;
create table t_movie(
movieid bigint,
moviename string,
movietype string) 
row format serde 'org.apache.hadoop.hive.serde2.RegexSerDe' 
with serdeproperties('input.regex'='(.*)::(.*)::(.*)','output.format.string'='%1$s %2$s %3$s')
stored as textfile;
```

#### （5）创建t_rating表

```
use movie;
create table t_rating(
userid bigint,
movieid bigint,
rate double,
times string) 
row format serde 'org.apache.hadoop.hive.serde2.RegexSerDe' 
with serdeproperties('input.regex'='(.*)::(.*)::(.*)::(.*)','output.format.string'='%1$s %2$s %3$s %4$s')
stored as textfile;
```

#### （6）导入数据

```
0: jdbc:hive2://hadoop3:10000> load data local inpath "/home/hadoop/movie/users.dat" into table t_user;
No rows affected (0.928 seconds)
0: jdbc:hive2://hadoop3:10000> load data local inpath "/home/hadoop/movie/movies.dat" into table t_movie;
No rows affected (0.538 seconds)
0: jdbc:hive2://hadoop3:10000> load data local inpath "/home/hadoop/movie/ratings.dat" into table t_rating;
No rows affected (0.963 seconds)
0: jdbc:hive2://hadoop3:10000> 
```

#### （7）验证

```
select t.* from t_user t;
```

![img](https://images2018.cnblogs.com/blog/1228818/201804/1228818-20180410163511811-1216719632.png)

```
select t.* from t_movie t;
```

![img](https://images2018.cnblogs.com/blog/1228818/201804/1228818-20180410163646769-516781465.png)

```
select t.* from t_rating t;
```

![img](https://images2018.cnblogs.com/blog/1228818/201804/1228818-20180410163751235-1412738103.png)



### 2、求被评分次数最多的10部电影，并给出评分次数（电影名，评分次数）

#### （1）思路分析：

　　1、需求字段：电影名    t_movie.moviename

　　　　　　　　 评分次数  t_rating.rate     count()

　　2、核心SQL：按照电影名进行分组统计，求出每部电影的评分次数并按照评分次数降序排序

#### （2）完整SQL：

```
create table answer2 as 
select a.moviename as moviename,count(a.moviename) as total 
from t_movie a join t_rating b on a.movieid=b.movieid 
group by a.moviename 
order by total desc 
limit 10;
```

```
select * from answer2;
```

![img](https://images2018.cnblogs.com/blog/1228818/201804/1228818-20180410164633081-629022259.png)

### 3、分别求男性，女性当中评分最高的10部电影（性别，电影名，影评分）

####  （1）分析思路：

　　1、需求字段：性别　　t_user.sex

　　　　　　　　 电影名　t_movie.moviename

　　　　　　　　 影评分　t_rating.rate

　　2、核心SQL：三表联合查询，按照性别过滤条件，电影名作为分组条件，影评分作为排序条件进行查询

#### （2）完整SQL：

女性当中评分最高的10部电影（性别，电影名，影评分）评论次数大于等于50次

```
create table answer3_F as 
select "F" as sex, c.moviename as name, avg(a.rate) as avgrate, count(c.moviename) as total  
from t_rating a 
join t_user b on a.userid=b.userid 
join t_movie c on a.movieid=c.movieid 
where b.sex="F" 
group by c.moviename 
having total >= 50
order by avgrate desc 
limit 10;
```

```
select * from answer3_F；
```

![img](https://images2018.cnblogs.com/blog/1228818/201804/1228818-20180410170214113-1840938708.png)

男性当中评分最高的10部电影（性别，电影名，影评分）评论次数大于等于50次

```
create table answer3_M as 
select "M" as sex, c.moviename as name, avg(a.rate) as avgrate, count(c.moviename) as total  
from t_rating a 
join t_user b on a.userid=b.userid 
join t_movie c on a.movieid=c.movieid 
where b.sex="M" 
group by c.moviename 
having total >= 50
order by avgrate desc 
limit 10;
```

```
select * from answer3_M；
```

![img](https://images2018.cnblogs.com/blog/1228818/201804/1228818-20180410170316416-1654694572.png)

### 4、求movieid = 2116这部电影各年龄段（因为年龄就只有7个，就按这个7个分就好了）的平均影评（年龄段，影评分）

#### （1）分析思路：

　　1、需求字段：年龄段　　t_user.age

　　　　　　　　 影评分　t_rating.rate

　　2、核心SQL：t_user和t_rating表进行联合查询，用movieid=2116作为过滤条件，用年龄段作为分组条件

#### （2）完整SQL：

```
create table answer4 as 
select a.age as age, avg(b.rate) as avgrate 
from t_user a join t_rating b on a.userid=b.userid 
where b.movieid=2116 
group by a.age;
select * from answer4;
```

![img](https://images2018.cnblogs.com/blog/1228818/201804/1228818-20180410170836707-1154639273.png)

### 5、求最喜欢看电影（影评次数最多）的那位女性评最高分的10部电影的平均影评分（观影者，电影名，影评分）

#### （1）分析思路：

　　1、需求字段：观影者　t_rating.userid

　　　　　　　　 电影名　t_movie.moviename

　　　　　　　　 影评分　t_rating.rate

　　2、核心SQL：

　　　　A.　　需要先求出最喜欢看电影的那位女性

　　　　　　　　　　需要查询的字段：性别：t_user.sex

　　　　　　　　　　　　　　　　　　观影次数：count(t_rating.userid)

　　　　B.　　根据A中求出的女性userid作为where过滤条件，以看过的电影的影评分rate作为排序条件进行排序，求出评分最高的10部电影

　　　　　　　　　　需要查询的字段：电影的ID：t_rating.movieid

　　　　C.　　求出B中10部电影的平均影评分

　　　　　　　　　　需要查询的字段：电影的ID：answer5_B.movieid

　　　　　　　　　　　　　　　　　　影评分：t_rating.rate

#### （2）完整SQL：

 A.　　需要先求出最喜欢看电影的那位女性

```
select a.userid, count(a.userid) as total 
from t_rating a join t_user b on a.userid = b.userid 
where b.sex="F" 
group by a.userid 
order by total desc 
limit 1;
```

![img](https://images2018.cnblogs.com/blog/1228818/201804/1228818-20180410172157137-2052414004.png)

B.　　根据A中求出的女性userid作为where过滤条件，以看过的电影的影评分rate作为排序条件进行排序，求出评分最高的10部电影

```
create table answer5_B as 
select a.movieid as movieid, a.rate as rate  
from t_rating a 
where a.userid=1150 
order by rate desc 
limit 10;
```

```
select * from answer5_B;
```

![img](https://images2018.cnblogs.com/blog/1228818/201804/1228818-20180410172614162-1700464534.png)

C.　　求出B中10部电影的平均影评分

```
create table answer5_C as 
select b.movieid as movieid, c.moviename as moviename, avg(b.rate) as avgrate 
from answer5_B a 
join t_rating b on a.movieid=b.movieid 
join t_movie c on b.movieid=c.movieid 
group by b.movieid,c.moviename;
```

```
select * from answer5_C;
```

![img](https://images2018.cnblogs.com/blog/1228818/201804/1228818-20180410173209987-971071603.png)

### 6、求好片（评分>=4.0）最多的那个年份的最好看的10部电影

#### （1）分析思路：

　　1、需求字段：电影id　t_rating.movieid

　　　　　　　　 电影名　t_movie.moviename（包含年份）

　　　　　　　　 影评分　t_rating.rate

　　　　　　　　 上映年份　xxx.years

　　2、核心SQL：

　　　　A.　　需要将t_rating和t_movie表进行联合查询，将电影名当中的上映年份截取出来，保存到临时表answer6_A中

　　　　　　　　　　需要查询的字段：电影id　t_rating.movieid

　　　　　　　　　　　　　　　　　　电影名　t_movie.moviename（包含年份）

　　　　　　　　　　　　　　　　　　影评分　t_rating.rate

　　　　B.　　从answer6_A按照年份进行分组条件，按照评分>=4.0作为where过滤条件，按照count(years)作为排序条件进行查询

　　　　　　　　　　需要查询的字段：电影的ID：answer6_A.years

　　　　C.　　从answer6_A按照years=1998作为where过滤条件，按照评分作为排序条件进行查询

　　　　　　　　　　需要查询的字段：电影的ID：answer6_A.moviename

　　　　　　　　　　　　　　　　　　影评分：answer6_A.avgrate

#### （2）完整SQL：

A.　　需要将t_rating和t_movie表进行联合查询，将电影名当中的上映年份截取出来

```
create table answer6_A as
select  a.movieid as movieid, a.moviename as moviename, substr(a.moviename,-5,4) as years, avg(b.rate) as avgrate
from t_movie a join t_rating b on a.movieid=b.movieid 
group by a.movieid, a.moviename;
select * from answer6_A;
```

![img](https://images2018.cnblogs.com/blog/1228818/201804/1228818-20180410184026515-566278137.png)

B.　　从answer6_A按照年份进行分组条件，按照评分>=4.0作为where过滤条件，按照count(years)作为排序条件进行查询

```
select years, count(years) as total 
from answer6_A a 
where avgrate >= 4.0 
group by years 
order by total desc 
limit 1;
```

![img](https://images2018.cnblogs.com/blog/1228818/201804/1228818-20180410184710241-171102133.png)

C.　　从answer6_A按照years=1998作为where过滤条件，按照评分作为排序条件进行查询

```
create table answer6_C as
select a.moviename as name, a.avgrate as rate 
from answer6_A a 
where a.years=1998 
order by rate desc 
limit 10;
```

```
select * from answer6_C;
```

![img](https://images2018.cnblogs.com/blog/1228818/201804/1228818-20180410185149876-690755903.png)

### 7、求1997年上映的电影中，评分最高的10部Comedy类电影

#### （1）分析思路：

　　1、需求字段：电影id　t_rating.movieid

　　　　　　　　 电影名　t_movie.moviename（包含年份）

　　　　　　　　 影评分　t_rating.rate

　　　　　　　　 上映年份　xxx.years（最终查询结果可不显示）

　　　　　　　　 电影类型　xxx.type（最终查询结果可不显示）

　　2、核心SQL：

　　　　A.　　需要电影类型，所有可以将第六步中求出answer6_A表和t_movie表进行联合查询

　　　　　　　　　　需要查询的字段：电影id　answer6_A.movieid

　　　　　　　　　　　　　　　　　　电影名　answer6_A.moviename

　　　　　　　　　　　　　　　　　　影评分　answer6_A.rate

　　　　　　　　　　　　　　　　　　电影类型　t_movie.movietype　

　　　　　　　　　　　　　　　　　　上映年份　answer6_A.years

　　　　B.　　从answer7_A按照电影类型中是否包含Comedy和按上映年份作为where过滤条件，按照评分作为排序条件进行查询，将结果保存到answer7_B中

　　　　　　　　　　需要查询的字段：电影的ID：answer7_A.id

　　　　　　　　　　　　　　　　　　电影的名称：answer7_A.name

　　　　　　　　　　　　　　　　　　电影的评分：answer7_A.rate

#### （2）完整SQL：

A.　　需要电影类型，所有可以将第六步中求出answer6_A表和t_movie表进行联合查询

```
create table answer7_A as 
select b.movieid as id, b.moviename as name, b.years as years, b.avgrate as rate, a.movietype as type 
from t_movie a join answer6_A b on a.movieid=b.movieid;
select t.* from answer7_A t;
```

![img](https://images2018.cnblogs.com/blog/1228818/201804/1228818-20180410192030317-443456540.png)

B.　　从answer7_A按照电影类型中是否包含Comedy和按照评分>=4.0作为where过滤条件，按照评分作为排序条件进行查询，将结果保存到answer7_B中

```
create table answer7_B as 
select t.id as id, t.name as name, t.rate as rate 
from answer7_A t 
where t.years=1997 and instr(lcase(t.type),'comedy') >0 
order by rate desc
limit 10;
```

```
select * from answer7_B;
```

![img](https://images2018.cnblogs.com/blog/1228818/201804/1228818-20180410193635839-727010468.png)

### 8、该影评库中各种类型电影中评价最高的5部电影（类型，电影名，平均影评分）

#### （1）分析思路：

　　1、需求字段：电影id　movieid

　　　　　　　　 电影名　moviename

　　　　　　　　 影评分　rate（排序条件）

　　　　　　　　 电影类型　type（分组条件）

　　2、核心SQL：

　　　　A.　　需要电影类型，所有需要将answer7_A中的type字段进行裂变，将结果保存到answer8_A中

　　　　　　　　　　需要查询的字段：电影id　answer7_A.id

　　　　　　　　　　　　　　　　　　电影名　answer7_A.name（包含年份）

　　　　　　　　　　　　　　　　　　上映年份　answer7_A.years

　　　　　　　　　　　　　　　　　　影评分　answer7_A.rate

　　　　　　　　　　　　　　　　　　电影类型　answer7_A.movietype　

　　　　B.　　求TopN，按照type分组，需要添加一列来记录每组的顺序，将结果保存到answer8_B中

> row_number() ：用来生成 num字段的值
>
> distribute by movietype ：按照type进行分组
>
> sort by avgrate desc ：每组数据按照rate排降序
>
> num：新列， 值就是每一条记录在每一组中按照排序规则计算出来的排序值

　　　　C.　　从answer8_B中取出num列序号<=5的

#### （2）完整SQL：

A.　　需要电影类型，所有需要将answer7_A中的type字段进行裂变，将结果保存到answer8_A中

```
create table answer8_A as 
select a.id as id, a.name as name, a.years as years, a.rate as rate, tv.type as type 
from answer7_A a 
lateral view explode(split(a.type,"\\|")) tv as type;
select * from answer8_A; 
```

![img](https://images2018.cnblogs.com/blog/1228818/201804/1228818-20180410194801735-397806737.png)

B.　　求TopN，按照type分组，需要添加一列来记录每组的顺序，将结果保存到answer8_B中

```
create table answer8_B as 
select id,name,years,rate,type,row_number() over(distribute by type sort by rate desc ) as num
from answer8_A;
select * from answer8_B;
```

![img](https://images2018.cnblogs.com/blog/1228818/201804/1228818-20180410195900115-1011015068.png)

C.　　从answer8_B中取出num列序号<=5的

```
select a.* from answer8_B a where a.num <= 5;
```

![img](https://images2018.cnblogs.com/blog/1228818/201804/1228818-20180410200238591-1498325084.png)

### 9、各年评分最高的电影类型（年份，类型，影评分）

#### （1）分析思路：

　　1、需求字段：电影id　movieid

　　　　　　　　 电影名　moviename

　　　　　　　　 影评分　rate（排序条件）

　　　　　　　　 电影类型　type（分组条件）

　　　　　　　　 上映年份　years（分组条件）

　　2、核心SQL：

　　　　A.　　需要按照电影类型和上映年份进行分组，按照影评分进行排序，将结果保存到answer9_A中

　　　　　　　　　　需要查询的字段：

　　　　　　　　　　　　　　　　　　上映年份　answer7_A.years

　　　　　　　　　　　　　　　　　　影评分　answer7_A.rate

　　　　　　　　　　　　　　　　　　电影类型　answer7_A.movietype　

　　　　B.　　求TopN，按照years分组，需要添加一列来记录每组的顺序，将结果保存到answer9_B中

　　　　C.　　按照num=1作为where过滤条件取出结果数据

#### （2）完整SQL：

A.　　需要按照电影类型和上映年份进行分组，按照影评分进行排序，将结果保存到answer9_A中

```
create table answer9_A as 
select a.years as years, a.type as type, avg(a.rate) as rate 
from answer8_A a 
group by a.years,a.type 
order by rate desc;
select * from answer9_A;
```

![img](https://images2018.cnblogs.com/blog/1228818/201804/1228818-20180410201710877-165323305.png)

B.　　求TopN，按照years分组，需要添加一列来记录每组的顺序，将结果保存到answer9_B中

```
create table answer9_B as 
select years,type,rate,row_number() over (distribute by years sort by rate) as num
from answer9_A;
select * from answer9_B;
```

![img](https://images2018.cnblogs.com/blog/1228818/201804/1228818-20180410202105613-1494957407.png)

C.　　按照num=1作为where过滤条件取出结果数据

```
select * from answer9_B where num=1;
```

![img](https://images2018.cnblogs.com/blog/1228818/201804/1228818-20180410202237858-1872842838.png)

### 10、每个地区最高评分的电影名，把结果存入HDFS（地区，电影名，影评分）

#### （1）分析思路：

　　1、需求字段：电影id　t_movie.movieid

　　　　　　　　 电影名　t_movie.moviename

　　　　　　　　 影评分　t_rating.rate（排序条件）

　　　　　　　　 地区　t_user.zipcode（分组条件）

　　2、核心SQL：

　　　　A.　　需要把三张表进行联合查询，取出电影id、电影名称、影评分、地区，将结果保存到answer10_A表中

　　　　　　　　　　需要查询的字段：电影id　t_movie.movieid

　　　　　　　　 　　　　　　　　　 电影名　t_movie.moviename

　　　　　　　　 　　　　　　　　　 影评分　t_rating.rate（排序条件）

　　　　　　　　 　　　　　　　　　 地区　t_user.zipcode（分组条件）

　　　　B.　　求TopN，按照地区分组，按照平均排序，添加一列num用来记录地区排名，将结果保存到answer10_B表中

　　　　C.　　按照num=1作为where过滤条件取出结果数据

#### （2）完整SQL：

 A.　　需要把三张表进行联合查询，取出电影id、电影名称、影评分、地区，将结果保存到answer10_A表中

```
create table answer10_A as
select c.movieid, c.moviename, avg(b.rate) as avgrate, a.zipcode
from t_user a 
join t_rating b on a.userid=b.userid 
join t_movie c on b.movieid=c.movieid 
group by a.zipcode,c.movieid, c.moviename;
```

```
select t.* from answer10_A t;
```

![img](https://images2018.cnblogs.com/blog/1228818/201804/1228818-20180410203328988-1213590724.png)

B.　　求TopN，按照地区分组，按照平均排序，添加一列num用来记录地区排名，将结果保存到answer10_B表中

```
create table answer10_B as
select movieid,moviename,avgrate,zipcode, row_number() over (distribute by zipcode sort by avgrate) as num 
from answer10_A; 
select t.* from answer10_B t;
```

![img](https://images2018.cnblogs.com/blog/1228818/201804/1228818-20180410204043457-1325388358.png)

C.　　按照num=1作为where过滤条件取出结果数据并保存到HDFS上

```
insert overwrite directory "/movie/answer10/" select t.* from answer10_B t where t.num=1;
```

![img](https://images2018.cnblogs.com/blog/1228818/201804/1228818-20180410204402851-479530041.png)