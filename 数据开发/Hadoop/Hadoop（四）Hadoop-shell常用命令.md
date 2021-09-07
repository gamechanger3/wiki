<p align="center">
    <img width="280px" src="image/konglong/m2.png" >
</p>

# Hadoop（四）Hadoop shell常用命令

## Hadoop常用命令

### 启动HDFS集群

```
[hadoop@hadoop1 ~]$ start-dfs.sh
Starting namenodes on [hadoop1]
hadoop1: starting namenode, logging to /home/hadoop/apps/hadoop-2.7.5/logs/hadoop-hadoop-namenode-hadoop1.out
hadoop2: starting datanode, logging to /home/hadoop/apps/hadoop-2.7.5/logs/hadoop-hadoop-datanode-hadoop2.out
hadoop3: starting datanode, logging to /home/hadoop/apps/hadoop-2.7.5/logs/hadoop-hadoop-datanode-hadoop3.out
hadoop4: starting datanode, logging to /home/hadoop/apps/hadoop-2.7.5/logs/hadoop-hadoop-datanode-hadoop4.out
hadoop1: starting datanode, logging to /home/hadoop/apps/hadoop-2.7.5/logs/hadoop-hadoop-datanode-hadoop1.out
Starting secondary namenodes [hadoop3]
hadoop3: starting secondarynamenode, logging to /home/hadoop/apps/hadoop-2.7.5/logs/hadoop-hadoop-secondarynamenode-hadoop3.out
[hadoop@hadoop1 ~]$ 
```

### 启动YARN集群

```
[hadoop@hadoop4 ~]$ start-yarn.sh
starting yarn daemons
starting resourcemanager, logging to /home/hadoop/apps/hadoop-2.7.5/logs/yarn-hadoop-resourcemanager-hadoop4.out
hadoop2: starting nodemanager, logging to /home/hadoop/apps/hadoop-2.7.5/logs/yarn-hadoop-nodemanager-hadoop2.out
hadoop3: starting nodemanager, logging to /home/hadoop/apps/hadoop-2.7.5/logs/yarn-hadoop-nodemanager-hadoop3.out
hadoop4: starting nodemanager, logging to /home/hadoop/apps/hadoop-2.7.5/logs/yarn-hadoop-nodemanager-hadoop4.out
hadoop1: starting nodemanager, logging to /home/hadoop/apps/hadoop-2.7.5/logs/yarn-hadoop-nodemanager-hadoop1.out
[hadoop@hadoop4 ~]$ 
```

### 查看HDFS系统根目录

```
[hadoop@hadoop1 ~]$ hadoop fs -ls /
Found 2 items
drwxr-xr-x   - hadoop supergroup          0 2018-03-03 11:42 /test
drwx------   - hadoop supergroup          0 2018-03-03 11:42 /tmp
[hadoop@hadoop1 ~]$ 
```



### 创建文件夹

```
[hadoop@hadoop1 ~]$ hadoop fs -mkdir /a
[hadoop@hadoop1 ~]$ hadoop fs -ls /
Found 3 items
drwxr-xr-x   - hadoop supergroup          0 2018-03-08 11:09 /a
drwxr-xr-x   - hadoop supergroup          0 2018-03-03 11:42 /test
drwx------   - hadoop supergroup          0 2018-03-03 11:42 /tmp
[hadoop@hadoop1 ~]$ 
```



### 级联创建文件夹

```
[hadoop@hadoop1 ~]$ hadoop fs -mkdir -p /aa/bb/cc
[hadoop@hadoop1 ~]$ 
```



### 查看hsdf系统根目录下的所有文件包括子文件夹里面的文件

**[hadoop@hadoop1 ~]$ hadoop fs -ls -R /aa**
drwxr-xr-x - hadoop supergroup 0 2018-03-08 11:12 /aa/bb
drwxr-xr-x - hadoop supergroup 0 2018-03-08 11:12 /aa/bb/cc
[hadoop@hadoop1 ~]$

