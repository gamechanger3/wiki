<p align="center">
    <img width="280px" src="image/konglong/m3.png" >
</p>

# Zookeeper（三）shell操作

## Zookeeper的shell操作

### Zookeeper命令工具

在启动Zookeeper服务之后，输入以下命令，连接到Zookeeper服务：

```
[hadoop@hadoop1 ~]$ zkCli.sh -server hadoop2:2181
```

```
 1 [hadoop@hadoop1 ~]$ zkCli.sh -server hadoop2:2181
 2 Connecting to hadoop2:2181
 3 2018-03-21 19:55:53,744 [myid:] - INFO  [main:Environment@100] - Client environment:zookeeper.version=3.4.10-39d3a4f269333c922ed3db283be479f9deacaa0f, built on 03/23/2017 10:13 GMT
 4 2018-03-21 19:55:53,748 [myid:] - INFO  [main:Environment@100] - Client environment:host.name=hadoop1
 5 2018-03-21 19:55:53,749 [myid:] - INFO  [main:Environment@100] - Client environment:java.version=1.8.0_73
 6 2018-03-21 19:55:53,751 [myid:] - INFO  [main:Environment@100] - Client environment:java.vendor=Oracle Corporation
 7 2018-03-21 19:55:53,751 [myid:] - INFO  [main:Environment@100] - Client environment:java.home=/usr/local/jdk1.8.0_73/jre
 8 2018-03-21 19:55:53,751 [myid:] - INFO  [main:Environment@100] - Client environment:java.class.path=/home/hadoop/apps/zookeeper-3.4.10/bin/../build/classes:/home/hadoop/apps/zookeeper-3.4.10/bin/../build/lib/*.jar:/home/hadoop/apps/zookeeper-3.4.10/bin/../lib/slf4j-log4j12-1.6.1.jar:/home/hadoop/apps/zookeeper-3.4.10/bin/../lib/slf4j-api-1.6.1.jar:/home/hadoop/apps/zookeeper-3.4.10/bin/../lib/netty-3.10.5.Final.jar:/home/hadoop/apps/zookeeper-3.4.10/bin/../lib/log4j-1.2.16.jar:/home/hadoop/apps/zookeeper-3.4.10/bin/../lib/jline-0.9.94.jar:/home/hadoop/apps/zookeeper-3.4.10/bin/../zookeeper-3.4.10.jar:/home/hadoop/apps/zookeeper-3.4.10/bin/../src/java/lib/*.jar:/home/hadoop/apps/zookeeper-3.4.10/bin/../conf::/usr/local/jdk1.8.0_73/lib:/usr/local/jdk1.8.0_73/jre/lib
 9 2018-03-21 19:55:53,751 [myid:] - INFO  [main:Environment@100] - Client environment:java.library.path=/usr/java/packages/lib/amd64:/usr/lib64:/lib64:/lib:/usr/lib
10 2018-03-21 19:55:53,751 [myid:] - INFO  [main:Environment@100] - Client environment:java.io.tmpdir=/tmp
11 2018-03-21 19:55:53,751 [myid:] - INFO  [main:Environment@100] - Client environment:java.compiler=<NA>
12 2018-03-21 19:55:53,752 [myid:] - INFO  [main:Environment@100] - Client environment:os.name=Linux
13 2018-03-21 19:55:53,752 [myid:] - INFO  [main:Environment@100] - Client environment:os.arch=amd64
14 2018-03-21 19:55:53,752 [myid:] - INFO  [main:Environment@100] - Client environment:os.version=2.6.32-573.el6.x86_64
15 2018-03-21 19:55:53,752 [myid:] - INFO  [main:Environment@100] - Client environment:user.name=hadoop
16 2018-03-21 19:55:53,752 [myid:] - INFO  [main:Environment@100] - Client environment:user.home=/home/hadoop
17 2018-03-21 19:55:53,752 [myid:] - INFO  [main:Environment@100] - Client environment:user.dir=/home/hadoop
18 2018-03-21 19:55:53,755 [myid:] - INFO  [main:ZooKeeper@438] - Initiating client connection, connectString=hadoop2:2181 sessionTimeout=30000 watcher=org.apache.zookeeper.ZooKeeperMain$MyWatcher@5c29bfd
19 Welcome to ZooKeeper!
20 2018-03-21 19:55:53,789 [myid:] - INFO  [main-SendThread(hadoop2:2181):ClientCnxn$SendThread@1032] - Opening socket connection to server hadoop2/192.168.123.103:2181. Will not attempt to authenticate using SASL (unknown error)
21 JLine support is enabled
22 2018-03-21 19:55:53,931 [myid:] - INFO  [main-SendThread(hadoop2:2181):ClientCnxn$SendThread@876] - Socket connection established to hadoop2/192.168.123.103:2181, initiating session
23 2018-03-21 19:55:53,977 [myid:] - INFO  [main-SendThread(hadoop2:2181):ClientCnxn$SendThread@1299] - Session establishment complete on server hadoop2/192.168.123.103:2181, sessionid = 0x262486284b70000, negotiated timeout = 30000
24 
25 WATCHER::
26 
27 WatchedEvent state:SyncConnected type:None path:null
28 [zk: hadoop2:2181(CONNECTED) 0] 
```

