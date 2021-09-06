<p align="center">
    <img width="280px" src="image/konglong/m1.png" >
</p>

# Phoenix

## Phoenix是什么

Phoenix = HBase + SQL

社区公认的HBase上最合适的SQL层，支持毫秒级到秒级的OLTP和操作型分析查询

**Phoenix** 直接使用 **HBase API**、协同处理器与自定义过滤器，对于简单查询来说，其性能量级是毫秒，对于百万级别的行数来说，其性能量级是秒。 

## 集群搭建

### 前置环境

> Java、Hadoop、Hbase
>
> 示例环境：Java = 1.8, Hadoop=2.7.7, Hbase=2.1.9, phoenix=phoenix-hbase-2.1-5.1.2

### 安装

- 根据Hbase版本下载对应的phoenix包到HMaster节点

- 解压 tar -zxvf phoenix-hbase-2.1-5.1.2-bin.tar.gz -C /opt/***

- [可选] 创建软链 ln -s /opt/***/phoenix-hbase-2.1-5.1.2-bin  phoenix

- 把phoenix目录下的phoenix-server-hbase-2.1-5.1.2.jar复制到hbase每个regionserver的lib目录下

- 如果需要启用二级索引，需要修改Hbase regionserver的配置文件hbase-site.xml，并将hbase-site.xml复制到phoenix/bin目录下，还有hadoop的core-site.xml、hdfs-site.xml也复制到phoenix/bin目录下

  > <--! 建议zk只写最后一个端口，不然phoenix建索引时可能会出错-->
  >
  > <property>
  >    <name>hbase.zookeeper.quorum</name>
  >    <value>hadoop01,hadoop02,hadoop03:2181</value>
  > </property>
  >
  > <property>
  >   <name>hbase.regionserver.wal.codec</name>
  >   <value>org.apache.hadoop.hbase.regionserver.wal.IndexedWALEditCodec</value>
  > </property>
  >
  > <property>
  >   <name>hbase.region.server.rpc.scheduler.factory.class</name>
  >   <value>org.apache.hadoop.hbase.ipc.PhoenixRpcSchedulerFactory</value>
  >   <description>Factory to create the Phoenix RPC Scheduler that uses separate queues for index and metadata updates</description>
  > </property>
  >
  > <property>
  >   <name>hbase.rpc.controllerfactory.class</name>
  >   <value>org.apache.hadoop.hbase.ipc.controller.ServerRpcControllerFactory</value>
  >   <description>Factory to create the Phoenix RPC Scheduler that uses separate queues for index and metadata updates</description>
  > </property>

- **注**：如果phoenix版本低，还需配置HMaster的hbase-site.xml，配置参考网上

### 启动phoenix

- 【可选】将phoenix加入到classpath下

  > sudo vi /etc/profile
  >
  > export PHOENIX_HOME=/opt/hadoop/phoenix
  > export PATH=$PHOENIX_HOME/bin:$PATH

- phoenix/bin目录下执行 ./sqlline.py hadoop01,hadoop02,hadoop03:2181 后面为hbase的zk地址

- !table 可看到phoenix现存的system表

### 客户端安装

- DBeaver : 参考http://t.zoukankan.com/aixing-p-13327211.html，注意zk地址，连接成功。
- SQuirreL :参考https://blog.csdn.net/qq_40315068/article/details/86519497，略略略，没验证。

## Phoenix操作

### DDL

#### 查看所有表

```sql
0: jdbc:phoenix:mini1,mini2,mini3:2181> !tables
+------------+--------------+-------------+---------------+----------+------------+----------------------------+-----------------+--------------+----------+
| TABLE_CAT  | TABLE_SCHEM  | TABLE_NAME  |  TABLE_TYPE   | REMARKS  | TYPE_NAME  | SELF_REFERENCING_COL_NAME  | REF_GENERATION  | INDEX_STATE  | IMMUTABL |
+------------+--------------+-------------+---------------+----------+------------+----------------------------+-----------------+--------------+----------+
|            | SYSTEM       | CATALOG     | SYSTEM TABLE  |          |            |                            |                 |              | false    |
|            | SYSTEM       | FUNCTION    | SYSTEM TABLE  |          |            |                            |                 |              | false    |
|            | SYSTEM       | LOG         | SYSTEM TABLE  |          |            |                            |                 |              | true     |
|            | SYSTEM       | SEQUENCE    | SYSTEM TABLE  |          |            |                            |                 |              | false    |
|            | SYSTEM       | STATS       | SYSTEM TABLE  |          |            |                            |                 |              | false    |
+------------+--------------+-------------+---------------+----------+------------+----------------------------+-----------------+--------------+----------+
0: jdbc:phoenix:mini1,mini2,mini3:2181> 
```

#### 创建表

```sql
0: jdbc:phoenix:mini1,mini2,mini3:2181> CREATE TABLE IF NOT EXISTS us_population (
. . . . . . . . . . . . . . . . . . . >       state CHAR(2) NOT NULL,
. . . . . . . . . . . . . . . . . . . >       city VARCHAR NOT NULL,
. . . . . . . . . . . . . . . . . . . >       population BIGINT
. . . . . . . . . . . . . . . . . . . >       CONSTRAINT my_pk PRIMARY KEY (state, city));
No rows affected (1.844 seconds)
```

> [!Tip]

>  char类型必须添加长度限制
  varchar 可以不用长度限制
  主键映射到 HBase 中会成为 Rowkey. 如果有多个主键(联合主键), 会把多个主键的值拼成 rowkey
  在 Phoenix 中, 默认会把表名,字段名等自动转换成大写. 如果要使用小写, 需要把他们用双引号括起来.
  建表时注意数据类型

http://phoenix.apache.org/language/datatypes.html

#### 查看表结构

