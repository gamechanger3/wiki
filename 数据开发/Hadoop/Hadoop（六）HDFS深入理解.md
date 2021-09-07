<p align="center">
    <img width="280px" src="image/konglong/m3.png" >
</p>

# Hadoop（六）HDFS深入理解

##  HDFS的优点和缺点

### HDFS的优点

1、可构建在廉价机器上

　　　　通过多副本提高可靠性，提供了容错和恢复机制

　　　　服务器节点的宕机是常态  必须理性对象

2、高容错性

　　　　数据自动保存多个副本，副本丢失后，自动恢复

　　　　**HDFS的核心设计思想： 分散均匀存储 + 备份冗余存储**

3、适合批处理

　　　　移动计算而非数据，数据位置暴露给计算框架

　　　　海量数据的计算 任务 最终是一定要被切分成很多的小任务进行

4、适合大数据处理

　　　　GB、TB、甚至 PB 级数据，百万规模以上的文件数量，10K+节点规模

5、流式文件访问

　　　　 一次性写入，多次读取，保证数据一致性

### HDFS的缺点

#### 不适合以下操作

1、低延迟数据访问

　　　　比如毫秒级 低延迟与高吞吐率

2、小文件存取

　　　　占用 NameNode 大量内存 150b* 1000W = 15E,1.5G 寻道时间超过读取时间

3、并发写入、文件随机修改

　　　　一个文件只能有一个写者 仅支持 append

#### 抛出问题：HDFS文件系统为什么不适用于存储小文件？

这是和HDFS系统底层设计实现有关系的，HDFS本身的设计就是用来解决海量大文件数据的存储.，他天生喜欢大数据的处理，大文件存储在HDFS中，会被切分成很多的小数据块，任何一个文件不管有多小，都是一个独立的数据块，而这些数据块的信息则是保存在元数据中的，在之前的博客HDFS基础里面介绍过在HDFS集群的namenode中会存储元数据的信息，这里再说一下，元数据的信息主要包括以下3部分：

　　1）抽象目录树

　　2）文件和数据块的映射关系，一个数据块的元数据大小大约是150byte

　　3）数据块的多个副本存储地

而元数据的存储在磁盘（1和2）和内存中（1、2和3），而服务器中的内存是有上限的，举个例子：

有100个1M的文件存储进入HDFS系统，那么数据块的个数就是100个，元数据的大小就是100*150byte，消耗了15000byte的内存，但是只存储了100M的数据。

有1个100M的文件存储进入HDFS系统，那么数据块的个数就是1个，元数据的大小就是150byte，消耗量150byte的内存，存储量100M的数据。

所以说HDFS文件系统不适用于存储小文件。

## HDFS的辅助功能

HDFS作为一个文件系统。有两个最主要的功能：**上传和下载**。而为了保障这两个功能的完美和高效实现，HDFS提供了很多的辅助功能

### 1.心跳机制

#### 普通话讲解

1、 Hadoop 是 Master/Slave 结构，Master 中有 NameNode 和 ResourceManager，Slave 中有 Datanode 和 NodeManager 

2、 Master 启动的时候会启动一个 IPC（Inter-Process Comunication，进程间通信）server 服 务，等待 slave 的链接

3、 Slave 启动时，会主动链接 master 的 ipc server 服务，并且每隔 3 秒链接一次 master，这 个间隔时间是可以调整的，参数为 dfs.heartbeat.interval，这个每隔一段时间去连接一次 的机制，我们形象的称为心跳。Slave 通过心跳汇报自己的信息给 master，master 也通 过心跳给 slave 下达命令，

4、 NameNode 通过心跳得知 Datanode 的状态 ，ResourceManager 通过心跳得知 NodeManager 的状态

5、 如果 master 长时间都没有收到 slave 的心跳，就认为该 slave 挂掉了。！！！！！

#### 大白话讲解

1、DataNode启动的时候会向NameNode汇报信息，就像钉钉上班打卡一样，你打卡之后，你领导才知道你今天来上班了，同样的道理，DataNode也需要向NameNode进行汇报，只不过每次汇报的时间间隔有点短而已，默认是3秒中，**DataNode向NameNode汇报的信息有2点，一个是自身DataNode的状态信息，另一个是自身DataNode所持有的所有的数据块的信息。**而DataNode是不会知道他保存的所有的数据块副本到底是属于哪个文件，这些都是存储在NameNode的元数据中。

2、按照规定，每个DataNode都是需要向NameNode进行汇报。那么如果从某个时刻开始，某个DataNode再也不向NameNode进行汇报了。 有可能宕机了。因为只要通过网络传输数据，就一定存在一种可能： 丢失 或者 延迟。

