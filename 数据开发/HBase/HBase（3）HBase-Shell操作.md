
<p align="center">
    <img width="280px" src="image/konglong/m2.png" >
</p>

# HBase（3）HBase-Shell操作

## HBase命令行

在你安装的随意台服务器节点上，执行命令：hbase shell，会进入到你的 hbase shell 客 户端

```
[hadoop@hadoop1 ~]$ hbase shell
SLF4J: Class path contains multiple SLF4J bindings.
SLF4J: Found binding in [jar:file:/home/hadoop/apps/hbase-1.2.6/lib/slf4j-log4j12-1.7.5.jar!/org/slf4j/impl/StaticLoggerBinder.class]
SLF4J: Found binding in [jar:file:/home/hadoop/apps/hadoop-2.7.5/share/hadoop/common/lib/slf4j-log4j12-1.7.10.jar!/org/slf4j/impl/StaticLoggerBinder.class]
SLF4J: See http://www.slf4j.org/codes.html#multiple_bindings for an explanation.
SLF4J: Actual binding is of type [org.slf4j.impl.Log4jLoggerFactory]
HBase Shell; enter 'help<RETURN>' for list of supported commands.
Type "exit<RETURN>" to leave the HBase Shell
Version 1.2.6, rUnknown, Mon May 29 02:25:32 CDT 2017

hbase(main):001:0> 
```

 说明，先看一下提示。其实是不是有一句很重要的话：

```
HBase Shell; enter 'help<RETURN>' for list of supported commands.
Type "exit<RETURN>" to leave the HBase Shell
```

讲述了怎么获得帮助，怎么退出客户端

help 获取帮助

　　help：获取所有命令提示

　　help "dml" ：获取一组命令的提示

　　help "put" ：获取一个单独命令的提示帮助

exit 退出 hbase shell 客户端

## HBase表的操作

 关于表的操作包括（创建create，查看表列表list。查看表的详细信息desc，删除表drop，清空表truncate，修改表的定义alter）

### 创建create

可以输入以下命令进行查看帮助命令

```
hbase(main):001:0> help 'create'
```