```sql
0: jdbc:phoenix:mini1,mini2,mini3:2181> !describe us_population
+------------+--------------+----------------+--------------+------------+------------+--------------+----------------+-----------------+------------------+
| TABLE_CAT  | TABLE_SCHEM  |   TABLE_NAME   | COLUMN_NAME  | DATA_TYPE  | TYPE_NAME  | COLUMN_SIZE  | BUFFER_LENGTH  | DECIMAL_DIGITS  | NUM_PREC_RADIX   |
+------------+--------------+----------------+--------------+------------+------------+--------------+----------------+-----------------+------------------+
|            |              | US_POPULATION  | STATE        | 1          | CHAR       | 2            | null           | null            | null             |
|            |              | US_POPULATION  | CITY         | 12         | VARCHAR    | null         | null           | null            | null             |
|            |              | US_POPULATION  | POPULATION   | -5         | BIGINT     | null         | null           | null            | null             |
+------------+--------------+----------------+--------------+------------+------------+--------------+----------------+-----------------+------------------+

```

#### 修改表

> Alters an existing table by adding or removing columns or updating table options. When a column is dropped from a table, the data in that column is deleted as well. PK columns may not be dropped, and only nullable PK columns may be added. For a view, the data is not affected when a column is dropped. Note that creating or dropping columns only affects subsequent queries and data modifications. Snapshot queries that are connected at an earlier timestamp will still use the prior schema that was in place when the data was written.
>
> Example:
>
> ALTER TABLE my_schema.my_table ADD d.dept_id char(10) VERSIONS=10
> ALTER TABLE my_table ADD dept_name char(50), parent_id char(15) null primary key
> ALTER TABLE my_table DROP COLUMN d.dept_id, parent_id;
> ALTER VIEW my_view DROP COLUMN new_col;
> ALTER TABLE my_table SET IMMUTABLE_ROWS=true,DISABLE_WAL=true

```sql
0: jdbc:phoenix:mini1,mini2,mini3:2181> ALTER TABLE us_population ADD dept_id char(10),parent_id char(15);
No rows affected (6.063 seconds)
0: jdbc:phoenix:mini1,mini2,mini3:2181> !describe us_population
+------------+--------------+----------------+--------------+------------+------------+--------------+----------------+-----------------+------------------+
| TABLE_CAT  | TABLE_SCHEM  |   TABLE_NAME   | COLUMN_NAME  | DATA_TYPE  | TYPE_NAME  | COLUMN_SIZE  | BUFFER_LENGTH  | DECIMAL_DIGITS  | NUM_PREC_RADIX   |
+------------+--------------+----------------+--------------+------------+------------+--------------+----------------+-----------------+------------------+
|            |              | US_POPULATION  | STATE        | 1          | CHAR       | 2            | null           | null            | null             |
|            |              | US_POPULATION  | CITY         | 12         | VARCHAR    | null         | null           | null            | null             |
|            |              | US_POPULATION  | POPULATION   | -5         | BIGINT     | null         | null           | null            | null             |
|            |              | US_POPULATION  | DEPT_ID      | 1          | CHAR       | 10           | null           | null            | null             |
|            |              | US_POPULATION  | PARENT_ID    | 1          | CHAR       | 15           | null           | null            | null             |
+------------+--------------+----------------+--------------+------------+------------+--------------+----------------+-----------------+------------------+

```

```sql
0: jdbc:phoenix:mini1,mini2,mini3:2181> ALTER TABLE us_population DROP COLUMN dept_id, parent_id ;
No rows affected (0.262 seconds)
0: jdbc:phoenix:mini1,mini2,mini3:2181> !describe us_population
+------------+--------------+----------------+--------------+------------+------------+--------------+----------------+-----------------+------------------+
| TABLE_CAT  | TABLE_SCHEM  |   TABLE_NAME   | COLUMN_NAME  | DATA_TYPE  | TYPE_NAME  | COLUMN_SIZE  | BUFFER_LENGTH  | DECIMAL_DIGITS  | NUM_PREC_RADIX   |
+------------+--------------+----------------+--------------+------------+------------+--------------+----------------+-----------------+------------------+
|            |              | US_POPULATION  | STATE        | 1          | CHAR       | 2            | null           | null            | null             |
|            |              | US_POPULATION  | CITY         | 12         | VARCHAR    | null         | null           | null            | null             |
|            |              | US_POPULATION  | POPULATION   | -5         | BIGINT     | null         | null           | null            | null             |
+------------+--------------+----------------+--------------+------------+------------+--------------+----------------+-----------------+------------------+

```

#### 删除表

```sql
0: jdbc:phoenix:mini1,mini2,mini3:2181> drop table us_population;
No rows affected (4.121 seconds)
0: jdbc:phoenix:mini1,mini2,mini3:2181> !table
+------------+--------------+-------------+---------------+----------+------------+----------------------------+-----------------+--------------+----------+
| TABLE_CAT  | TABLE_SCHEM  | TABLE_NAME  |  TABLE_TYPE   | REMARKS  | TYPE_NAME  | SELF_REFERENCING_COL_NAME  | REF_GENERATION  | INDEX_STATE  | IMMUTABL |
+------------+--------------+-------------+---------------+----------+------------+----------------------------+-----------------+--------------+----------+
|            | SYSTEM       | CATALOG     | SYSTEM TABLE  |          |            |                            |                 |              | false    |
|            | SYSTEM       | FUNCTION    | SYSTEM TABLE  |          |            |                            |                 |              | false    |
|            | SYSTEM       | LOG         | SYSTEM TABLE  |          |            |                            |                 |              | true     |
|            | SYSTEM       | SEQUENCE    | SYSTEM TABLE  |          |            |                            |                 |              | false    |
|            | SYSTEM       | STATS       | SYSTEM TABLE  |          |            |                            |                 |              | false    |
+------------+--------------+-------------+---------------+----------+------------+----------------------------+-----------------+--------------+----------+

```