3、HDFS的标准： NameNode如果连续10次没有收到DataNode的汇报。 那么NameNode就会认为该DataNode存在宕机的可能。

4、DataNode启动好了之后，会专门启动一个线程，去负责给NameNode发送心跳数据包，如果说整个DataNode没有任何问题，但是仅仅只是当前负责发送信条数据包的线程挂了。NameNode会发送命令向这个DataNode进行确认。查看这个发送心跳数据包的服务是否还能正常运行，而为了保险起见，NameNode会向DataNode确认2遍，每5分钟确认一次。如果2次都没有返回 结果，那么NameNode就会认为DataNode已经GameOver了！！！

**最终NameNode判断一个DataNode死亡的时间计算公式：**

**timeout = 10 \* 心跳间隔时间 + 2 \* 检查一次消耗的时间**

 心跳间隔时间：dfs.heartbeat.interval 心跳时间：3s
检查一次消耗的时间：heartbeat.recheck.interval checktime : 5min

最终结果默认是630s。

![img](https://images2018.cnblogs.com/blog/1228818/201803/1228818-20180309200331518-737230339.png)



### 2.安全模式

1、HDFS的启动和关闭都是先启动NameNode，在启动DataNode，最后在启动secondarynamenode。

![img](https://images2018.cnblogs.com/blog/1228818/201803/1228818-20180309200650648-965578498.png)

2、决定HDFS集群的启动时长会有两个因素：

　　1）磁盘元数据的大小

　　2）datanode的节点个数

 当元数据很大，或者 节点个数很多的时候，那么HDFS的启动，需要一段很长的时间，那么在还没有完全启动的时候HDFS能否对外提供服务？

在HDFS的启动命令start-dfs.sh执行的时候，HDFS会自动进入安全模式

为了确保用户的操作是可以高效的执行成功的，在HDFS发现自身不完整的时候，会进入安全模式。保护自己。

 ![img](https://images2018.cnblogs.com/blog/1228818/201803/1228818-20180309201610593-1303464002.png)

在正常启动之后，如果HDFS发现所有的数据都是齐全的，那么HDFS会启动的退出安全模式

![img](https://images2018.cnblogs.com/blog/1228818/201803/1228818-20180309201716199-1209493841.png)

3、对安全模式进行测试

安全模式常用操作命令：

```
hdfs dfsadmin -safemode leave //强制 NameNode 退出安全模式

hdfs dfsadmin -safemode enter //进入安全模式

hdfs dfsadmin -safemode get //查看安全模式状态

hdfs dfsadmin -safemode wait //等待，一直到安全模式结束
```

手工进入安全模式进行测试

1、测试创建文件夹

```
[hadoop@hadoop1 ~]$ hdfs dfsadmin -safemode enter
Safe mode is ON
[hadoop@hadoop1 ~]$ hadoop fs -mkdir -p /xx/yy/zz
mkdir: Cannot create directory /xx/yy/zz. Name node is in safe mode.
[hadoop@hadoop1 ~]$ 
```

![img](https://images2018.cnblogs.com/blog/1228818/201803/1228818-20180309202244027-371942508.png)

2、测试下载文件

```
[hadoop@hadoop1 ~]$ ls
apps  data
[hadoop@hadoop1 ~]$ hdfs dfsadmin -safemode get
Safe mode is ON
[hadoop@hadoop1 ~]$ hadoop fs -get /aa/1.txt ~/1.txt
[hadoop@hadoop1 ~]$ ls
1.txt  apps  data
[hadoop@hadoop1 ~]$ 
```

![img](https://images2018.cnblogs.com/blog/1228818/201803/1228818-20180309202543275-2048560291.png)

3、测试上传

```
[hadoop@hadoop1 ~]$ hadoop fs -put 1.txt /a/xx.txt
put: Cannot create file/a/xx.txt._COPYING_. Name node is in safe mode.
[hadoop@hadoop1 ~]$ 
```

![img](https://images2018.cnblogs.com/blog/1228818/201803/1228818-20180309202633279-1165284600.png)

4、得出结论，在安全模式下：

如果一个操作涉及到元数据的修改的话。都不能进行操作

如果一个操作仅仅只是查询。那是被允许的。

所谓的安全模式，仅仅只是保护namenode，而不是保护datanode

### 3.副本存放策略

第一副本：放置在上传文件的DataNode上；如果是集群外提交，则随机挑选一台磁盘不太慢、CPU不太忙的节点上；
第二副本：放置在于第一个副本不同的机架的节点上；
第三副本：与第二个副本相同机架的不同节点上；
如果还有更多的副本：随机放在节点中；

![img](https://images2018.cnblogs.com/blog/1228818/201803/1228818-20180309202954396-121440380.png)



### 4.负载均衡

负载均衡理想状态：节点均衡、机架均衡和磁盘均衡。

Hadoop的HDFS集群非常容易出现机器与机器之间磁盘利用率不平衡的情况，例如：当集群内新增、删除节点，或者某个节点机器内硬盘存储达到饱和值。当数据不平衡时，Map任务可能会分配到没有存储数据的机器，这将导致网络带宽的消耗，也无法很好的进行本地计算。
当HDFS负载不均衡时，需要对HDFS进行数据的负载均衡调整，即对各节点机器上数据的存储分布进行调整。从而，让数据均匀的分布在各个DataNode上，均衡IO性能，防止热点的发生。进行数据的负载均衡调整，必须要满足如下原则：

- - 数据平衡不能导致数据块减少，数据块备份丢失
  - 管理员可以中止数据平衡进程
  - 每次移动的数据量以及占用的网络资源，必须是可控的
  - 数据均衡过程，不能影响namenode的正常工作

#### 负载均衡的原理

数据均衡过程的核心是一个数据均衡算法，该数据均衡算法将不断迭代数据均衡逻辑，直至集群内数据均衡为止。该数据均衡算法每次迭代的逻辑如下：

![img](https://images2018.cnblogs.com/blog/1228818/201803/1228818-20180309203437807-414611294.png)

步骤分析如下：

1. 数据均衡服务（Rebalancing Server）首先要求 NameNode 生成 DataNode 数据分布分析报告,获取每个DataNode磁盘使用情况
2. Rebalancing Server汇总需要移动的数据分布情况，计算具体数据块迁移路线图。数据块迁移路线图，确保网络内最短路径
3. 开始数据块迁移任务，Proxy Source Data Node复制一块需要移动数据块
4. 将复制的数据块复制到目标DataNode上
5. 删除原始数据块
6. 目标DataNode向Proxy Source Data Node确认该数据块迁移完成
7. Proxy Source Data Node向Rebalancing Server确认本次数据块迁移完成。然后继续执行这个过程，直至集群达到数据均衡标准

#### **DataNode分组**

在第2步中，HDFS会把当前的DataNode节点,根据阈值的设定情况划分到Over、Above、Below、Under四个组中。在移动数据块的时候，Over组、Above组中的块向Below组、Under组移动。四个组定义如下：

 ![img](https://images2018.cnblogs.com/blog/1228818/201803/1228818-20180309203541810-527363557.png)

- **Over组**：此组中的DataNode的均满足

> DataNode_usedSpace_percent **>** Cluster_usedSpace_percent + threshold

- **Above组**：此组中的DataNode的均满足

> Cluster_usedSpace_percent + threshold **>** DataNode_ usedSpace _percent **>**Cluster_usedSpace_percent

- **Below组**：此组中的DataNode的均满足

> Cluster_usedSpace_percent **>** DataNode_ usedSpace_percent **>** Cluster_ usedSpace_percent – threshold

- **Under组**：此组中的DataNode的均满足

> Cluster_usedSpace_percent – threshold **>** DataNode_usedSpace_percent

#### Hadoop HDFS 数据自动平衡脚本使用方法

在Hadoop中，包含一个start-balancer.sh脚本，通过运行这个工具，启动HDFS数据均衡服务。该工具可以做到热插拔，即无须重启计算机和 Hadoop 服务。HadoopHome/bin目录下的start−balancer.sh脚本就是该任务的启动脚本。启动命令为：‘HadoopHome/bin目录下的start−balancer.sh脚本就是该任务的启动脚本。启动命令为：‘Hadoop_home/bin/start-balancer.sh –threshold`

**影响Balancer的几个参数：**

- -threshold
  - 默认设置：10，参数取值范围：0-100
  - 参数含义：判断集群是否平衡的阈值。理论上，该参数设置的越小，整个集群就越平衡
- dfs.balance.bandwidthPerSec
  - 默认设置：1048576（1M/S）
  - 参数含义：Balancer运行时允许占用的带宽

示例如下：

```
#启动数据均衡，默认阈值为 10%
$Hadoop_home/bin/start-balancer.sh

#启动数据均衡，阈值 5%
bin/start-balancer.sh –threshold 5

#停止数据均衡
$Hadoop_home/bin/stop-balancer.sh
```

在hdfs-site.xml文件中可以设置数据均衡占用的网络带宽限制

```
    <property>
    <name>dfs.balance.bandwidthPerSec</name>
    <value>1048576</value>
    <description> Specifies the maximum bandwidth that each datanode can utilize for the balancing purpose in term of the number of bytes per second. </description>
    </property>
```

