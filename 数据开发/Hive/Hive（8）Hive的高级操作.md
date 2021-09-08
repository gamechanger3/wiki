<p align="center">
    <img width="280px" src="image/dongwu/dog/8.png" >
</p>

# Hive（八）Hive的高级操作

## 一、复杂数据类型

### 1、array

 现有数据如下：

1 huangbo guangzhou,xianggang,shenzhen a1:30,a2:20,a3:100 beijing,112233,13522334455,500
2 xuzheng xianggang b2:50,b3:40 tianjin,223344,13644556677,600
3 wangbaoqiang beijing,zhejinag c1:200 chongqinjg,334455,15622334455,20

建表语句

```sql
use class;
create table cdt(
id int, 
name string, 
work_location array<string>, 
piaofang map<string,bigint>, 
address struct<location:string,zipcode:int,phone:string,value:int>) 
row format delimited 
fields terminated by "\t" 
collection items terminated by "," 
map keys terminated by ":" 
lines terminated by "\n";
```



![img](https://images2018.cnblogs.com/blog/1228818/201804/1228818-20180408173043585-1275161000.png)

导入数据

```
0: jdbc:hive2://hadoop3:10000> load data local inpath "/home/hadoop/cdt.txt" into table cdt;
```

查询语句

```
select * from cdt;
```

![img](https://images2018.cnblogs.com/blog/1228818/201804/1228818-20180408183049458-1912277812.png)

```
select name from cdt;
```

![img](https://images2018.cnblogs.com/blog/1228818/201804/1228818-20180408183202688-414846691.png)

```
select work_location from cdt;
```

![img](https://images2018.cnblogs.com/blog/1228818/201804/1228818-20180408183336575-889255211.png)

```
select work_location[0] from cdt;
```

![img](https://images2018.cnblogs.com/blog/1228818/201804/1228818-20180408183446633-233162014.png)

```
select work_location[1] from cdt;
```

![img](https://images2018.cnblogs.com/blog/1228818/201804/1228818-20180408183519651-594024573.png)

### 2、map

建表语句、导入数据同1

查询语句

```
select piaofang from cdt;
```

![img](https://images2018.cnblogs.com/blog/1228818/201804/1228818-20180408183748427-1339156513.png)

```
select piaofang["a1"] from cdt;
```

 ![img](https://images2018.cnblogs.com/blog/1228818/201804/1228818-20180408183858137-827620470.png)

### 3、struct

建表语句、导入数据同1

查询语句

```
select address from cdt;
```

![img](https://images2018.cnblogs.com/blog/1228818/201804/1228818-20180408184016402-1457666185.png)

```
select address.location from cdt;
```

 ![img](https://images2018.cnblogs.com/blog/1228818/201804/1228818-20180408184050383-1588329051.png)

### 4、uniontype

很少使用

参考资料：http://yugouai.iteye.com/blog/1849192

## 二、视图

### 1、Hive 的视图和关系型数据库的视图区别

和关系型数据库一样，Hive 也提供了视图的功能，不过请注意，Hive 的视图和关系型数据库的数据还是有很大的区别：

　　（1）只有逻辑视图，没有物化视图；

　　（2）视图只能查询，不能 Load/Insert/Update/Delete 数据；

　　（3）视图在创建时候，只是保存了一份元数据，当查询视图的时候，才开始执行视图对应的 那些子查询

### 2、Hive视图的创建语句

```
create view view_cdt as select * from cdt;
```

![img](https://images2018.cnblogs.com/blog/1228818/201804/1228818-20180408184549702-1005718852.png)

### 3、Hive视图的查看语句

```
show views;
desc view_cdt;-- 查看某个具体视图的信息
```

![img](https://images2018.cnblogs.com/blog/1228818/201804/1228818-20180408184741330-1744317581.png)



### 4、Hive视图的使用语句

```
select * from view_cdt;
```

![img](https://images2018.cnblogs.com/blog/1228818/201804/1228818-20180408184854110-707403466.png)



### 5、Hive视图的删除语句

```
drop view view_cdt;
```

![img](https://images2018.cnblogs.com/blog/1228818/201804/1228818-20180408185042082-261986868.png)



## 三、函数

### 1、内置函数

具体可看http://www.cnblogs.com/qingyunzong/p/8744593.html

#### （1）查看内置函数

```
show functions;
```

![img](https://images2018.cnblogs.com/blog/1228818/201804/1228818-20180408185346738-551528669.png)

#### （2）显示函数的详细信息

```
desc function substr;
```

![img](https://images2018.cnblogs.com/blog/1228818/201804/1228818-20180408185504054-1979368578.png)

#### （3）显示函数的扩展信息

```
desc function extended substr;
```

![img](https://images2018.cnblogs.com/blog/1228818/201804/1228818-20180408185612437-1436305025.png)



### 2、自定义函数UDF

当 Hive 提供的内置函数无法满足业务处理需要时，此时就可以考虑使用用户自定义函数。

**UDF**（user-defined function）作用于单个数据行，产生一个数据行作为输出。（数学函数，字 符串函数）

**UDAF**（用户定义聚集函数 User- Defined Aggregation Funcation）：接收多个输入数据行，并产 生一个输出数据行。（count，max）

**UDTF**（表格生成函数 User-Defined Table Functions）：接收一行输入，输出多行（explode）

### (1) 简单UDF示例

#### A.　导入hive需要的jar包，自定义一个java类继承UDF，重载 evaluate 方法

ToLowerCase.java

```
import org.apache.hadoop.hive.ql.exec.UDF;

public class ToLowerCase extends UDF{
    
    // 必须是 public，并且 evaluate 方法可以重载
    public String evaluate(String field) {
    String result = field.toLowerCase();
    return result;
    }
    
}
```



#### B.　打成 jar 包上传到服务器

#### C.　将 jar 包添加到 hive 的 classpath

```
add JAR /home/hadoop/udf.jar;
```

![img](https://images2018.cnblogs.com/blog/1228818/201804/1228818-20180414105512185-1132423466.png)

#### D.　创建临时函数与开发好的 class 关联起来

```
0: jdbc:hive2://hadoop3:10000> create temporary function tolowercase as 'com.study.hive.udf.ToLowerCase';
```

![img](https://images2018.cnblogs.com/blog/1228818/201804/1228818-20180414105730255-1712623480.png)

#### E.　至此，便可以在 hql 在使用自定义的函数

```
0: jdbc:hive2://hadoop3:10000> select tolowercase('HELLO');
```

![img](https://images2018.cnblogs.com/blog/1228818/201804/1228818-20180414105920517-647807254.png)



### (2) JSON数据解析UDF开发

现有原始 json 数据（rating.json）如下

> {"movie":"1193","rate":"5","timeStamp":"978300760","uid":"1"}
>
> {"movie":"661","rate":"3","timeStamp":"978302109","uid":"1"}
>
> {"movie":"914","rate":"3","timeStamp":"978301968","uid":"1"}
>
> {"movie":"3408","rate":"4","timeStamp":"978300275","uid":"1"}
>
> {"movie":"2355","rate":"5","timeStamp":"978824291","uid":"1"}
>
> {"movie":"1197","rate":"3","timeStamp":"978302268","uid":"1"}
>
> {"movie":"1287","rate":"5","timeStamp":"978302039","uid":"1"}
>
> {"movie":"2804","rate":"5","timeStamp":"978300719","uid":"1"}
>
> {"movie":"594","rate":"4","timeStamp":"978302268","uid":"1"}

现在需要将数据导入到 hive 仓库中，并且最终要得到这么一个结果：

![img](https://images2018.cnblogs.com/blog/1228818/201804/1228818-20180414110149949-679183333.png)

该怎么做、？？？（提示：可用内置 get_json_object 或者自定义函数完成）

#### A.　get_json_object(string json_string, string path)

返回值: string 

说明：解析json的字符串json_string,返回path指定的内容。如果输入的json字符串无效，那么返回NULL。 这个函数每次只能返回一个数据项。

```
0: jdbc:hive2://hadoop3:10000> select get_json_object('{"movie":"594","rate":"4","timeStamp":"978302268","uid":"1"}','$.movie');
```

![img](https://images2018.cnblogs.com/blog/1228818/201804/1228818-20180414111001588-234254636.png)

创建json表并将数据导入进去

```
0: jdbc:hive2://hadoop3:10000> create table json(data string);
No rows affected (0.983 seconds)
0: jdbc:hive2://hadoop3:10000> load data local inpath '/home/hadoop/json.txt' into table json;
No rows affected (1.046 seconds)
0: jdbc:hive2://hadoop3:10000> 
```

 ![img](https://images2018.cnblogs.com/blog/1228818/201804/1228818-20180414111449488-1854085883.png)

```
0: jdbc:hive2://hadoop3:10000> select 
. . . . . . . . . . . . . . .> get_json_object(data,'$.movie') as movie 
. . . . . . . . . . . . . . .> from json；
```

![img](https://images2018.cnblogs.com/blog/1228818/201804/1228818-20180414111805847-1824092036.png)

#### B.　json_tuple(jsonStr, k1, k2, ...)

参数为一组键k1，k2……和JSON字符串，返回值的元组。该方法比 `get_json_object` 高效，因为可以在一次调用中输入多个键

```
0: jdbc:hive2://hadoop3:10000> select 
. . . . . . . . . . . . . . .>   b.b_movie,
. . . . . . . . . . . . . . .>   b.b_rate,
. . . . . . . . . . . . . . .>   b.b_timeStamp,
. . . . . . . . . . . . . . .>   b.b_uid   
. . . . . . . . . . . . . . .> from json a 
. . . . . . . . . . . . . . .> lateral view json_tuple(a.data,'movie','rate','timeStamp','uid') b as b_movie,b_rate,b_timeStamp,b_uid;
```

![img](https://images2018.cnblogs.com/blog/1228818/201804/1228818-20180414113012552-2019392285.png)

### (3) Transform实现

Hive 的 TRANSFORM 关键字提供了在 SQL 中调用自写脚本的功能。适合实现 Hive 中没有的 功能又不想写 UDF 的情况

具体以一个实例讲解。

Json 数据： {"movie":"1193","rate":"5","timeStamp":"978300760","uid":"1"}

需求：把 timestamp 的值转换成日期编号

1、先加载 rating.json 文件到 hive 的一个原始表 rate_json

```
create table rate_json(line string) row format delimited;
load data local inpath '/home/hadoop/rating.json' into table rate_json;
```

2、创建 rate 这张表用来存储解析 json 出来的字段：

```
create table rate(movie int, rate int, unixtime int, userid int) row format delimited fields
terminated by '\t';
```

解析 json，得到结果之后存入 rate 表：

```
insert into table rate select
get_json_object(line,'$.movie') as moive,
get_json_object(line,'$.rate') as rate,
get_json_object(line,'$.timeStamp') as unixtime,
get_json_object(line,'$.uid') as userid
from rate_json;
```

3、使用 transform+python 的方式去转换 unixtime 为 weekday

先编辑一个 python 脚本文件

```
########python######代码
## vi weekday_mapper.py
#!/bin/python
import sys
import datetime
for line in sys.stdin:
 line = line.strip()
 movie,rate,unixtime,userid = line.split('\t')
 weekday = datetime.datetime.fromtimestamp(float(unixtime)).isoweekday()
 print '\t'.join([movie, rate, str(weekday),userid])
```

保存文件 然后，将文件加入 hive 的 classpath：

```
hive>add file /home/hadoop/weekday_mapper.py;
hive> insert into table lastjsontable select transform(movie,rate,unixtime,userid)
using 'python weekday_mapper.py' as(movie,rate,weekday,userid) from rate;
```

创建最后的用来存储调用 python 脚本解析出来的数据的表：lastjsontable

```
create table lastjsontable(movie int, rate int, weekday int, userid int) row format delimited
fields terminated by '\t';
```

最后查询看数据是否正确

```
select distinct(weekday) from lastjsontable;
```

## 四、特殊分隔符处理

补充：hive 读取数据的机制：

1、 首先用 InputFormat<默认是：org.apache.hadoop.mapred.TextInputFormat >的一个具体实 现类读入文件数据，返回一条一条的记录（可以是行，或者是你逻辑中的“行”）

2、 然后利用 SerDe<默认：org.apache.hadoop.hive.serde2.lazy.LazySimpleSerDe>的一个具体 实现类，对上面返回的一条一条的记录进行字段切割

Hive 对文件中字段的分隔符默认情况下只支持单字节分隔符，如果数据文件中的分隔符是多 字符的，如下所示：

01||huangbo

02||xuzheng

03||wangbaoqiang

### 1、使用RegexSerDe正则表达式解析

创建表

```
create table t_bi_reg(id string,name string)
row format serde 'org.apache.hadoop.hive.serde2.RegexSerDe'
with serdeproperties('input.regex'='(.*)\\|\\|(.*)','output.format.string'='%1$s %2$s')
stored as textfile;
```

![img](https://images2018.cnblogs.com/blog/1228818/201804/1228818-20180414113909764-1659213834.png)

导入数据并查询

```
0: jdbc:hive2://hadoop3:10000> load data local inpath '/home/hadoop/data.txt' into table t_bi_reg;
No rows affected (0.747 seconds)
0: jdbc:hive2://hadoop3:10000> select a.* from t_bi_reg a;
```

![img](https://images2018.cnblogs.com/blog/1228818/201804/1228818-20180414114100544-189365287.png)

### 2、通过自定义InputFormat处理特殊分隔符