### 上传文件

[hadoop@hadoop1 ~]$ ls
apps data words.txt
**[hadoop@hadoop1 ~]$ hadoop fs -put words.txt /aa**
**[hadoop@hadoop1 ~]$ hadoop fs -copyFromLocal words.txt /aa/bb**
[hadoop@hadoop1 ~]$

### 下载文件

```
[hadoop@hadoop1 ~]$ hadoop fs -get /aa/words.txt ~/newwords.txt
[hadoop@hadoop1 ~]$ ls
apps  data  newwords.txt  words.txt
[hadoop@hadoop1 ~]$ hadoop fs -copyToLocal /aa/words.txt ~/newwords1.txt
[hadoop@hadoop1 ~]$ ls
apps  data  newwords1.txt  newwords.txt  words.txt
[hadoop@hadoop1 ~]$ 
```

### 合并下载

```
[hadoop@hadoop1 ~]$ hadoop fs -getmerge /aa/words.txt /aa/bb/words.txt ~/2words.txt
[hadoop@hadoop1 ~]$ ll
总用量 24
-rw-r--r--. 1 hadoop hadoop   78 3月   8 12:42 2words.txt
drwxrwxr-x. 3 hadoop hadoop 4096 3月   3 10:30 apps
drwxrwxr-x. 3 hadoop hadoop 4096 3月   3 11:40 data
-rw-r--r--. 1 hadoop hadoop   39 3月   8 11:49 newwords1.txt
-rw-r--r--. 1 hadoop hadoop   39 3月   8 11:48 newwords.txt
-rw-rw-r--. 1 hadoop hadoop   39 3月   3 11:31 words.txt
[hadoop@hadoop1 ~]$ 
```

### 复制

从HDFS一个路径拷贝到HDFS另一个路径

```
[hadoop@hadoop1 ~]$ hadoop fs -ls /a
[hadoop@hadoop1 ~]$ hadoop fs -cp /aa/words.txt /a
[hadoop@hadoop1 ~]$ hadoop fs -ls /a
Found 1 items
-rw-r--r--   2 hadoop supergroup         39 2018-03-08 12:46 /a/words.txt
[hadoop@hadoop1 ~]$ 
```



### 移动

在HDFS目录中移动文件

```
[hadoop@hadoop1 ~]$ hadoop fs -ls /aa/bb/cc
[hadoop@hadoop1 ~]$ hadoop fs -mv /a/words.txt /aa/bb/cc
[hadoop@hadoop1 ~]$ hadoop fs -ls /aa/bb/cc
Found 1 items
-rw-r--r--   2 hadoop supergroup         39 2018-03-08 12:46 /aa/bb/cc/words.txt
[hadoop@hadoop1 ~]$ 
```



### 删除

删除文件或文件夹

```
[hadoop@hadoop1 ~]$ hadoop fs -rm /aa/bb/cc/words.txt
18/03/08 12:49:08 INFO fs.TrashPolicyDefault: Namenode trash configuration: Deletion interval = 0 minutes, Emptier interval = 0 minutes.
Deleted /aa/bb/cc/words.txt
[hadoop@hadoop1 ~]$ hadoop fs -ls /aa/bb/cc
[hadoop@hadoop1 ~]$ 
```



删除空目录

```
[hadoop@hadoop1 ~]$ hadoop fs -rmdir /aa/bb/cc/
[hadoop@hadoop1 ~]$ hadoop fs -ls /aa/bb/
Found 1 items
-rw-r--r--   2 hadoop supergroup         39 2018-03-08 11:43 /aa/bb/words.txt
[hadoop@hadoop1 ~]$ 
```

强制删除