#### 退出命令行

```
!quit
```

### DML

#### 插入记录

```sql
0: jdbc:phoenix:mini1,mini2,mini3:2181> upsert into us_population values('NY','NewYork',8143197);
1 row affected (0.035 seconds)
0: jdbc:phoenix:mini1,mini2,mini3:2181> upsert into us_population values('CA','Los Angeles',3844829);
1 row affected (0.005 seconds)
0: jdbc:phoenix:mini1,mini2,mini3:2181> upsert into us_population values('IL','Chicago',2842518);
1 row affected (0.004 seconds)

```

> 说明: upset可以看成是update和insert的结合体.

#### 查询记录

```sql
0: jdbc:phoenix:mini1,mini2,mini3:2181> select * from US_POPULATION;
+--------+--------------+-------------+
| STATE  |     CITY     | POPULATION  |
+--------+--------------+-------------+
| CA     | Los Angeles  | 3844829     |
| IL     | Chicago      | 2842518     |
| NY     | NewYork      | 8143197     |
+--------+--------------+-------------+
3 rows selected (0.049 seconds)
```

> 复杂查询语法请参看官网说明

#### 删除记录

```sql
0: jdbc:phoenix:mini1,mini2,mini3:2181> delete from us_population where state='NY';
1 row affected (0.014 seconds)
0: jdbc:phoenix:mini1,mini2,mini3:2181> select * from US_POPULATION;
+--------+--------------+-------------+
| STATE  |     CITY     | POPULATION  |
+--------+--------------+-------------+
| CA     | Los Angeles  | 3844829     |
| IL     | Chicago      | 2842518     |
+--------+--------------+-------------+
2 rows selected (0.214 seconds)
```

#### 修改记录

批量修改

> Inserts if not present and updates otherwise rows in the table based on the results of running another query. The values are set based on their matching position between the source and target tables. The list of columns is optional and if not present will map to the column in the order they are declared in the schema. If auto commit is on, and both a) the target table matches the source table, and b) the select performs no aggregation, then the population of the target table will be done completely on the server-side (with constraint violations logged, but otherwise ignored). Otherwise, data is buffered on the client and, if auto commit is on, committed in row batches as specified by the UpsertBatchSize connection property (or the phoenix.mutate.upsertBatchSize HBase config property which defaults to 10000 rows)
>
> Example:
>
> UPSERT INTO test.targetTable(col1, col2) SELECT col3, col4 FROM test.sourceTable WHERE col5 < 100
> UPSERT INTO foo SELECT * FROM bar;

```sql
UPSERT INTO US_POPULATION(STATE, CITY,POPULATION) SELECT 'CA','Los Angeles_update',3844827 FROM US_POPULATION WHERE POPULATION > 3844828;

```

单值修改

> Inserts if not present and updates otherwise the value in the table. The list of columns is optional and if not present, the values will map to the column in the order they are declared in the schema. The values must evaluate to constants.
>
> Use the ON DUPLICATE KEY clause (available in Phoenix 4.9) if you need the UPSERT to be atomic. Performance will be slower in this case as the row needs to be read on the server side when the commit is done. Use IGNORE if you do not want the UPSERT performed if the row already exists. Otherwise, with UPDATE, the expression will be evaluated and the result used to set the column, for example to perform an atomic increment. An UPSERT to the same row in the same commit batch will be processed in the order of execution.
>
> Example:
>
> UPSERT INTO TEST VALUES(‘foo’,‘bar’,3);
> UPSERT INTO TEST(NAME,ID) VALUES(‘foo’,123);
> UPSERT INTO TEST(ID, COUNTER) VALUES(123, 0) ON DUPLICATE KEY UPDATE COUNTER = COUNTER + 1;
> UPSERT INTO TEST(ID, MY_COL) VALUES(123, 0) ON DUPLICATE KEY IGNORE;

```sql
0: jdbc:phoenix:mini1,mini2,mini3:2181> select * from US_POPULATION;
+--------+---------------------+-------------+
| STATE  |        CITY         | POPULATION  |
+--------+---------------------+-------------+
| A      | Rose                | 3844822     |
| CA     | Los Angeles         | 3844829     |
| CA     | Los Angeles_update  | 3844827     |
| IL     | Chicago             | 2842518     |
+--------+---------------------+-------------+
4 rows selected (0.039 seconds)
0: jdbc:phoenix:mini1,mini2,mini3:2181> UPSERT INTO US_POPULATION(STATE, CITY,POPULATION) VALUES('A','Rose',3844821);
1 row affected (0.005 seconds)
0: jdbc:phoenix:mini1,mini2,mini3:2181> select * from US_POPULATION;
+--------+---------------------+-------------+
| STATE  |        CITY         | POPULATION  |
+--------+---------------------+-------------+
| A      | Rose                | 3844821     |
| CA     | Los Angeles         | 3844829     |
| CA     | Los Angeles_update  | 3844827     |
| IL     | Chicago             | 2842518     |
+--------+---------------------+-------------+
4 rows selected (0.032 seconds)
```

> 如上，如果主键记录不存在会直接insert，如果存在会update

### Phoenix表映射

> Phoenix 和 HBase 映射关系：默认情况下, 直接在 HBase 中创建的表通过 Phoenix 是查不到的；
>
> 如果要在 Phoenix 中操作直接在 HBase 中创建的表，则需要在 Phoenix 中进行表的映射。
>
> 映射方式有两种： 1. 视图映射 2. 表映射

准备HBase表来测试