连接成功之后，系统会输出Zookeeper的相关环境及配置信息，并在屏幕输出“welcome to Zookeeper！”等信息。

![img](https://images2018.cnblogs.com/blog/1228818/201803/1228818-20180321195759304-642290839.png)

输入help之后，屏幕会输出可用的Zookeeper命令，如下图所示

![img](https://images2018.cnblogs.com/blog/1228818/201803/1228818-20180321195909767-971823545.png)

### 使用Zookeeper命令的简单操作步骤

(1) 使用ls命令查看当前Zookeeper中所包含的内容：ls /

```
[zk: hadoop2:2181(CONNECTED) 1] ls /
[zookeeper]
[zk: hadoop2:2181(CONNECTED) 2] 
```

(2) 创建一个新的Znode节点"aa"，以及和它相关字符，执行命令：create /aa "my first zk"，默认是不带编号的

```
[zk: hadoop2:2181(CONNECTED) 2] create /aa "my first zk"
Created /aa
[zk: hadoop2:2181(CONNECTED) 3] 
```

　　创建带编号的持久性节点"bb"，

```
[zk: localhost:2181(CONNECTED) 1] create -s /bb "bb"
Created /bb0000000001
[zk: localhost:2181(CONNECTED) 2] 
```

　　创建不带编号的临时节点"cc"

```
[zk: localhost:2181(CONNECTED) 2] create -e /cc "cc"
Created /cc
[zk: localhost:2181(CONNECTED) 3] 
```

　　创建带编号的临时节点"dd"

```
[zk: localhost:2181(CONNECTED) 3] create -s -e /dd "dd"
Created /dd0000000003
[zk: localhost:2181(CONNECTED) 4] 
```

(3) 再次使用ls命令来查看现在Zookeeper的中所包含的内容：ls /

[zk: localhost:2181(CONNECTED) 4] ls /
[cc, dd0000000003, zookeeper, bb0000000001]
[zk: localhost:2181(CONNECTED) 5]

此时看到，aa节点已经被创建。 

关闭本次连接回话session，再重新打开一个连接

```
[zk: localhost:2181(CONNECTED) 5] close
2018-03-22 13:03:29,137 [myid:] - INFO  [main:ZooKeeper@684] - Session: 0x1624c10e8d90000 closed
[zk: localhost:2181(CLOSED) 6] 2018-03-22 13:03:29,139 [myid:] - INFO  [main-EventThread:ClientCnxn$EventThread@519] - EventThread shut down for session: 0x1624c10e8d90000

[zk: localhost:2181(CLOSED) 6] ls /
Not connected
[zk: localhost:2181(CLOSED) 7] connect hadoop1:2181
```

重新查看，临时节点已经随着上一次的会话关闭自动删除了

```
[zk: hadoop1:2181(CONNECTED) 8] ls /
[zookeeper, bb0000000001]
[zk: hadoop1:2181(CONNECTED) 9] 
```

(4) 使用get命令来确认第二步中所创建的Znode是否包含我们创建的字符串，执行命令：get /aa

```
[zk: hadoop2:2181(CONNECTED) 4] get /aa
my first zk
cZxid = 0x100000002
ctime = Wed Mar 21 20:01:02 CST 2018
mZxid = 0x100000002
mtime = Wed Mar 21 20:01:02 CST 2018
pZxid = 0x100000002
cversion = 0
dataVersion = 0
aclVersion = 0
ephemeralOwner = 0x0
dataLength = 11
numChildren = 0
[zk: hadoop2:2181(CONNECTED) 5] 
```

(5) 接下来通过set命令来对zk所关联的字符串进行设置，执行命令：set /aa haha123

```
[zk: hadoop2:2181(CONNECTED) 6] set /aa haha123 
cZxid = 0x100000002
ctime = Wed Mar 21 20:01:02 CST 2018
mZxid = 0x100000004
mtime = Wed Mar 21 20:04:10 CST 2018
pZxid = 0x100000002
cversion = 0
dataVersion = 1
aclVersion = 0
ephemeralOwner = 0x0
dataLength = 7
numChildren = 0
[zk: hadoop2:2181(CONNECTED) 7] 
```

(6) 再次使用get命令来查看，上次修改的内容，执行命令：get /aa

```
[zk: hadoop2:2181(CONNECTED) 7] get /aa
haha123
cZxid = 0x100000002
ctime = Wed Mar 21 20:01:02 CST 2018
mZxid = 0x100000004
mtime = Wed Mar 21 20:04:10 CST 2018
pZxid = 0x100000002
cversion = 0
dataVersion = 1
aclVersion = 0
ephemeralOwner = 0x0
dataLength = 7
numChildren = 0
[zk: hadoop2:2181(CONNECTED) 8] 
```

(7) 下面我们将刚才创建的Znode删除，执行命令：delete /aa

```
[zk: hadoop2:2181(CONNECTED) 8] delete /aa
[zk: hadoop2:2181(CONNECTED) 9] 
```

(8) 最后再次使用ls命令查看Zookeeper中的内容，执行命令：ls /

```
[zk: hadoop2:2181(CONNECTED) 9] ls /
[zookeeper]
[zk: hadoop2:2181(CONNECTED) 10] 
```

经过验证，zk节点已经删除。

(9) 退出，执行命令：quit

```
[zk: hadoop2:2181(CONNECTED) 10] quit
Quitting...
2018-03-21 20:07:11,133 [myid:] - INFO  [main:ZooKeeper@684] - Session: 0x262486284b70000 closed
2018-03-21 20:07:11,139 [myid:] - INFO  [main-EventThread:ClientCnxn$EventThread@519] - EventThread shut down for session: 0x262486284b70000
[hadoop@hadoop1 ~]$ 
```

### 状态信息

查看一个文件的状态信息

```
[zk: localhost:2181(CONNECTED) 1] stat /a
cZxid = 0x200000009
ctime = Thu Mar 22 13:07:19 CST 2018
mZxid = 0x200000009
mtime = Thu Mar 22 13:07:19 CST 2018
pZxid = 0x200000009
cversion = 0
dataVersion = 0
aclVersion = 0
ephemeralOwner = 0x0
dataLength = 1
numChildren = 0
[zk: localhost:2181(CONNECTED) 2] 
```

详细解释：

zxid： 一个事务编号，zookeeper集群内部的所有事务，都有一个全局的唯一的顺序的编号

　　它由两部分组成： 就是一个 64位的长整型 long

　　**高32位: 用来标识leader关系是否改变，如  0x2**　　

　　**低32位： 用来做当前这个leader领导期间的全局的递增的事务编号，如  00000009**

| **状态属性**   | **说明**                                                     |
| -------------- | ------------------------------------------------------------ |
| cZxid          | 数据节点创建时的事务ID                                       |
| ctime          | 数据节点创建时的时间                                         |
| mZxid          | 数据节点最后一次更新时的事务ID                               |
| mtime          | 数据节点最后一次更新时的时间                                 |
| pZxid          | 数据节点的子节点列表最后一次被修改（是子节点列表变更，而不是子节点内容变更）时的事务ID |
| cversion       | 子节点的版本号                                               |
| dataVersion    | 数据节点的版本号                                             |
| aclVersion     | 数据节点的ACL版本号                                          |
| ephemeralOwner | 如果节点是临时节点，则表示创建该节点的会话的SessionID；如果节点是持久节点，则该属性值为0 |
| dataLength     | 数据内容的长度                                               |
| numChildren    | 数据节点当前的子节点个数                                     |

（1）修改节点a的数据，**mZxid、** **dataVersion、** 

**dataLength 存储信息发生变化**

```
[zk: localhost:2181(CONNECTED) 2] set /a 'aaa'
cZxid = 0x200000009
ctime = Thu Mar 22 13:07:19 CST 2018
mZxid = 0x20000000a
mtime = Thu Mar 22 13:12:53 CST 2018
pZxid = 0x200000009
cversion = 0
dataVersion = 1
aclVersion = 0
ephemeralOwner = 0x0
dataLength = 3
numChildren = 0
[zk: localhost:2181(CONNECTED) 3] 
```

（2）创建新的节点b，状态信息都会发生变化，zxid的事物ID也会增加

```
[zk: localhost:2181(CONNECTED) 5] stat /b
cZxid = 0x20000000b
ctime = Thu Mar 22 13:15:56 CST 2018
mZxid = 0x20000000b
mtime = Thu Mar 22 13:15:56 CST 2018
pZxid = 0x20000000b
cversion = 0
dataVersion = 0
aclVersion = 0
ephemeralOwner = 0x0
dataLength = 1
numChildren = 0
[zk: localhost:2181(CONNECTED) 6]  
```

（3）在a节点下面新增节点c，**pZxid、** **cversion** **numChildren 发生改变**

```
[zk: localhost:2181(CONNECTED) 6] create /a/c 'c'
Created /a/c
[zk: localhost:2181(CONNECTED) 7] stat /a
cZxid = 0x200000009
ctime = Thu Mar 22 13:07:19 CST 2018
mZxid = 0x20000000a
mtime = Thu Mar 22 13:12:53 CST 2018
pZxid = 0x20000000c
cversion = 1
dataVersion = 1
aclVersion = 0
ephemeralOwner = 0x0
dataLength = 3
numChildren = 1
[zk: localhost:2181(CONNECTED) 8] 
```

（4）ephemeralOwner 持久性的节点信息是0x0临时的几点信息是本次会话的sessionid，如图

![img](https://images2018.cnblogs.com/blog/1228818/201803/1228818-20180322132219641-524856760.png)

(5) 将leader干掉，此时第二台机器成为leader，重新创建一个文件y，此时发现czxid的前3位和之前发生变化，说明换了leader

![img](https://images2018.cnblogs.com/blog/1228818/201803/1228818-20180322132450294-640720361.png)

![img](https://images2018.cnblogs.com/blog/1228818/201803/1228818-20180322132507158-644407845.png)

```
[zk: localhost:2181(CONNECTED) 0] create /y 'yy'
Created /y
[zk: localhost:2181(CONNECTED) 2] stat /y
cZxid = 0x300000003
ctime = Thu Mar 22 13:25:51 CST 2018
mZxid = 0x300000003
mtime = Thu Mar 22 13:25:51 CST 2018
pZxid = 0x300000003
cversion = 0
dataVersion = 0
aclVersion = 0
ephemeralOwner = 0x0
dataLength = 2
numChildren = 0
[zk: localhost:2181(CONNECTED) 3] 
```



