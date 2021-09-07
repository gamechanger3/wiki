<p align="center">
    <img width="280px" src="image/konglong/m3.png" >
</p>
# Hadoop(三) HA 模式集群搭建

## 一 、 Hadoop 集群架构设计

![img](https://img2018.cnblogs.com/blog/1370918/201905/1370918-20190506212956918-1823961082.png)

## 二 、 搭建集群

修改IP地址与hostname以及部署zookeeper、hadoop见上一篇博文《Hadoop 完全分布式搭建》。

### zookeeper搭建

1、zookeeper集群搭建

a) 将zookeeper.tar.gz上传到node002、node003、node004

b) 解压到/opt

tar -zxf zookeeper-3.4.6.tar.gz -C /opt

c) 配置环境变量：

export ZOOKEEPER_PREFIX=/opt/zookeeper-3.4.6

export PATH=$PATH:$ZOOKEEPER_PREFIX/bin

然后. /etc/profile让配置生效

d) 到$ZOOKEEPER_PREFIX/conf下

复制zoo_sample.cfg为zoo.cfg执行该命令（因为zookeeper默认使用zoo.cfg文件）：cp zoo_sample.cfg zoo.cfg

e) 编辑zoo.cfg

添加如下行：

2881为选择端口（进行通信），3881位投票端口

server.1=node002:2881:3881

server.2=node003:2881:3881

server.3=node004:2881:3881

修改

dataDir=/var/bjsxt/zookeeper/data（可以自己定义自己的路径，没必要和我的一样）

f) 创建/var/bjsxt/zookeeper/data目录，并在该目录下放一个文件：myid

在myid中写下当前zookeeper的编号

在node004上操作一下命令：

mkdir -p /var/bjsxt/zookeeper/data

echo 3 > /var/bjsxt/zookeeper/data/myid

g) 将/opt/zookeeper-3.4.6通过网络拷贝到node002、node003上

scp -r zookeeper-3.4.6/ node002:/opt

scp -r zookeeper-3.4.6/ node003:/opt

h) 在node002和node003上分别创建/var/bjsxt/zookeeper/data目录，

并在该目录下放一个文件：myid

node002:

mkdir -p /var/bjsxt/zookeeper/data

echo 1 > /var/bjsxt/zookeeper/data/myid

node003:

mkdir -p /var/bjsxt/zookeeper/data

echo 2 > /var/bjsxt/zookeeper/data/myid

i) 启动zookeeper