```sql
hbase(main):001:0>  create 'test', 'name', 'company'
0 row(s) in 1.5560 seconds

=> Hbase::Table - test

hbase(main):008:0> describe 'test'
Table test is ENABLED                                                                                                                                       
test                                                                                                                                                        
COLUMN FAMILIES DESCRIPTION                                                                                                                                 
{NAME => 'company', BLOOMFILTER => 'ROW', VERSIONS => '1', IN_MEMORY => 'false', KEEP_DELETED_CELLS => 'FALSE', DATA_BLOCK_ENCODING => 'NONE', TTL => 'FOREV
ER', COMPRESSION => 'NONE', MIN_VERSIONS => '0', BLOCKCACHE => 'true', BLOCKSIZE => '65536', REPLICATION_SCOPE => '0'}                                      
{NAME => 'name', BLOOMFILTER => 'ROW', VERSIONS => '1', IN_MEMORY => 'false', KEEP_DELETED_CELLS => 'FALSE', DATA_BLOCK_ENCODING => 'NONE', TTL => 'FOREVER'
, COMPRESSION => 'NONE', MIN_VERSIONS => '0', BLOCKCACHE => 'true', BLOCKSIZE => '65536', REPLICATION_SCOPE => '0'}                                         
2 row(s) in 0.1270 seconds
```

此时在Phoenix中看不到该表

```sql
0: jdbc:phoenix:mini1,mini2,mini3:2181> !table
+------------+--------------+----------------+---------------+----------+------------+----------------------------+-----------------+--------------+-------+
| TABLE_CAT  | TABLE_SCHEM  |   TABLE_NAME   |  TABLE_TYPE   | REMARKS  | TYPE_NAME  | SELF_REFERENCING_COL_NAME  | REF_GENERATION  | INDEX_STATE  | IMMUT |
+------------+--------------+----------------+---------------+----------+------------+----------------------------+-----------------+--------------+-------+
|            | SYSTEM       | CATALOG        | SYSTEM TABLE  |          |            |                            |                 |              | false |
|            | SYSTEM       | FUNCTION       | SYSTEM TABLE  |          |            |                            |                 |              | false |
|            | SYSTEM       | LOG            | SYSTEM TABLE  |          |            |                            |                 |              | true  |
|            | SYSTEM       | SEQUENCE       | SYSTEM TABLE  |          |            |                            |                 |              | false |
|            | SYSTEM       | STATS          | SYSTEM TABLE  |          |            |                            |                 |              | false |
|            |              | US_POPULATION  | TABLE         |          |            |                            |                 |              | false |
+------------+--------------+----------------+---------------+----------+------------+----------------------------+-----------------+--------------+-------+

```

#### 视图映射

> Phoenix 创建的视图是只读的, 所以只能用来查询, 无法通过视图对数据进行修改等操作.

```sql
0: jdbc:phoenix:mini1,mini2,mini3:2181> create view "test"(empid varchar primary key,"name"."firstname" varchar,"name"."lastname" varchar,"company"."name" varchar,"company"."address" varchar);
No rows affected (6.225 seconds)
0: jdbc:phoenix:mini1,mini2,mini3:2181> !table
+------------+--------------+----------------+---------------+----------+------------+----------------------------+-----------------+--------------+-------+
| TABLE_CAT  | TABLE_SCHEM  |   TABLE_NAME   |  TABLE_TYPE   | REMARKS  | TYPE_NAME  | SELF_REFERENCING_COL_NAME  | REF_GENERATION  | INDEX_STATE  | IMMUT |
+------------+--------------+----------------+---------------+----------+------------+----------------------------+-----------------+--------------+-------+
|            | SYSTEM       | CATALOG        | SYSTEM TABLE  |          |            |                            |                 |              | false |
|            | SYSTEM       | FUNCTION       | SYSTEM TABLE  |          |            |                            |                 |              | false |
|            | SYSTEM       | LOG            | SYSTEM TABLE  |          |            |                            |                 |              | true  |
|            | SYSTEM       | SEQUENCE       | SYSTEM TABLE  |          |            |                            |                 |              | false |
|            | SYSTEM       | STATS          | SYSTEM TABLE  |          |            |                            |                 |              | false |
|            |              | US_POPULATION  | TABLE         |          |            |                            |                 |              | false |
|            |              | test           | VIEW          |          |            |                            |                 |              | false |
+------------+--------------+----------------+---------------+----------+------------+----------------------------+-----------------+--------------+-------+

```

之后就可以在phoenix中查询了，但是注意表名如果是小写要加引号

```sql
0: jdbc:phoenix:mini1,mini2,mini3:2181> select * from "test";
+--------+------------+-----------+-------+----------+
| EMPID  | firstname  | lastname  | name  | address  |
+--------+------------+-----------+-------+----------+
+--------+------------+-----------+-------+----------+
No rows selected (0.126 seconds)


```

#### 表映射

> 表映射可以更改Hbase中表数据

- HBase中表不存在时
       Hbase中表不存在时，可以直接使用 create table 指令创建需要的表,系统将会自动在 Phoenix 和 HBase 中创建 person_infomation 的表，并会根据指令内的参数对表结构进行初始化。

- HBase中表存在时
      HBase 中已经存在表时，可以以类似创建视图的方式创建关联表，只需要将create view 改为 create table 即可

在 HBase 中创建表：

```sql
hbase(main):010:0>  create 'test1', 'name', 'company'
0 row(s) in 1.3540 seconds

=> Hbase::Table - test1
hbase(main):011:0> list
TABLE                                                                                     ..                                                                                       US_POPULATION                                                                             test                                                                                     test1                                                                                                                                                       
9 row(s) in 0.0160 seconds

=> ["SYSTEM.CATALOG", "SYSTEM.FUNCTION", "SYSTEM.LOG", "SYSTEM.MUTEX", "SYSTEM.SEQUENCE", "SYSTEM.STATS", "US_POPULATION", "test", "test1"]
```

在Phoenix中进行表映射：