![img](https://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif) View Code

可以看到其中一条提示

```
hbase> create 't1', {NAME => 'f1'}, {NAME => 'f2'}, {NAME => 'f3'}
```

其中t1是表名，f1,f2,f3是列簇的名，如：

```
hbase(main):002:0> create 'myHbase',{NAME => 'myCard',VERSIONS => 5}
0 row(s) in 3.1270 seconds

=> Hbase::Table - myHbase
hbase(main):003:0> 
```

创建了一个名为myHbase的表，表里面有1个列簇，名为**myCard**，保留5个版本信息

### 查看表列表list

可以输入以下命令进行查看帮助命令

```
hbase(main):003:0> help 'list'
List all tables in hbase. Optional regular expression parameter could
be used to filter the output. Examples:

  hbase> list
  hbase> list 'abc.*'
  hbase> list 'ns:abc.*'
  hbase> list 'ns:.*'
hbase(main):004:0> 
```

直接输入list进行查看

```
hbase(main):004:0> list
TABLE                                                                                                           
myHbase                                                                                                         
1 row(s) in 0.0650 seconds

=> ["myHbase"]
hbase(main):005:0> 
```

只有一条结果，就是刚刚创建的表myHbase

### 查看表的详细信息desc

一个大括号，就相当于一个列簇。

```
hbase(main):006:0> desc 'myHbase'
Table myHbase is ENABLED                                                                                        
myHbase                                                                                                         
COLUMN FAMILIES DESCRIPTION                                                                                     
{NAME => 'myCard', BLOOMFILTER => 'ROW', VERSIONS => '5', IN_MEMORY => 'false', KEEP_DELETED_CELLS => 'FALSE', D
ATA_BLOCK_ENCODING => 'NONE', TTL => 'FOREVER', COMPRESSION => 'NONE', MIN_VERSIONS => '0', BLOCKCACHE => 'true'
, BLOCKSIZE => '65536', REPLICATION_SCOPE => '0'}                                                               
1 row(s) in 0.2160 seconds

hbase(main):007:0> 
```

### 修改表的定义alter

#### 添加一个列簇

hbase(main):007:0> **alter 'myHbase', NAME => 'myInfo'**
Updating all regions with the new schema...
1/1 regions updated.
Done.
0 row(s) in 2.0690 seconds

hbase(main):008:0> **desc 'myHbase'**
Table myHbase is ENABLED
myHbase
COLUMN FAMILIES DESCRIPTION
{NAME => 'myCard', BLOOMFILTER => 'ROW', VERSIONS => '5', IN_MEMORY => 'false', KEEP_DELETED_CELLS => 'FALSE', D
ATA_BLOCK_ENCODING => 'NONE', TTL => 'FOREVER', COMPRESSION => 'NONE', MIN_VERSIONS => '0', BLOCKCACHE => 'true'
, BLOCKSIZE => '65536', REPLICATION_SCOPE => '0'}
{NAME => 'myInfo', BLOOMFILTER => 'ROW', VERSIONS => '1', IN_MEMORY => 'false', KEEP_DELETED_CELLS => 'FALSE', D
ATA_BLOCK_ENCODING => 'NONE', TTL => 'FOREVER', COMPRESSION => 'NONE', MIN_VERSIONS => '0', BLOCKCACHE => 'true'
, BLOCKSIZE => '65536', REPLICATION_SCOPE => '0'}
2 row(s) in 0.0420 seconds

hbase(main):009:0>

#### 删除一个列簇

```
hbase(main):009:0> alter 'myHbase', NAME => 'myCard', METHOD => 'delete'
Updating all regions with the new schema...
1/1 regions updated.
Done.
0 row(s) in 2.1920 seconds

hbase(main):010:0> desc 'myHbase'
Table myHbase is ENABLED                                                                                        
myHbase                                                                                                         
COLUMN FAMILIES DESCRIPTION                                                                                     
{NAME => 'myInfo', BLOOMFILTER => 'ROW', VERSIONS => '1', IN_MEMORY => 'false', KEEP_DELETED_CELLS => 'FALSE', D
ATA_BLOCK_ENCODING => 'NONE', TTL => 'FOREVER', COMPRESSION => 'NONE', MIN_VERSIONS => '0', BLOCKCACHE => 'true'
, BLOCKSIZE => '65536', REPLICATION_SCOPE => '0'}                                                               
1 row(s) in 0.0290 seconds

hbase(main):011:0> 
```

删除一个列簇也可以执行以下命令

```
alter 'myHbase', 'delete' => 'myCard'
```

#### 添加列簇hehe同时删除列簇myInfo

```
hbase(main):011:0> alter 'myHbase', {NAME => 'hehe'}, {NAME => 'myInfo', METHOD => 'delete'}
Updating all regions with the new schema...
1/1 regions updated.
Done.
Updating all regions with the new schema...
1/1 regions updated.
Done.
0 row(s) in 3.8260 seconds

hbase(main):012:0> desc 'myHbase'
Table myHbase is ENABLED                                                                                        
myHbase                                                                                                         
COLUMN FAMILIES DESCRIPTION                                                                                     
{NAME => 'hehe', BLOOMFILTER => 'ROW', VERSIONS => '1', IN_MEMORY => 'false', KEEP_DELETED_CELLS => 'FALSE', DAT
A_BLOCK_ENCODING => 'NONE', TTL => 'FOREVER', COMPRESSION => 'NONE', MIN_VERSIONS => '0', BLOCKCACHE => 'true', 
BLOCKSIZE => '65536', REPLICATION_SCOPE => '0'}                                                                 
1 row(s) in 0.0410 seconds

hbase(main):013:0> 
```

#### 清空表truncate                                          

```
hbase(main):013:0> truncate 'myHbase'
Truncating 'myHbase' table (it may take a while):
 - Disabling table...
 - Truncating table...
0 row(s) in 3.6760 seconds

hbase(main):014:0> 
```

#### 删除表drop

```
hbase(main):014:0> drop 'myHbase'

ERROR: Table myHbase is enabled. Disable it first.

Here is some help for this command:
Drop the named table. Table must first be disabled:
  hbase> drop 't1'
  hbase> drop 'ns1:t1'


hbase(main):015:0> 
```

直接删除表会报错，根据提示需要先停用表

```
hbase(main):015:0> disable 'myHbase'
0 row(s) in 2.2620 seconds

hbase(main):016:0> drop 'myHbase'
0 row(s) in 1.2970 seconds

hbase(main):017:0> list
TABLE                                                                                                           
0 row(s) in 0.0110 seconds

=> []
hbase(main):018:0> 
```

## HBase表中数据的操作

关于数据的操作（增put，删delete，查get + scan, 改==变相的增加）

创建 user 表，包含 info、data 两个列簇

```
hbase(main):018:0> create 'user_info',{NAME=>'base_info',VERSIONS=>3 },{NAME=>'extra_info',VERSIONS=>1 } 
0 row(s) in 4.2670 seconds

=> Hbase::Table - user_info
hbase(main):019:0> 
```

### 新增put

查看帮助，需要传入表名，rowkey，列簇名、值等

```
hbase(main):019:0> help 'put'
Put a cell 'value' at specified table/row/column and optionally
timestamp coordinates.  To put a cell value into table 'ns1:t1' or 't1'
at row 'r1' under column 'c1' marked with the time 'ts1', do:

  hbase> put 'ns1:t1', 'r1', 'c1', 'value'
  hbase> put 't1', 'r1', 'c1', 'value'
  hbase> put 't1', 'r1', 'c1', 'value', ts1
  hbase> put 't1', 'r1', 'c1', 'value', {ATTRIBUTES=>{'mykey'=>'myvalue'}}
  hbase> put 't1', 'r1', 'c1', 'value', ts1, {ATTRIBUTES=>{'mykey'=>'myvalue'}}
  hbase> put 't1', 'r1', 'c1', 'value', ts1, {VISIBILITY=>'PRIVATE|SECRET'}

The same commands also can be run on a table reference. Suppose you had a reference
t to table 't1', the corresponding command would be:

  hbase> t.put 'r1', 'c1', 'value', ts1, {ATTRIBUTES=>{'mykey'=>'myvalue'}}
hbase(main):020:0> 
```

 向 user 表中插入信息，row key 为 user0001，列簇 base_info 中添加 name 列标示符，值为 zhangsan1

```
hbase(main):020:0> put 'user_info', 'user0001', 'base_info:name', 'zhangsan1'
0 row(s) in 0.2900 seconds

hbase(main):021:0> 
```

此处可以多添加几条数据

```
put 'user_info', 'zhangsan_20150701_0001', 'base_info:name', 'zhangsan1'
put 'user_info', 'zhangsan_20150701_0002', 'base_info:name', 'zhangsan2'
put 'user_info', 'zhangsan_20150701_0003', 'base_info:name', 'zhangsan3'
put 'user_info', 'zhangsan_20150701_0004', 'base_info:name', 'zhangsan4'
put 'user_info', 'zhangsan_20150701_0005', 'base_info:name', 'zhangsan5'
put 'user_info', 'zhangsan_20150701_0006', 'base_info:name', 'zhangsan6'
put 'user_info', 'zhangsan_20150701_0007', 'base_info:name', 'zhangsan7'
put 'user_info', 'zhangsan_20150701_0008', 'base_info:name', 'zhangsan8'

put 'user_info', 'zhangsan_20150701_0001', 'base_info:age', '21'
put 'user_info', 'zhangsan_20150701_0002', 'base_info:age', '22'
put 'user_info', 'zhangsan_20150701_0003', 'base_info:age', '23'
put 'user_info', 'zhangsan_20150701_0004', 'base_info:age', '24'
put 'user_info', 'zhangsan_20150701_0005', 'base_info:age', '25'
put 'user_info', 'zhangsan_20150701_0006', 'base_info:age', '26'
put 'user_info', 'zhangsan_20150701_0007', 'base_info:age', '27'
put 'user_info', 'zhangsan_20150701_0008', 'base_info:age', '28'

put 'user_info', 'zhangsan_20150701_0001', 'extra_info:Hobbies', 'music'
put 'user_info', 'zhangsan_20150701_0002', 'extra_info:Hobbies', 'sport'
put 'user_info', 'zhangsan_20150701_0003', 'extra_info:Hobbies', 'music'
put 'user_info', 'zhangsan_20150701_0004', 'extra_info:Hobbies', 'sport'
put 'user_info', 'zhangsan_20150701_0005', 'extra_info:Hobbies', 'music'
put 'user_info', 'zhangsan_20150701_0006', 'extra_info:Hobbies', 'sport'
put 'user_info', 'zhangsan_20150701_0007', 'extra_info:Hobbies', 'music'

put 'user_info', 'baiyc_20150716_0001', 'base_info:name', 'baiyc1'
put 'user_info', 'baiyc_20150716_0002', 'base_info:name', 'baiyc2'
put 'user_info', 'baiyc_20150716_0003', 'base_info:name', 'baiyc3'
put 'user_info', 'baiyc_20150716_0004', 'base_info:name', 'baiyc4'
put 'user_info', 'baiyc_20150716_0005', 'base_info:name', 'baiyc5'
put 'user_info', 'baiyc_20150716_0006', 'base_info:name', 'baiyc6'
put 'user_info', 'baiyc_20150716_0007', 'base_info:name', 'baiyc7'
put 'user_info', 'baiyc_20150716_0008', 'base_info:name', 'baiyc8'

put 'user_info', 'baiyc_20150716_0001', 'base_info:age', '21'
put 'user_info', 'baiyc_20150716_0002', 'base_info:age', '22'
put 'user_info', 'baiyc_20150716_0003', 'base_info:age', '23'
put 'user_info', 'baiyc_20150716_0004', 'base_info:age', '24'
put 'user_info', 'baiyc_20150716_0005', 'base_info:age', '25'
put 'user_info', 'baiyc_20150716_0006', 'base_info:age', '26'
put 'user_info', 'baiyc_20150716_0007', 'base_info:age', '27'
put 'user_info', 'baiyc_20150716_0008', 'base_info:age', '28'

put 'user_info', 'baiyc_20150716_0001', 'extra_info:Hobbies', 'music'
put 'user_info', 'baiyc_20150716_0002', 'extra_info:Hobbies', 'sport'
put 'user_info', 'baiyc_20150716_0003', 'extra_info:Hobbies', 'music'
put 'user_info', 'baiyc_20150716_0004', 'extra_info:Hobbies', 'sport'
put 'user_info', 'baiyc_20150716_0005', 'extra_info:Hobbies', 'music'
put 'user_info', 'baiyc_20150716_0006', 'extra_info:Hobbies', 'sport'
put 'user_info', 'baiyc_20150716_0007', 'extra_info:Hobbies', 'music'
put 'user_info', 'baiyc_20150716_0008', 'extra_info:Hobbies', 'sport'
```



### 查get + scan

获取 user 表中 row key 为 user0001 的所有信息

```
hbase(main):022:0> get 'user_info', 'user0001'
COLUMN                        CELL                                                                              
 base_info:name               timestamp=1522320801670, value=zhangsan1                                          
1 row(s) in 0.1310 seconds

hbase(main):023:0> 
```

获取user表中row key为rk0001，info列簇的所有信息

```
hbase(main):025:0> get 'user_info', 'rk0001', 'base_info'
COLUMN                        CELL                                                                              
 base_info:name               timestamp=1522321247732, value=zhangsan                                           
1 row(s) in 0.0320 seconds

hbase(main):026:0> 
```

查询user_info表中的所有信息

```
hbase(main):026:0> scan 'user_info'
ROW                           COLUMN+CELL                                                                       
 rk0001                       column=base_info:name, timestamp=1522321247732, value=zhangsan                    
 user0001                     column=base_info:name, timestamp=1522320801670, value=zhangsan1                   
2 row(s) in 0.0970 seconds

hbase(main):027:0> 
```

查询user_info表中列簇为base_info的信息

```
hbase(main):027:0> scan 'user_info', {COLUMNS => 'base_info'}
ROW                           COLUMN+CELL                                                                       
 rk0001                       column=base_info:name, timestamp=1522321247732, value=zhangsan                    
 user0001                     column=base_info:name, timestamp=1522320801670, value=zhangsan1                   
2 row(s) in 0.0620 seconds

hbase(main):028:0> 
```

### 删delete

删除user_info表row key为rk0001，列标示符为base_info:name的数据

```
hbase(main):028:0> delete 'user_info', 'rk0001', 'base_info:name'
0 row(s) in 0.0780 seconds

hbase(main):029:0> scan 'user_info', {COLUMNS => 'base_info'}
ROW                           COLUMN+CELL                                                                       
 user0001                     column=base_info:name, timestamp=1522320801670, value=zhangsan1                   
1 row(s) in 0.0530 seconds

hbase(main):030:0> 
```