[zkServer.sh](https://link.zhihu.com/?target=http%3A//zkServer.sh) start

[zkServer.sh](https://link.zhihu.com/?target=http%3A//zkServer.sh) start|stop|status

j) 关闭zookeeper

[zkServer.sh](https://link.zhihu.com/?target=http%3A//zkServer.sh) stop

l) 连接zookeeper

[zkCli.sh](https://link.zhihu.com/?target=http%3A//zkCli.sh)

m) 退出zkCli.sh命令

quit

zk启动脚本

```shell
#!/bin/bash

for host in hadoop01 hadoop02 hadoop03
do
echo $host
ssh $host "source /etc/profile; zkServer.sh start"
done
```

zk停止脚本

```shell
#!/bin/bash

for host in hadoop01 hadoop02 hadoop03
do
echo $host
ssh $host "source /etc/profile; zkServer.sh stop"
done
```



## 三 、修改配置文件

修改nna上的**core-site.xml**

```xml
<configuration>
<!-- 指定hdfs的nameservice为ns1 -->
    <property>
        <name>fs.defaultFS</name>
        <value>hdfs://ns1/</value>
    </property>
    <!-- 指定hadoop临时目录 -->
    <property>
        <name>hadoop.tmp.dir</name>
        <value>/home/zhihua/data</value>
    </property>

    <!-- 指定zookeeper地址 -->
    <property>
        <name>ha.zookeeper.quorum</name>
        <value>dn1:2181,dn2:2181,dn3:2181</value>
    </property>
    <property>
                <name>hadoop.proxyuser.zhihua.hosts</name>
                <value>*</value>
        </property>
    <property>
                <name>hadoop.proxyuser.zhihua.groups</name>
                <value>*</value>
        </property>
</configuration>
```

修改 **hdfs-site.xml** 

```xml
<configuration>
<!--指定hdfs的nameservice为ns1，需要和core-site.xml中的保持一致 -->
    <property>
        <name>dfs.nameservices</name>
        <value>ns1</value>
    </property>
    <!-- ns1下面有两个NameNode，分别是nna，nns -->
    <property>
        <name>dfs.ha.namenodes.ns1</name>
        <value>nna,nns</value>
    </property>
    <!-- nna的RPC通信地址 -->
    <property>
        <name>dfs.namenode.rpc-address.ns1.nna</name>
        <value>nna:9000</value>
    </property>
    <!-- nna的http通信地址 -->
    <property>
        <name>dfs.namenode.http-address.ns1.nna</name>
        <value>nna:50070</value>
    </property>
    <!-- nns的RPC通信地址 -->
    <property>
        <name>dfs.namenode.rpc-address.ns1.nns</name>
        <value>nns:9000</value>
    </property>
    <!-- nns的http通信地址 -->
    <property>
        <name>dfs.namenode.http-address.ns1.nns</name>
        <value>nns:50070</value>
    </property>
    <!-- 指定NameNode的元数据在JournalNode上的存放位置 -->
    <property>
        <name>dfs.namenode.shared.edits.dir</name>
        <value>qjournal://dn1:8485;dn2:8485;dn3:8485/ns1</value>
    </property>
    <!-- 指定JournalNode在本地磁盘存放数据的位置 -->
    <property>
        <name>dfs.journalnode.edits.dir</name>
        <value>/home/zhihua/data/journaldata</value>
    </property>
    <!-- 开启NameNode失败自动切换 -->
    <property>
        <name>dfs.ha.automatic-failover.enabled</name>
        <value>true</value>
    </property>
    <!-- 配置失败自动切换实现方式 -->
    <property>
        <name>dfs.client.failover.proxy.provider.ns1</name>
        <value>org.apache.hadoop.hdfs.server.namenode.ha.ConfiguredFailoverProxyProvider</value>
    </property>
    <!-- 配置隔离机制方法，多个机制用换行分割，即每个机制暂用一行-->
    <property>
        <name>dfs.ha.fencing.methods</name>
        <value>
            sshfence
            shell(/bin/true)
        </value>
    </property>

    <!-- 配置sshfence隔离机制超时时间 -->
    <property>
        <name>dfs.ha.fencing.ssh.connect-timeout</name>
        <value>30000</value>
    </property>
    <property>
                <name>dfs.webhdfs.enabled</name>
                <value>true</value>
        </property>

</configuration>
```

修改 **mapred-site.xml**

```xml
<configuration>
<!-- 指定mr框架为yarn方式 -->
    <property>
        <name>mapreduce.framework.name</name>
        <value>yarn</value>
    </property>
</configuration>
```

修改 **yarn-site.xml**

```xml
<configuration>

<!-- Site specific YARN configuration properties -->
<!-- 开启RM高可用 -->
        <property>
           <name>yarn.resourcemanager.ha.enabled</name>
           <value>true</value>
        </property>
        <!-- 指定RM的cluster id -->
        <property>
           <name>yarn.resourcemanager.cluster-id</name>
           <value>cluster_id</value>
        </property>
        <!-- 指定RM的名字 -->
        <property>
           <name>yarn.resourcemanager.ha.rm-ids</name>
           <value>rm1,rm2</value>
        </property>
        <!-- 分别指定RM的地址 -->
        <property>
           <name>yarn.resourcemanager.hostname.rm1</name>
           <value>nna</value>
        </property>
        <property>
           <name>yarn.resourcemanager.hostname.rm2</name>
           <value>nns</value>
        </property>
        <property>
        <name>yarn.resourcemanager.webapp.address.rm1</name>
        <value>nna:8088</value>
        </property>
        <property>
        <name>yarn.resourcemanager.webapp.address.rm2</name>
        <value>nns:8088</value>
        </property>
        <!-- 指定zk集群地址 -->
        <property>
           <name>yarn.resourcemanager.zk-address</name>
           <value>dn1:2181,dn2:2181,dn3:2181</value>
        </property>
        <property>
           <name>yarn.nodemanager.aux-services</name>
           <value>mapreduce_shuffle</value>
        </property>
</configuration>
```

分发配置文件

```
1 [zhihua@nna /soft/hadoop/etc]$scp -r hadoop zhihua@nns:/soft/hadoop/etc/
2 [zhihua@nna /soft/hadoop/etc]$scp -r hadoop zhihua@dn1:/soft/hadoop/etc/
3 [zhihua@nna /soft/hadoop/etc]$scp -r hadoop zhihua@dn2:/soft/hadoop/etc/
4 [zhihua@nna /soft/hadoop/etc]$scp -r hadoop zhihua@dn3:/soft/hadoop/etc/
```

## 四 、启动集群

4.1 格式化 Hadoop 集群

```
1 [zhihua@nna /soft/hadoop/etc]$hadoop namenode -format 
```

4.2 复制 Hadoop 临时目录到nns上

```
1 [zhihua@nna /home/zhihua]$scp -r data zhihua@nns:/home/zhihua/
```

4.3 启动zookeeper

```
1 [zhihua@dn1 /soft/zk/bin]$./zkServer.sh start 
2 [zhihua@dn2 /soft/zk/bin]$./zkServer.sh  start 
3 [zhihua@dn3 /soft/zk/bin]$./zkServer.sh  start 
```

4.4 启动 JournalNode 

```
1 [zhihua@nna /home/zhihua]$hadoop-daemons.sh start journalnode 
```

4.5 启动 HDFS 与 YARN 

```
1 [zhihua@nna /home/zhihua]$start-dfs.sh 
2 [zhihua@nna /home/zhihua]$start-yarn.sh 
```

4.6 启动 NNS上的ResourceManager

```
1 [zhihua@nns /soft/hadoop/etc/hadoop]$yarn-daemon.sh start resourcemanager
```

## 五 、 测试集群是否部署成功

检查nna上面的进程

![img](https://img2018.cnblogs.com/blog/1370918/201905/1370918-20190506220508855-1276740813.png)

检查nns 上面的进程

![img](https://img2018.cnblogs.com/blog/1370918/201905/1370918-20190506220610140-1579058444.png)

检查dn1-dn3上的进程

![img](https://img2018.cnblogs.com/blog/1370918/201905/1370918-20190506220642150-2053995536.png)

webUI检查集群是否正常

![img](https://img2018.cnblogs.com/blog/1370918/201905/1370918-20190506220333369-1078442356.png)

 

 ![img](https://img2018.cnblogs.com/blog/1370918/201905/1370918-20190506220403676-1866260183.png)

 