```sql
0: jdbc:phoenix:mini1,mini2,mini3:2181> create table "test1"(empid varchar primary key,"name"."firstname" varchar,"name"."lastname" varchar,"company"."name" varchar,"company"."address" varchar) column_encoded_bytes=0;
No rows affected (6.22 seconds)
0: jdbc:phoenix:mini1,mini2,mini3:2181> !table
+------------+--------------+----------------+---------------+----------+------------+----------------------------+-----------------+--------------+-------+
| TABLE_CAT  | TABLE_SCHEM  |   TABLE_NAME   |  TABLE_TYPE   | REMARKS  | TYPE_NAME  | SELF_REFERENCING_COL_NAME  | REF_GENERATION  | INDEX_STATE  | IMMUT |
+------------+--------------+----------------+---------------+----------+------------+----------------------------+-----------------+--------------+-------+
|            | SYSTEM       | CATALOG        | SYSTEM TABLE  |          |            |                            |                 |              | false |
|            | SYSTEM       | FUNCTION       | SYSTEM TABLE  |          |            |                            |                 |              | false |
|            | SYSTEM       | LOG            | SYSTEM TABLE  |          |            |                            |                 |              | true  |
|            | SYSTEM       | SEQUENCE       | SYSTEM TABLE  |          |            |                            |                 |              | false |
|            | SYSTEM       | STATS          | SYSTEM TABLE  |          |            |                            |                 |              | false |
|            |              | US_POPULATION  | TABLE         |          |            |                            |                 |              | false |
|            |              | test1          | TABLE         |          |            |                            |                 |              | false |
|            |              | test           | VIEW          |          |            |                            |                 |              | false |
+------------+--------------+----------------+---------------+----------+------------+----------------------------+-----------------+--------------+-------+
```

> Phoenix 区分大小写，切默认情况下会将小写转成大写，所以表名、列簇、列名需要用双引号。
> Phoenix 4.10 版本之后，在创建表映射时需要将 COLUMN_ENCODED_BYTES 置为 0。
> 删除映射表，会同时删除原有 HBase 表。所以如果只做查询操作，建议做视图映射。

**视图映射和表映射总结**
相比于直接创建映射表，视图的查询效率会低， 原因是：创建映射表的时候，Phoenix 会在表中创建一些空的键值对，这些空键值对的存在可以用来提高查询效率。

使用create table创建的关联表，如果对表进行了修改，源数据也会改变，同时如果关联表被删除，源表也会被删除。但是视图就不会，如果删除视图，源数据不会发生改变

### phoenix函数

http://phoenix.apache.org/language/functions.html

网络上没有很好的使用用例，可去jira上看issue

https://issues.apache.org/jira/browse/PHOENIX-1290?jql=project%20%3D%20PHOENIX%20AND%20text%20~%20%22first_value%22

## phoenix创建HBase二级索引

### 配置二级索引

**需要先给 HBase 配置支持创建二级索引，如上或如下**

添加如下配置到 HBase 的 Hregionerver 节点的 hbase-site.xml，后重启hbase

```xml
<!-- phoenix regionserver 配置参数 -->
<property>
    <name>hbase.regionserver.wal.codec</name>
    <value>org.apache.hadoop.hbase.regionserver.wal.IndexedWALEditCodec</value>
</property>

<property>
    <name>hbase.region.server.rpc.scheduler.factory.class</name>
    <value>org.apache.hadoop.hbase.ipc.PhoenixRpcSchedulerFactory</value>
<description>Factory to create the Phoenix RPC Scheduler that uses separate queues for index and metadata updates</description>
</property>

<property>
    <name>hbase.rpc.controllerfactory.class</name>
    <value>org.apache.hadoop.hbase.ipc.controller.ServerRpcControllerFactory</value>
    <description>Factory to create the Phoenix RPC Scheduler that uses separate queues for index and metadata updates</description>
</property>

```

### 测试二级索引

准备数据

```sql
0: jdbc:phoenix:mini1,mini2,mini3:2181> create table user_1(id varchar primary key, name varchar, addr varchar);
No rows affected (2.59 seconds)
0: jdbc:phoenix:mini1,mini2,mini3:2181> upsert into user_1 values ('1', 'zs', 'beijing');
1 row affected (0.078 seconds)
0: jdbc:phoenix:mini1,mini2,mini3:2181> upsert into user_1 values ('2', 'lisi', 'shanghai');
1 row affected (0.006 seconds)
0: jdbc:phoenix:mini1,mini2,mini3:2181> upsert into user_1 values ('3', 'ww', 'sz');
1 row affected (0.006 seconds)
0: jdbc:phoenix:mini1,mini2,mini3:2181> select * from user_1;
+-----+-------+-----------+
| ID  | NAME  |   ADDR    |
+-----+-------+-----------+
| 1   | zs    | beijing   |
| 2   | lisi  | shanghai  |
| 3   | ww    | sz        |
+-----+-------+-----------+
3 rows selected (0.074 seconds)
```

如下rowkey查询时是使用了索引的，仍然是使用`explain`关键字查看

```sql
0: jdbc:phoenix:mini1,mini2,mini3:2181> select * from user_1 where ID = '1';
+-----+-------+----------+
| ID  | NAME  |   ADDR   |
+-----+-------+----------+
| 1   | zs    | beijing  |
+-----+-------+----------+
1 row selected (0.036 seconds)
0: jdbc:phoenix:mini1,mini2,mini3:2181> explain select * from user_1 where ID = '1';
+-----------------------------------------------------------------------------------------------+-----------------+----------------+--------------+
|                                             PLAN                                              | EST_BYTES_READ  | EST_ROWS_READ  | EST_INFO_TS  |
+-----------------------------------------------------------------------------------------------+-----------------+----------------+--------------+
| CLIENT 1-CHUNK 1 ROWS 205 BYTES PARALLEL 1-WAY ROUND ROBIN POINT LOOKUP ON 1 KEY OVER USER_1  | 205             | 1              | 0            |
+-----------------------------------------------------------------------------------------------+-----------------+----------------+--------------+
1 row selected (0.036 seconds)
```

