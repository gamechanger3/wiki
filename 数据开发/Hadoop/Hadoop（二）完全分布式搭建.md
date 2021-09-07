<p align="center">
    <img width="280px" src="image/konglong/m2.png" >
</p>
# Hadoop（二）完全分布式搭建

## 概念了解

主从结构：在一个集群中，会有部分节点充当主服务器的角色，其他服务器都是从服务器的角色，当前这种架构模式叫做主从结构。

主从结构分类：

1、一主多从

2、多主多从

Hadoop中的HDFS和YARN都是主从结构，主从结构中的主节点和从节点有多重概念方式：

1、主节点　　从节点

2、master　　slave

3、管理者　　工作者

4、leader　　follower

Hadoop集群中各个角色的名称：

| 服务 | 主节点          | 从节点      |
| ---- | --------------- | ----------- |
| HDFS | NameNode        | DataNode    |
| YARN | ResourceManager | NodeManager |



## 集群服务器规划

使用4台CentOS-6.7虚拟机进行集群搭建

![img](https://images2018.cnblogs.com/blog/1228818/201803/1228818-20180303115310907-1555873629.png)



## 软件安装步骤概述

1、获取安装包

2、解压缩和安装

3、修改配置文件

4、初始化，配置环境变量，启动，验证



## Hadoop安装



### 1、规划

规划安装用户：hadoop

规划安装目录：/home/hadoop/apps

规划数据目录：/home/hadoop/data

注：apps和data文件夹需要自己单独创建



### 2、上传解压缩

注：使用hadoop用户

```
[hadoop@hadoop1 apps]$ ls
hadoop-2.7.5-centos-6.7.tar.gz
[hadoop@hadoop1 apps]$ tar -zxvf hadoop-2.7.5-centos-6.7.tar.gz 
```



### 3、修改配置文件

配置文件目录：/home/hadoop/apps/hadoop-2.7.5/etc/hadoop

![img](https://images2018.cnblogs.com/blog/1228818/201803/1228818-20180303103331419-2007341529.png)

#### A.　hadoop-env.sh

```
[hadoop@hadoop1 hadoop]$ vi hadoop-env.sh 
```

修改JAVA_HOME

```
export JAVA_HOME=/usr/local/jdk1.8.0_73
```

![img](https://images2018.cnblogs.com/blog/1228818/201803/1228818-20180303104035548-3546092.png)

B.　core-site.xml

```
[hadoop@hadoop1 hadoop]$ vi core-site.xml 
```

fs.defaultFS ： 这个属性用来指定namenode的hdfs协议的文件系统通信地址，可以指定一个主机+端口，也可以指定为一个namenode服务（这个服务内部可以有多台namenode实现ha的namenode服务

hadoop.tmp.dir : hadoop集群在工作的时候存储的一些临时文件的目录



```xml
<configuration>
        <property>
                <name>fs.defaultFS</name>
                <value>hdfs://hadoop1:9000</value>
        </property>
        <property>
                <name>hadoop.tmp.dir</name>
                <value>/home/hadoop/data/hadoopdata</value>
        </property>
</configuration>
```

 ![img](https://images2018.cnblogs.com/blog/1228818/201803/1228818-20180303104511807-906110891.png)

C.　hdfs-site.xml

```
[hadoop@hadoop1 hadoop]$ vi hdfs-site.xml 
```

 dfs.namenode.name.dir：namenode数据的存放地点。也就是namenode元数据存放的地方，记录了hdfs系统中文件的元数据。

 dfs.datanode.data.dir： datanode数据的存放地点。也就是block块存放的目录了。

dfs.replication：hdfs的副本数设置。也就是上传一个文件，其分割为block块后，每个block的冗余副本个数，默认配置是3。

dfs.secondary.http.address：secondarynamenode 运行节点的信息，和 namenode 不同节点



```xml
<configuration>
        <property>
                <name>dfs.namenode.name.dir</name>
                <value>/home/hadoop/data/hadoopdata/name</value>
                <description>为了保证元数据的安全一般配置多个不同目录</description>
        </property>

        <property>
                <name>dfs.datanode.data.dir</name>
                <value>/home/hadoop/data/hadoopdata/data</value>
                <description>datanode 的数据存储目录</description>
        </property>

        <property>
                <name>dfs.replication</name>
                <value>2</value>
                <description>HDFS 的数据块的副本存储个数, 默认是3</description>
        </property>

        <property>
                <name>dfs.secondary.http.address</name>
                <value>hadoop3:50090</value>
                <description>secondarynamenode 运行节点的信息，和 namenode 不同节点</description>
        </property>
</configuration>
```



![img](https://images2018.cnblogs.com/blog/1228818/201803/1228818-20180303104911508-1011649393.png)

D.　mapred-site.xml

```
[hadoop@hadoop1 hadoop]$ cp mapred-site.xml.template mapred-site.xml
[hadoop@hadoop1 hadoop]$ vi mapred-site.xml
```

 mapreduce.framework.name：指定mr框架为yarn方式,Hadoop二代MP也基于资源管理系统Yarn来运行 。



```xml
<configuration>
        <property>
                <name>mapreduce.framework.name</name>
                <value>yarn</value>
        </property>
</configuration>
```



![img](https://images2018.cnblogs.com/blog/1228818/201803/1228818-20180303105128551-1997748350.png)

E.　yarn-site.xml

```
[hadoop@hadoop1 hadoop]$ vi yarn-site.xml 
```

 yarn.resourcemanager.hostname：yarn总管理器的IPC通讯地址

 yarn.nodemanager.aux-services：



```xml
<configuration>

<!-- Site specific YARN configuration properties -->

        <property>
                <name>yarn.resourcemanager.hostname</name>
                <value>hadoop4</value>
        </property>
        
        <property>
                <name>yarn.nodemanager.aux-services</name>
                <value>mapreduce_shuffle</value>
                <description>YARN 集群为 MapReduce 程序提供的 shuffle 服务</description>
        </property>

</configuration>
```



 ![img](https://images2018.cnblogs.com/blog/1228818/201803/1228818-20180303105358230-1169766109.png)

F.　slaves

```
[hadoop@hadoop1 hadoop]$ vi slaves 
hadoop1
hadoop2
hadoop3
hadoop4
```

![img](https://images2018.cnblogs.com/blog/1228818/201803/1228818-20180303105518279-1427162583.png)



### 4、把安装包分别分发给其他的节点

重点强调： 每台服务器中的hadoop安装包的目录必须一致， 安装包的配置信息还必须保持一致
重点强调： 每台服务器中的hadoop安装包的目录必须一致， 安装包的配置信息还必须保持一致
重点强调： 每台服务器中的hadoop安装包的目录必须一致， 安装包的配置信息还必须保持一致

```
[hadoop@hadoop1 hadoop]$ scp -r ~/apps/hadoop-2.7.5/ hadoop2:~/apps/
[hadoop@hadoop1 hadoop]$ scp -r ~/apps/hadoop-2.7.5/ hadoop3:~/apps/
[hadoop@hadoop1 hadoop]$ scp -r ~/apps/hadoop-2.7.5/ hadoop4:~/apps/
```

注意：上面的命令等同于下面的命令

```
[hadoop@hadoop1 hadoop]$ scp -r ~/apps/hadoop-2.7.5/ hadoop@hadoop2:~/apps/
```



### 5、配置Hadoop环境变量

千万注意：

1、如果你使用root用户进行安装。 vi /etc/profile 即可 系统变量

2、如果你使用普通用户进行安装。 vi ~/.bashrc 用户变量

```
[hadoop@hadoop1 ~]$ vi .bashrc
export HADOOP_HOME=/home/hadoop/apps/hadoop-2.7.5
export PATH=$PATH:$HADOOP_HOME/bin:$HADOOP_HOME/sbin:
```

![img](https://images2018.cnblogs.com/blog/1228818/201803/1228818-20180303110711007-1612467070.png)

使环境变量生效

```
[hadoop@hadoop1 bin]$ source ~/.bashrc 
```



### 6、查看hadoop版本



```
[hadoop@hadoop1 bin]$ hadoop version
Hadoop 2.7.5
Subversion Unknown -r Unknown
Compiled by root on 2017-12-24T05:30Z
Compiled with protoc 2.5.0
From source with checksum 9f118f95f47043332d51891e37f736e9
This command was run using /home/hadoop/apps/hadoop-2.7.5/share/hadoop/common/hadoop-common-2.7.5.jar
[hadoop@hadoop1 bin]$ 
```



![img](https://images2018.cnblogs.com/blog/1228818/201803/1228818-20180303111035963-1326015903.png)



### 7、Hadoop初始化

注意：HDFS初始化只能在主节点上进行

```
[hadoop@hadoop1 ~]$ hadoop namenode -format
```

![img](https://images2018.cnblogs.com/blog/1228818/201803/1228818-20180303111431844-1654882274.png)



### 8、启动

A.　启动HDFS

注意：不管在集群中的那个节点都可以

```
[hadoop@hadoop1 ~]$ start-dfs.sh
Starting namenodes on [hadoop1]
hadoop1: starting namenode, logging to /home/hadoop/apps/hadoop-2.7.5/logs/hadoop-hadoop-namenode-hadoop1.out
hadoop3: starting datanode, logging to /home/hadoop/apps/hadoop-2.7.5/logs/hadoop-hadoop-datanode-hadoop3.out
hadoop2: starting datanode, logging to /home/hadoop/apps/hadoop-2.7.5/logs/hadoop-hadoop-datanode-hadoop2.out
hadoop4: starting datanode, logging to /home/hadoop/apps/hadoop-2.7.5/logs/hadoop-hadoop-datanode-hadoop4.out
hadoop1: starting datanode, logging to /home/hadoop/apps/hadoop-2.7.5/logs/hadoop-hadoop-datanode-hadoop1.out
Starting secondary namenodes [hadoop3]
hadoop3: starting secondarynamenode, logging to /home/hadoop/apps/hadoop-2.7.5/logs/hadoop-hadoop-secondarynamenode-hadoop3.out
[hadoop@hadoop1 ~]$ 
```

![img](https://images2018.cnblogs.com/blog/1228818/201803/1228818-20180303111708314-1318291140.png)

B.　启动YARN

注意：只能在主节点中进行启动

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



![img](https://images2018.cnblogs.com/blog/1228818/201803/1228818-20180303112021638-408158798.png)



### 9、查看4台服务器的进程

hadoop1

![img](https://images2018.cnblogs.com/blog/1228818/201803/1228818-20180303112120800-1423701937.png)

hadoop2

![img](https://images2018.cnblogs.com/blog/1228818/201803/1228818-20180303112139568-661988009.png)

hadoop3

![img](https://images2018.cnblogs.com/blog/1228818/201803/1228818-20180303112211083-74589397.png)

hadoop4

![img](https://images2018.cnblogs.com/blog/1228818/201803/1228818-20180303112225280-1304512419.png)



### 10、启动HDFS和YARN的web管理界面

HDFS : http://192.168.123.102:50070
YARN ： http://hadoop05:8088

疑惑： fs.defaultFS = hdfs://hadoop02:9000

解答：客户单访问HDFS集群所使用的URL地址

同时，HDFS提供了一个web管理界面 端口：50070

#### HDFS界面

![img](https://images2018.cnblogs.com/blog/1228818/201803/1228818-20180303112510116-894901884.png)

点击Datanodes可以查看四个节点

![img](https://images2018.cnblogs.com/blog/1228818/201803/1228818-20180303112628507-1186899766.png)

#### YARN界面

![img](https://images2018.cnblogs.com/blog/1228818/201803/1228818-20180303112735895-1867837449.png)

点击Nodes可以查看节点

![img](https://images2018.cnblogs.com/blog/1228818/201803/1228818-20180303112833214-1809517028.png)



## Hadoop的简单使用



### 创建文件夹

在HDFS上创建一个文件夹/test/input

```
[hadoop@hadoop1 ~]$ hadoop fs -mkdir -p /test/input
```



### 查看创建的文件夹



```
[hadoop@hadoop1 ~]$ hadoop fs -ls /
Found 1 items
drwxr-xr-x   - hadoop supergroup          0 2018-03-03 11:33 /test
[hadoop@hadoop1 ~]$ hadoop fs -ls /test
Found 1 items
drwxr-xr-x   - hadoop supergroup          0 2018-03-03 11:33 /test/input
[hadoop@hadoop1 ~]$ 
```



![img](https://images2018.cnblogs.com/blog/1228818/201803/1228818-20180303113610630-1170857589.png)



### 上传文件

创建一个文件words.txt

```
[hadoop@hadoop1 ~]$ vi words.txt
hello zhangsan
hello lisi
hello wangwu
```

上传到HDFS的/test/input文件夹中

```
[hadoop@hadoop1 ~]$ hadoop fs -put ~/words.txt /test/input
```

 查看是否上传成功

```
[hadoop@hadoop1 ~]$ hadoop fs -ls /test/input
Found 1 items
-rw-r--r--   2 hadoop supergroup         39 2018-03-03 11:37 /test/input/words.txt
[hadoop@hadoop1 ~]$ 
```

![img](https://images2018.cnblogs.com/blog/1228818/201803/1228818-20180303113847538-1539539423.png)



### 下载文件

将刚刚上传的文件下载到~/data文件夹中

```
[hadoop@hadoop1 ~]$ hadoop fs -get /test/input/words.txt ~/data
```

查看是否下载成功

```
[hadoop@hadoop1 ~]$ ls data
hadoopdata  words.txt
[hadoop@hadoop1 ~]$ 
```

![img](https://images2018.cnblogs.com/blog/1228818/201803/1228818-20180303114047756-345390954.png)



### 运行一个mapreduce的例子程序： wordcount

```
[hadoop@hadoop1 ~]$ hadoop jar ~/apps/hadoop-2.7.5/share/hadoop/mapreduce/hadoop-mapreduce-examples-2.7.5.jar wordcount /test/input /test/output
```

![img](https://images2018.cnblogs.com/blog/1228818/201803/1228818-20180303114406061-137199367.png)

在YARN Web界面查看

![img](https://images2018.cnblogs.com/blog/1228818/201803/1228818-20180303114707863-2058235690.png)

 

查看结果



```
[hadoop@hadoop1 ~]$ hadoop fs -ls /test/output
Found 2 items
-rw-r--r--   2 hadoop supergroup          0 2018-03-03 11:42 /test/output/_SUCCESS
-rw-r--r--   2 hadoop supergroup         35 2018-03-03 11:42 /test/output/part-r-00000
[hadoop@hadoop1 ~]$ hadoop fs -cat /test/output/part-r-00000
hello    3
lisi    1
wangwu    1
zhangsan    1
[hadoop@hadoop1 ~]$ 
```



![img](https://images2018.cnblogs.com/blog/1228818/201803/1228818-20180303114542861-1807717747.png)

 