```
[hadoop@hadoop1 ~]$ hadoop fs -rm /aa/bb/
rm: `/aa/bb': Is a directory
[hadoop@hadoop1 ~]$ hadoop fs -rm -r /aa/bb/
18/03/08 12:51:31 INFO fs.TrashPolicyDefault: Namenode trash configuration: Deletion interval = 0 minutes, Emptier interval = 0 minutes.
Deleted /aa/bb
[hadoop@hadoop1 ~]$ hadoop fs -ls /aa
Found 1 items
-rw-r--r--   2 hadoop supergroup         39 2018-03-08 11:41 /aa/words.txt
[hadoop@hadoop1 ~]$ 
```



### 从本地剪切文件到HDFS上

```
[hadoop@hadoop1 ~]$ ls
apps  data  hello.txt
[hadoop@hadoop1 ~]$ hadoop fs -moveFromLocal ~/hello.txt /aa
[hadoop@hadoop1 ~]$ ls
apps  data
[hadoop@hadoop1 ~]$ 
```



### 追加文件

追加之前hello.txt到words.txt之前

![img](https://images2018.cnblogs.com/blog/1228818/201803/1228818-20180308125830426-570415929.png)

```
[hadoop@hadoop1 ~]$ hadoop fs -appendToFile ~/hello.txt /aa/words.txt
[hadoop@hadoop1 ~]$ 
```

追加之前hello.txt到words.txt之后

![img](https://images2018.cnblogs.com/blog/1228818/201803/1228818-20180308130118866-1783962726.png)



### 查看文件内容

```
[hadoop@hadoop1 ~]$ hadoop fs -cat /aa/hello.txt
hello
hello
hello
[hadoop@hadoop1 ~]$ 
```



### chgrp

使用方法：hadoop fs -chgrp [-R] GROUP URI [URI …] Change group association of files. With -R, make the change recursively through the directory structure. The user must be the owner of files, or else a super-user. Additional information is in the [Permissions User Guide](http://hadoop.apache.org/docs/r1.0.4/cn/hdfs_permissions_guide.html). -->

改变文件所属的组。使用-R将使改变在目录结构下递归进行。命令的使用者必须是文件的所有者或者超级用户。更多的信息请参见[HDFS权限用户指南](http://hadoop.apache.org/docs/r1.0.4/cn/hdfs_permissions_guide.html)。

### chmod

使用方法：hadoop fs -chmod [-R] <MODE[,MODE]... | OCTALMODE> URI [URI …]

改变文件的权限。使用-R将使改变在目录结构下递归进行。命令的使用者必须是文件的所有者或者超级用户。更多的信息请参见[HDFS权限用户指南](http://hadoop.apache.org/docs/r1.0.4/cn/hdfs_permissions_guide.html)。

### chown

使用方法：hadoop fs -chown [-R] [OWNER][:[GROUP]] URI [URI ]

改变文件的拥有者。使用-R将使改变在目录结构下递归进行。命令的使用者必须是超级用户。更多的信息请参见[HDFS权限用户指南](http://hadoop.apache.org/docs/r1.0.4/cn/hdfs_permissions_guide.html)。

### du

使用方法：hadoop fs -du URI [URI …]

显示目录中所有文件的大小，或者当只指定一个文件时，显示此文件的大小。
示例：
hadoop fs -du /user/hadoop/dir1 /user/hadoop/file1 hdfs://host:port/user/hadoop/dir1 
返回值：
成功返回0，失败返回-1。 

### dus

使用方法：hadoop fs -dus <args>

显示文件的大小。

### expunge

使用方法：hadoop fs -expunge

清空回收站。请参考[HDFS设计](http://hadoop.apache.org/docs/r1.0.4/cn/hdfs_design.html)文档以获取更多关于回收站特性的信息。

### setrep

使用方法：hadoop fs -setrep [-R] <path>

改变一个文件的副本系数。-R选项用于递归改变目录下所有文件的副本系数。

示例：

- hadoop fs -setrep -w 3 -R /user/hadoop/dir1

返回值：

成功返回0，失败返回-1。

### tail

使用方法：hadoop fs -tail [-f] URI

将文件尾部1K字节的内容输出到stdout。支持-f选项，行为和Unix中一致。

示例：

- hadoop fs -tail pathname

返回值：
成功返回0，失败返回-1。

### test

使用方法：hadoop fs -test -[ezd] URI

选项：
-e 检查文件是否存在。如果存在则返回0。
-z 检查文件是否是0字节。如果是则返回0。 
-d 如果路径是个目录，则返回1，否则返回0。

示例：

- - hadoop fs -test -e filename

###  查看集群的工作状态

```
[hadoop@hadoop1 ~]$ hdfs dfsadmin -report
Configured Capacity: 73741402112 (68.68 GB)
Present Capacity: 52781039616 (49.16 GB)
DFS Remaining: 52780457984 (49.16 GB)
DFS Used: 581632 (568 KB)
DFS Used%: 0.00%
Under replicated blocks: 0
Blocks with corrupt replicas: 0
Missing blocks: 0
Missing blocks (with replication factor 1): 0