> 其他字段是不支持索引的目前
>
> 如下full scan

```sql
0: jdbc:phoenix:mini1,mini2,mini3:2181> explain select * from user_1 where NAME = 'zs';
+------------------------------------------------------------------+-----------------+----------------+--------------+
|                               PLAN                               | EST_BYTES_READ  | EST_ROWS_READ  | EST_INFO_TS  |
+------------------------------------------------------------------+-----------------+----------------+--------------+
| CLIENT 1-CHUNK PARALLEL 1-WAY ROUND ROBIN FULL SCAN OVER USER_1  | null            | null           | null         |
|     SERVER FILTER BY NAME = 'zs'                                 | null            | null           | null         |
+------------------------------------------------------------------+-----------------+----------------+--------------+
2 rows selected (0.097 seconds)
```

给 name 字段添加索引

```sql
0: jdbc:phoenix:mini1,mini2,mini3:2181> create index idx_user_1 on user_1(name);
3 rows affected (6.584 seconds)
0: jdbc:phoenix:mini1,mini2,mini3:2181> explain select * from user_1 where NAME = 'zs';
+------------------------------------------------------------------+-----------------+----------------+--------------+
|                               PLAN                               | EST_BYTES_READ  | EST_ROWS_READ  | EST_INFO_TS  |
+------------------------------------------------------------------+-----------------+----------------+--------------+
| CLIENT 1-CHUNK PARALLEL 1-WAY ROUND ROBIN FULL SCAN OVER USER_1  | null            | null           | null         |
|     SERVER FILTER BY NAME = 'zs'                                 | null            | null           | null         |
+------------------------------------------------------------------+-----------------+----------------+--------------+
2 rows selected (0.094 seconds)
```

### Phoenix 索引分类

#### 全局索引

> global index 全局索引是默认的索引格式。

- 适用于多读少写的业务场景。写数据的时候会消耗大量开销，因为索引表也要更新，而索引表是分布在不同的数据节点上的，跨节点的数据传输带来了较大的性能消耗。


- 在读数据的时候 Phoenix 会选择索引表来降低查询消耗的时间。


- 如果想查询的字段不是索引字段的话索引表不会被使用，也就是说不会带来查询速度的提升。

创建全局索引方法

```sql
CREATE INDEX my_index ON my_table (my_col)

0: jdbc:phoenix:mini1,mini2,mini3:2181> CREATE INDEX my_index_name on user_1(name);
3 rows affected (6.317 seconds)
```

创建全局索引, 也支持查询其他字段

```sql
CREATE INDEX my_index ON my_table (v1) INCLUDE (v2)

SELECT v2 FROM my_table WHERE v1 = 'foo'

0: jdbc:phoenix:mini1,mini2,mini3:2181> CREATE INDEX my_index_name on user_1(name) INCLUDE(ADDR);
3 rows affected (6.293 seconds)
0: jdbc:phoenix:mini1,mini2,mini3:2181> explain select * from user_1 where ADDR = 'beijing';
+-------------------------------------------------------------------------+-----------------+----------------+--------------+
|                                  PLAN                                   | EST_BYTES_READ  | EST_ROWS_READ  | EST_INFO_TS  |
+-------------------------------------------------------------------------+-----------------+----------------+--------------+
| CLIENT 1-CHUNK PARALLEL 1-WAY ROUND ROBIN FULL SCAN OVER MY_INDEX_NAME  | null            | null           | null         |
|     SERVER FILTER BY "ADDR" = 'beijing'                                 | null            | null           | null         |
+-------------------------------------------------------------------------+-----------------+----------------+--------------+
2 rows selected (0.069 seconds)
```

这里的意思是会在name字段的索引MY_INDEX_NAME的基础上full scan，查询会加快

可去Hbase中观察索引表的结构，就理解了。

#### 局部索引

- local index 适用于写操作频繁的场景。索引数据和数据表的数据是存放在相同的服务器中的，避免了在写操作的时候往不同服务器的索引表中写索引带来的额外开销。
- 查询的字段不是索引字段索引表也会被使用，这会带来查询速度的提升。

创建局部索引的方法(相比全局索引多了一个关键字 local):

```sql
CREATE LOCAL INDEX my_index ON my_table (my_col)

0: jdbc:phoenix:mini1,mini2,mini3:2181> CREATE LOCAL INDEX my_local_index_name on user_1(name);
3 rows affected (6.309 seconds)
```

> 注意，建立local索引时，hbase-site.xml配置文件的zk信息不能加2181，否则会报错，如上。

### Local index 和 Global index区别

- Local index 由于是数据与索引在同一服务器上，所以要查询的数据在哪台服务器的哪个region是无法定位的，只能先找到region然后再利用索引。


- Global index 是一种分布式索引，可以直接利用索引定位服务器和region，速度更快，但是由于分布式的原因，数据一旦出现新增变化，分布式的索引要进行跨服务的同步操作，带来大量的通信消耗。所以在写操作频繁的字段上不适合建立Global index。

### 删除索引

```sql
DROP INDEX my_index ON my_table
 
0: jdbc:phoenix:mini1,mini2,mini3:2181> DROP INDEX my_index_name ON user_1;
No rows affected (4.003 seconds)
```

### 异步创建索引

当表数据量过大的时候，创建索引会报错，可以修改服务器端的 `hbase.rpc.timeout`，默认是1分钟，可以自定义时间。也可以异步创建索引，通过在语句后面添加`async` 关键字。

需要注意的是：

1. 异步创建索引只支持全局索引
2. 执行async语句只是第一步，还需要通过执行jar包来保证索引真正的建立

#### 1. 为什么只支持全局索引？

首先是本地索引和全局索引的一些概念和区别

