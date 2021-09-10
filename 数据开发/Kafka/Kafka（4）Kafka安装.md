<p align="center">
    <img width="280px" src="image/konglong/m14.png" >
</p>

# Kafka（4）Kafka安装

## 一、下载

下载地址：

http://kafka.apache.org/downloads.html

http://mirrors.hust.edu.cn/apache/

## 二、安装前提（zookeeper安装）

略略略

## 三、安装

此处使用版本为kafka_2.11-0.8.2.0.tgz

### 2.1　上传解压缩

```
[hadoop@hadoop1 ~]$ tar -zxvf kafka_2.11-0.8.2.0.tgz -C apps
[hadoop@hadoop1 ~]$ cd apps/
[hadoop@hadoop1 apps]$ ln -s kafka_2.11-0.8.2.0/ kafka
```

### 2.2　修改配置文件

进入kafka的安装配置目录

```
[hadoop@hadoop1 ~]$ cd apps/kafka/config/
```

主要关注：**server.properties** 这个文件即可，我们可以发现在目录下：

有很多文件，这里可以发现有Zookeeper文件，我们可以根据Kafka内带的zk集群来启动，但是建议使用独立的zk集群

![img](https://images2018.cnblogs.com/blog/1228818/201805/1228818-20180507205256986-826023478.png)

server.properties（**broker.id和host.name每个节点都不相同**）

```
//当前机器在集群中的唯一标识，和zookeeper的myid性质一样
broker.id=0
//当前kafka对外提供服务的端口默认是9092
port=9092
//这个参数默认是关闭的，在0.8.1有个bug，DNS解析问题，失败率的问题。
host.name=hadoop1
//这个是borker进行网络处理的线程数
num.network.threads=3
//这个是borker进行I/O处理的线程数
num.io.threads=8
//发送缓冲区buffer大小，数据不是一下子就发送的，先回存储到缓冲区了到达一定的大小后在发送，能提高性能
socket.send.buffer.bytes=102400
//kafka接收缓冲区大小，当数据到达一定大小后在序列化到磁盘
socket.receive.buffer.bytes=102400
//这个参数是向kafka请求消息或者向kafka发送消息的请请求的最大数，这个值不能超过java的堆栈大小
socket.request.max.bytes=104857600
//消息存放的目录，这个目录可以配置为“，”逗号分割的表达式，上面的num.io.threads要大于这个目录的个数这个目录，
//如果配置多个目录，新创建的topic他把消息持久化的地方是，当前以逗号分割的目录中，那个分区数最少就放那一个
log.dirs=/home/hadoop/log/kafka-logs
//默认的分区数，一个topic默认1个分区数
num.partitions=1
//每个数据目录用来日志恢复的线程数目
num.recovery.threads.per.data.dir=1
//默认消息的最大持久化时间，168小时，7天
log.retention.hours=168
//这个参数是：因为kafka的消息是以追加的形式落地到文件，当超过这个值的时候，kafka会新起一个文件
log.segment.bytes=1073741824
//每隔300000毫秒去检查上面配置的log失效时间
log.retention.check.interval.ms=300000
//是否启用log压缩，一般不用启用，启用的话可以提高性能
log.cleaner.enable=false
//设置zookeeper的连接端口
zookeeper.connect=192.168.123.102:2181,192.168.123.103:2181,192.168.123.104:2181
//设置zookeeper的连接超时时间
zookeeper.connection.timeout.ms=6000
```

producer.properties

```
metadata.broker.list=192.168.123.102:9092,192.168.123.103:9092,192.168.123.104:9092
```

consumer.properties

```
zookeeper.connect=192.168.123.102:2181,192.168.123.103:2181,192.168.123.104:2181
```

### 2.3　将kafka的安装包分发到其他节点

```
[hadoop@hadoop1 apps]$ scp -r kafka_2.11-0.8.2.0/ hadoop2:$PWD
[hadoop@hadoop1 apps]$ scp -r kafka_2.11-0.8.2.0/ hadoop3:$PWD
[hadoop@hadoop1 apps]$ scp -r kafka_2.11-0.8.2.0/ hadoop4:$PWD
```

### 2.4　创建软连接

```
[hadoop@hadoop1 apps]$ ln -s kafka_2.11-0.8.2.0/ kafka
```

### 2.5　修改环境变量

```
[hadoop@hadoop1 ~]$ vi .bashrc 
#Kafka
export KAFKA_HOME=/home/hadoop/apps/kafka
export PATH=$PATH:$KAFKA_HOME/bin
```

保存使其立即生效

```
[hadoop@hadoop1 ~]$ source ~/.bashrc
```

## 三、启动

### 3.1　首先启动zookeeper集群

所有zookeeper节点都需要执行

```
[hadoop@hadoop1 ~]$ zkServer.sh start
```

### 3.2　启动Kafka集群服务

```
[hadoop@hadoop1 kafka]$ bin/kafka-server-start.sh config/server.properties
```

hadoop1

![img](https://images2018.cnblogs.com/blog/1228818/201805/1228818-20180508091330478-1618680698.png)

Hadoop2

![img](https://images2018.cnblogs.com/blog/1228818/201805/1228818-20180508091357167-630027883.png)

hadoop3

![img](https://images2018.cnblogs.com/blog/1228818/201805/1228818-20180508091419194-1962654932.png)

hadoop4

![img](https://images2018.cnblogs.com/blog/1228818/201805/1228818-20180508091439631-71784584.png)

### 3.3　创建的topic

```
[hadoop@hadoop1 kafka]$ bin/kafka-topics.sh --create --zookeeper hadoop1:2181 --replication-factor 3 --partitions 3 --topic topic2
```

![img](https://images2018.cnblogs.com/blog/1228818/201805/1228818-20180508093450833-853043206.png)

### 3.4　查看topic副本信息

```
[hadoop@hadoop1 kafka]$ bin/kafka-topics.sh --describe --zookeeper hadoop1:2181 --topic topic2
```

![img](https://images2018.cnblogs.com/blog/1228818/201805/1228818-20180508093738851-372829720.png)

### 3.5　查看已经创建的topic信息

```
[hadoop@hadoop1 kafka]$ bin/kafka-topics.sh --list --zookeeper hadoop1:2181
```

![img](https://images2018.cnblogs.com/blog/1228818/201805/1228818-20180508094000763-519973668.png)

### 3.6　生产者发送消息

```
[hadoop@hadoop1 kafka]$ bin/kafka-console-producer.sh --broker-list hadoop1:9092 --topic topic2
```

![img](https://images2018.cnblogs.com/blog/1228818/201805/1228818-20180508094536515-251235516.png)

hadoop1显示接收到消息

### 3.7　消费者消费消息

在hadoop2上消费消息

```
[hadoop@hadoop2 kafka]$ bin/kafka-console-consumer.sh --zookeeper hadoop1:2181 --from-beginning --topic topic2
```

![img](https://images2018.cnblogs.com/blog/1228818/201805/1228818-20180508094916762-1544456891.png)