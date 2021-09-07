<p align="center">
    <img width="280px" src="image/konglong/m3.png" >
</p>


# Hadoop（五）HDFS基础

## HDFS前言

HDFS：Hadoop Distributed File System ，Hadoop分布式文件系统，主要用来解决海量数据的存储问题

### 设计思想

1、分散均匀存储 dfs.blocksize = 128M

2、备份冗余存储 dfs.replication = 3

### 在大数据系统中作用

为各类分布式运算框架（如：mapreduce，spark，tez，……）提供数据存储服务。

### 重点概念

文件切块，副本存放，元数据

## **HDFS的概念和特性**

### **概念**

**首先，它是一个文件系统**，用于存储文件，通过统一的命名空间——目录树来定位文件

**其次，它是分布式的**，由很多服务器联合起来实现其功能，集群中的服务器有各自的角色；

### **重要特性**

（1）HDFS中的文件在物理上是**分块存储（block）**，块的大小可以通过配置参数( dfs.blocksize)来规定，默认大小在hadoop2.x版本中是128M，老版本中是64M

（2）HDFS文件系统会给客户端提供一个**统一的抽象目录树**，客户端通过路径来访问文件，形如：hdfs://namenode:port/dir-a/dir-b/dir-c/file.data

（3）**目录结构及文件分块信息****(元数据)**的管理由namenode节点承担

——namenode是HDFS集群主节点，负责维护整个hdfs文件系统的目录树，以及每一个路径（文件）所对应的block块信息（block的id，及所在的datanode服务器）

（4）文件的各个block的存储管理由datanode节点承担

---- datanode是HDFS集群从节点，每一个block都可以在多个datanode上存储多个副本（副本数量也可以通过参数设置dfs.replication）

（5）HDFS是设计成适应一次写入，多次读出的场景，且不支持文件的修改

*(注：适合用来做数据分析，并不适合用来做网盘应用，因为，不便修改，延迟大，网络开销大，成本太高)*

## 图解HDFS

通过上面的描述我们知道，hdfs很多特点：

　　保存多个副本，且提供容错机制，副本丢失或宕机自动恢复（默认存3份）。

　　运行在廉价的机器上

　　适合大数据的处理。HDFS默认会将文件分割成block，，在hadoop2.x以上版本默认128M为1个block。然后将block按键值对存储在HDFS上，并将键值对的映射存到内存中。如果小文件太多，那内存的负担会很重。

![img](https://images2018.cnblogs.com/blog/1228818/201803/1228818-20180308190320147-1328604927.png)

如上图所示，HDFS也是按照Master和Slave的结构。分NameNode、SecondaryNameNode、DataNode这几个角色。
　　　　NameNode：是Master节点，是大领导。管理数据块映射；处理客户端的读写请求；配置副本策略；管理HDFS的名称空间；
　　　　SecondaryNameNode：是一个小弟，分担大哥namenode的工作量；是NameNode的冷备份；合并fsimage和fsedits然后再发给namenode。
　　　　DataNode：Slave节点，奴隶，干活的。负责存储client发来的数据块block；执行数据块的读写操作。
　　　　热备份：b是a的热备份，如果a坏掉。那么b马上运行代替a的工作。
　　　　冷备份：b是a的冷备份，如果a坏掉。那么b不能马上代替a工作。但是b上存储a的一些信息，减少a坏掉之后的损失。
　　　　fsimage:元数据镜像文件（文件系统的目录树。）
　　　　edits：元数据的操作日志（针对文件系统做的修改操作记录）
　　　　namenode内存中存储的是=fsimage+edits。
　　　　SecondaryNameNode负责定时默认1小时，从namenode上，获取fsimage和edits来进行合并，然后再发送给namenode。减少namenode的工作量。

## HDFS的局限性

　　1）低延时数据访问。在用户交互性的应用中，应用需要在ms或者几个s的时间内得到响应。由于HDFS为高吞吐率做了设计，也因此牺牲了快速响应。对于低延时的应用，可以考虑使用HBase或者Cassandra。
　　2）大量的小文件。标准的HDFS数据块的大小是64M，存储小文件并不会浪费实际的存储空间，但是无疑会增加了在NameNode上的元数据，大量的小文件会影响整个集群的性能。

　　　　前面我们知道，Btrfs为小文件做了优化-inline file，对于小文件有很好的空间优化和访问时间优化。
　　3）多用户写入，修改文件。HDFS的文件只能有一个写入者，而且写操作只能在文件结尾以追加的方式进行。它不支持多个写入者，也不支持在文件写入后，对文件的任意位置的修改。
　　　　但是在大数据领域，分析的是已经存在的数据，这些数据一旦产生就不会修改，因此，HDFS的这些特性和设计局限也就很容易理解了。HDFS为大数据领域的数据分析，提供了非常重要而且十分基础的文件存储功能。

## HDFS保证可靠性的措施

　　1）冗余备份

　　　　每个文件存储成一系列数据块（Block）。为了容错，文件的所有数据块都会有副本（副本数量即复制因子，课配置）（dfs.replication）

　　2）副本存放

　　　　采用机架感知（Rak-aware）的策略来改进数据的可靠性、高可用和网络带宽的利用率

　　3）心跳检测

　　　　NameNode周期性地从集群中的每一个DataNode接受心跳包和块报告，收到心跳包说明该DataNode工作正常

　　4）安全模式

　　　　系统启动时，NameNode会进入一个安全模式。此时不会出现数据块的写操作。

　　5）数据完整性检测

　　　　HDFS客户端软件实现了对HDFS文件内容的校验和（Checksum）检查（dfs.bytes-per-checksum）。　



## 单点故障（单点失效）问题

### 单点故障问题

　　如果NameNode失效，那么客户端或MapReduce作业均无法读写查看文件

### 解决方案

　　1）启动一个拥有文件系统元数据的新NameNode（这个一般不采用，因为复制元数据非常耗时间）

　　2）配置一对活动-备用（Active-Sandby）NameNode，活动NameNode失效时，备用NameNode立即接管，用户不会有明显中断感觉。

　　　　共享编辑日志文件（借助NFS、zookeeper等）

　　　　DataNode同时向两个NameNode汇报数据块信息

　　　　客户端采用特定机制处理 NameNode失效问题，该机制对用户透明