- 本地索引

  - 适合写多读少的情况
  - 索引数据直接写在原表里，**不会新建一张表**。在 `phoenix-sqlline` 里执行 `!tables` 的确会发现创建的本地索引表，但是那个只是一个映射，并不是单独存在的。由于索引数据直接写在表里，所以原表的数据量=原始数据+索引数据。
  - 本地索引rowkey的设计规则: 原数据region的start key+"\x00"+二级索引字段1+"\x00"+二级索引字段2(复合索引)…+"\x00"+原rowkey。
  - 索引数据和真实数据存放在同一台机器上，减少了网络传输的开销。同理，创建索引后的rowkey的最开始的部分是 *原数据region的start key*，这样在通过二级索引定位到数据后，可以在当前的region中直接找到数据，减少网络开销。减少网络开销，也意味着写入的速度会变快。但是多了一步通过rowkey查找数据的过程，所以读的过程就不如直接读取列族里的数据的速度快。

- 全局索引

  - 适合读多写少的情况

  - 索引数据会单独存在一张表里。

  - 全局索引必须是查询语句中所有列都包含在全局索引中，它才会生效。

    > Select * 不会命中索引
    > select 具体的字段 from table where col ...
    > col 必须是第一个主键或者是索引里包含的字段才会命中索引
    > 如果索引表包含 a、b 三个字段，where 里有 a 和 c 两个字段，那么也不会走索引，因为c不在索引里，发现索引走不通，只能走全表

  - 为了命中索引，要把需要查询的字段通过 include 关键字来一起写入索引表里，也就是覆盖索引。

  - 写入数据的同时需要往索引表同步写数据，而索引表是分布在不同的数据节点上的，跨节点的数据传输带来了较大的性能消耗，所以写慢；但是查询的时候，如果命中了索引表，那就直接把数据带出来了，读会快。

综上，本地索引不是表，全局索引才是表，而async是针对表的一种方式，所以只能作用于全局索引

#### 2. 如何执行async

1. 首先是需要创建一个全局索引，同时使用 async

```
create index XXX on database.tablename(col1, col2) include(col3, col4) async
```

此时去看这个表，会发现 `index_state` 字段的值是 building，说明索引表还没创建好，这是因为 async 关键字会初始化一个mr作业，只是把创建索引的数据文件准备好，还没有正式开始

1. 执行mr作业

```
hbase org.apache.phoenix.mapreduce.index.IndexTool \
--schema 库名 --data-table 表名 --index-table 索引表名 \
--output-path hdfs路径指向一个文件件即可
```

> 库名、表名、索引表名尽量都不要小写

这个命令执行后可能会报错，遇到 org.apache.phoenix.mapreduce.index.IndexTool 依赖的jar没法加载，那就可以换一个方式执行

```
java -cp ./本地文件夹路径:/data1/cloudera/parcels/PHOENIX/lib/phoenix/phoenix-5.0.0-cdh6.2.0-client.jar  org.apache.phoenix.mapreduce.index.IndexTool   --schema 库名 --data-table 表名 --index-table 索引表名     --output-path hdfs路径指向一个文件即可
```

> 本地文件夹里需要包含 hbase yarn hdfs 的配置文件

如果遇到 `java.io.IOException: Can't get Master Kerberos principal for use as renewer` 说明缺少yarn的配置文件

如果遇到 `org.apache.hadoop.security.AccessControlException: Permission denied: user=phoenix, access=WRITE, inode="/user":hdfs:supergroup:drwxr-xr-x` 需要在 `hbase-site.xml` 文件里添加 `hbase.fs.tmp.dir` 配置项，值是hdfs上一个有读写权限的目录路径。
原因：从 org.apache.phoenix.mapreduce.index.IndexTool 开始追代码，会找到 org.apache.hadoop.hbase.mapreduce.HFileOutputFormat2，在配置mr作业的时候，`configurePartitioner()` 方法里 `String hbaseTmpFsDir = conf.get("hbase.fs.tmp.dir", HConstants.DEFAULT_TEMPORARY_HDFS_DIRECTORY);` 会去读取配置文件里的这个值，默认是 `"/user/" + System.getProperty("user.name") + "/hbase-staging"`

#### 3. 附

1. 查询执行计划，判断是否命中索引表

|              内容               |                             含义                             |
| :-----------------------------: | :----------------------------------------------------------: |
|             CLIENT              | 表明操作在客户端执行还是服务端执行，客户端尽量返回少的数据。若为 SERVER 表示在服务端执行。 |
|      FILTER BY expression       |                  返回和过滤条件匹配的结果。                  |
|    FULL SCAN OVER tableName     |                   表明全表扫描某张业务表。                   |
| RANGE SCAN OVER tableName [ … ] |   表明代表范围扫描某张表，括号内代表 rowkey 的开始和结束。   |
|           ROUND ROBIN           |  无 ORDER BY 操作时, ROUND ROBIN 代表最大化客户端的并行化。  |
|             x-CHUNK             |                     执行此操作的线程数。                     |
|         PARALLEL x-WAY          |                   表明合并多少并行的扫描。                   |
|         EST_BYTES_READ          |                执行查询时预计扫描的总字节数。                |
|          EST_ROWS_READ          |                  执行查询时预计扫描多少行。                  |
|           EST_INFO_TS           |                  收集查询信息的 epoch time                   |

1. 在创建索引的过程中，发现了一个可能是版本bug的地方，已提官网issue，链接如下