-------------------------------------------------
Live datanodes (4):

Name: 192.168.123.102:50010 (hadoop1)
Hostname: hadoop1
Decommission Status : Normal
Configured Capacity: 18435350528 (17.17 GB)
DFS Used: 114688 (112 KB)
Non DFS Used: 4298661888 (4.00 GB)
DFS Remaining: 13193277440 (12.29 GB)
DFS Used%: 0.00%
DFS Remaining%: 71.57%
Configured Cache Capacity: 0 (0 B)
Cache Used: 0 (0 B)
Cache Remaining: 0 (0 B)
Cache Used%: 100.00%
Cache Remaining%: 0.00%
Xceivers: 1
Last contact: Thu Mar 08 13:05:11 CST 2018


Name: 192.168.123.105:50010 (hadoop4)
Hostname: hadoop4
Decommission Status : Normal
Configured Capacity: 18435350528 (17.17 GB)
DFS Used: 49152 (48 KB)
Non DFS Used: 4295872512 (4.00 GB)
DFS Remaining: 13196132352 (12.29 GB)
DFS Used%: 0.00%
DFS Remaining%: 71.58%
Configured Cache Capacity: 0 (0 B)
Cache Used: 0 (0 B)
Cache Remaining: 0 (0 B)
Cache Used%: 100.00%
Cache Remaining%: 0.00%
Xceivers: 1
Last contact: Thu Mar 08 13:05:13 CST 2018


Name: 192.168.123.103:50010 (hadoop2)
Hostname: hadoop2
Decommission Status : Normal
Configured Capacity: 18435350528 (17.17 GB)
DFS Used: 233472 (228 KB)
Non DFS Used: 4295700480 (4.00 GB)
DFS Remaining: 13196120064 (12.29 GB)
DFS Used%: 0.00%
DFS Remaining%: 71.58%
Configured Cache Capacity: 0 (0 B)
Cache Used: 0 (0 B)
Cache Remaining: 0 (0 B)
Cache Used%: 100.00%
Cache Remaining%: 0.00%
Xceivers: 1
Last contact: Thu Mar 08 13:05:11 CST 2018


Name: 192.168.123.104:50010 (hadoop3)
Hostname: hadoop3
Decommission Status : Normal
Configured Capacity: 18435350528 (17.17 GB)
DFS Used: 184320 (180 KB)
Non DFS Used: 4296941568 (4.00 GB)
DFS Remaining: 13194928128 (12.29 GB)
DFS Used%: 0.00%
DFS Remaining%: 71.57%
Configured Cache Capacity: 0 (0 B)
Cache Used: 0 (0 B)
Cache Remaining: 0 (0 B)
Cache Used%: 100.00%
Cache Remaining%: 0.00%
Xceivers: 1
Last contact: Thu Mar 08 13:05:10 CST 2018


[hadoop@hadoop1 ~]$ 
```