[官网issue地址](https://issues.apache.org/jira/projects/PHOENIX/issues/PHOENIX-6215?filter=updatedrecently)

问题：如果在创建本地索引时，有一个字段设置了default value，在生成的索引表里就只会显示默认值，不管是什么类型；如果这个类型是tinyint的话，还可能会造成之后主键的数据，原表的数据是对的，但是索引表是错的，如果命中了索引表，那么就返回的是错误的数据。

## JAVA API

**POM.xml** ,参考，包含了java和spark连接phoenix

```xml
        <!-- https://mvnrepository.com/artifact/org.apache.spark/spark-core -->
        <dependency>
            <groupId>org.apache.spark</groupId>
            <artifactId>spark-core_2.11</artifactId>
            <version>2.4.5</version>
        </dependency>
        <!-- https://mvnrepository.com/artifact/org.apache.spark/spark-sql -->
        <dependency>
            <groupId>org.apache.spark</groupId>
            <artifactId>spark-sql_2.11</artifactId>
            <version>2.4.5</version>
<!--            <scope>provided</scope>-->
        </dependency>

        <!-- https://mvnrepository.com/artifact/org.apache.phoenix/phoenix-spark -->
        <dependency>
            <groupId>org.apache.phoenix</groupId>
            <artifactId>phoenix-spark</artifactId>
            <version>5.0.0-HBase-2.0</version>
        </dependency>

        <dependency>
            <groupId>org.apache.phoenix</groupId>
            <artifactId>phoenix-hbase-compat-2.1.6</artifactId>
            <version>5.1.2</version>
            <scope>runtime</scope>
            <optional>true</optional>
        </dependency>

        <dependency>
            <groupId>org.apache.hadoop</groupId>
            <artifactId>hadoop-common</artifactId>
            <version>2.7.7</version>
        </dependency>

        <dependency>
            <groupId>org.apache.hbase</groupId>
            <artifactId>hbase-client</artifactId>
            <version>2.1.9</version>
        </dependency>

        <dependency>
            <groupId>org.apache.phoenix</groupId>
            <artifactId>phoenix-core</artifactId>
            <version>5.1.2</version>
        </dependency>
```

**例子1**

```java
import java.sql.Connection;
import java.sql.DriverManager;
import java.sql.ResultSet;
import java.sql.Statement;
 
public class Phoenix_Test {
    /**
     * 使用phoenix提供的api操作hbase读取数据
     */
    public static void main(String[] args) throws Throwable {
        try {
            // 下面的驱动为Phoenix老版本使用2.11使用，对应hbase0.94+
            // Class.forName("com.salesforce.phoenix.jdbc.PhoenixDriver");
 
            // phoenix4.3用下面的驱动对应hbase0.98+
            Class.forName("org.apache.phoenix.jdbc.PhoenixDriver");
        } catch (Exception e) {
            e.printStackTrace();
        }
        // 这里配置zookeeper的地址，可单个，也可多个。可以是域名或者ip
        String url = "jdbc:phoenix:node5,node6,node7";
        // String url =
        // "jdbc:phoenix:41.byzoro.com,42.byzoro.com,43.byzoro.com:2181";
        Connection conn = DriverManager.getConnection(url);
        Statement statement = conn.createStatement();
        String sql = "select count(1) as num from WEB_STAT";
        long time = System.currentTimeMillis();
        ResultSet rs = statement.executeQuery(sql);
        while (rs.next()) {
            int count = rs.getInt("num");
            System.out.println("row count is " + count);
        }
        long timeUsed = System.currentTimeMillis() - time;
        System.out.println("time " + timeUsed + "mm");
        // 关闭连接
        rs.close();
        statement.close();
        conn.close();
    }
}
```

**例子2**

```java
import java.sql.Connection;
import java.sql.DriverManager;
import java.sql.ResultSet;
import java.sql.Statement;
 
public class Phoenix_Test2 {
    /**
     * 使用phoenix提供的api操作hbase中读取数据
     */
    public static void main(String[] args) throws Throwable {
        try {
            //下面的驱动为Phoenix老版本使用2.11使用，对应hbase0.94+
            //Class.forName("com.salesforce.phoenix.jdbc.PhoenixDriver");
            //phoenix4.3用下面的驱动对应hbase0.98+
            Class.forName("org.apache.phoenix.jdbc.PhoenixDriver");
        } catch (Exception e) {
            e.printStackTrace();
        }
        //这里配置zk的地址，可单个，也可多个。可以是域名或者ip
        String url = "jdbc:phoenix:node5,node6,node7";
        Connection conn = DriverManager.getConnection(url);
        Statement statement = conn.createStatement();
        String sql = "select *  from web_stat where core = 1";
        long time = System.currentTimeMillis();
        ResultSet rs = statement.executeQuery(sql);
        while (rs.next()) {
            //获取core字段值
            int core = rs.getInt("core");
            //获取core字段值
            String host = rs.getString("host");
            //获取domain字段值
            String domain = rs.getString("domain");
            //获取feature字段值
            String feature = rs.getString("feature");
            //获取date字段值,数据库中字段为Date类型，这里代码会自动转化为string类型
            String date = rs.getString("date");
            //获取db字段值
            String db = rs.getString("db");
            System.out.println("host:"+host+"\tdomain:"+domain+"\tfeature:"+feature+"\tdate:"+date+"\tcore:" + core+"\tdb:"+db);
        }
        long timeUsed = System.currentTimeMillis() - time;
        System.out.println("time " + timeUsed + "mm");
        //关闭连接
        rs.close();
        statement.close();
        conn.close();
    }
}
```

**spark例子**

```java
SparkSession spark = SparkSession.builder().appName("test").master("local[*]").getOrCreate();

Dataset<Row> df = spark.read().format("org.apache.phoenix.spark")
       .option("zkUrl", "hadoop01,hadoop02,hadoop03:2181")
       .option("table", "\"test\"") // 注意大小写
       .load();

df.show();

df.write().format("org.apache.phoenix.spark")
       .mode(SaveMode.Overwrite)
       .option("zkUrl",  "hadoop01,hadoop02,hadoop03:2181")
       .option("table", "test2") // 注意大小写
       .save();
```

其他例子参考网络

## Phoenix架构和原理

[Phoenix(HBase SQL)核心功能原理及应用场景介绍-阿里云开发者社区 (aliyun.com)](https://developer.aliyun.com/article/703818)

## 注意事项

