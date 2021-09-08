<p align="center">
    <img width="280px" src="image/konglong/m1.png" >
</p>

# Zookeeper（五）原理解析

## ZooKeeper中的各种角色

![img](https://images2018.cnblogs.com/blog/1228818/201803/1228818-20180323183446458-933789136.png)

## ZooKeeper与客户端

![img](https://images2018.cnblogs.com/blog/1228818/201803/1228818-20180323183652969-965453511.png)

　　每个Server在工作过程中有三种状态：
　　　　LOOKING：当前Server不知道leader是谁，正在搜寻
　　　　LEADING：当前Server即为选举出来的leader
　　　　FOLLOWING：leader已经选举出来，当前Server与之同步

## **Zookeeper节点数据操作流程**

![img](https://images2018.cnblogs.com/blog/1228818/201803/1228818-20180323184113970-414957166.png)

 

 

注：　　　　

　　1.在Client向Follwer发出一个写的请求

　　2.Follwer把请求发送给Leader

　　3.Leader接收到以后开始发起投票并通知Follwer进行投票

　　4.Follwer把投票结果发送给Leader

　　5.Leader将结果汇总后如果需要写入，则开始写入同时把写入操作通知给Leader，然后commit;

　　6.Follwer把请求结果返回给Client

 Follower主要有四个功能：

　　1. 向Leader发送请求（PING消息、REQUEST消息、ACK消息、REVALIDATE消息）；

　　2 .接收Leader消息并进行处理；

　　3 .接收Client的请求，如果为写请求，发送给Leader进行投票；

　　4 .返回Client结果。

Follower的消息循环处理如下几种来自Leader的消息：

　　1 .PING消息： 心跳消息；

　　2 .PROPOSAL消息：Leader发起的提案，要求Follower投票；

　　3 .COMMIT消息：服务器端最新一次提案的信息；

　　4 .UPTODATE消息：表明同步完成；

　　5 .REVALIDATE消息：根据Leader的REVALIDATE结果，关闭待revalidate的session还是允许其接受消息；

　　6 .SYNC消息：返回SYNC结果到客户端，这个消息最初由客户端发起，用来强制得到最新的更新。

## Paxos 算法概述（ZAB 协议） 

 　Paxos 算法是莱斯利•兰伯特（英语：Leslie Lamport）于 1990 年提出的一种基于消息传递且 具有高度容错特性的一致性算法。

　　分布式系统中的节点通信存在两种模型：**共享内存（Shared memory）和消息传递（Messages passing）**。基于消息传递通信模型的分布式系统，不可避免的会发生以下错误：进程可能会 慢、被杀死或者重启，消息可能会延迟、丢失、重复，在基础 Paxos 场景中，先不考虑可能 出现消息篡改即拜占庭错误（**Byzantine failure，即虽然有可能一个消息被传递了两次，但是 绝对不会出现错误的消息**）的情况。**Paxos 算法解决的问题是在一个可能发生上述异常的分 布式系统中如何就某个值达成一致，保证不论发生以上任何异常，都不会破坏决议一致性。**

　　Paxos 算法使用一个希腊故事来描述，在 Paxos 中，存在三种角色，分别为

　　　　**Proposer**（提议者，用来发出提案 proposal）

　　　　**Acceptor**（接受者，可以接受或拒绝提案）

　　　　**Learner**（学习者，学习被选定的提案，当提案被超过半数的 Acceptor 接受后为被批准）

　　下面更精确的定义 Paxos 要解决的问题：

　　　　1、决议(value)只有在被 proposer 提出后才能被批准

　　　　2、在一次 Paxos 算法的执行实例中，只批准(chose)一个 value

　　　　3、learner 只能获得被批准(chosen)的 value

　　ZooKeeper 的选举算法有两种：一种是基于 **Basic Paxos**（Google Chubby 采用）实现的，另外 一种是基于 **Fast Paxos**（ZooKeeper 采用）算法实现的。系统默认的选举算法为 Fast Paxos。 并且 ZooKeeper 在 3.4.0 版本后只保留了 FastLeaderElection 算法。

　　ZooKeeper 的核心是原子广播，这个机制保证了各个 Server 之间的同步。实现这个机制的协 议叫做 ZAB 协议（Zookeeper Atomic BrodCast）。 ZAB 协议有两种模式，它们分别是**崩溃恢复模式（选主）和原子广播模式（同步）**。

　　1、当服务启动或者在领导者崩溃后，ZAB 就进入了恢复模式，当领导者被选举出来，且大 多数 Server 完成了和 leader 的状态同步以后，恢复模式就结束了。状态同步保证了 leader 和 follower 之间具有相同的系统状态。

   2、当 ZooKeeper 集群选举出 leader 同步完状态退出恢复模式之后，便进入了原子广播模式。 所有的写请求都被转发给 leader，再由 leader 将更新 proposal 广播给 follower

 　为了保证事务的顺序一致性，zookeeper 采用了递增的事务 id 号（zxid）来标识事务。所有 的提议（proposal）都在被提出的时候加上了 zxid。实现中 zxid 是一个 64 位的数字，它高 32 位是 epoch 用来标识 leader 关系是否改变，每次一个 leader 被选出来，它都会有一个新 的 epoch，标识当前属于那个 leader 的统治时期。低 32 位用于递增计数。　　

 　这里给大家介绍以下 Basic Paxos 流程：

> 1、选举线程由当前 Server 发起选举的线程担任，其主要功能是对投票结果进行统计，并选 出推荐的 Server
>
> 2、选举线程首先向所有 Server 发起一次询问(包括自己)
>
> 3、选举线程收到回复后，验证是否是自己发起的询问(验证 zxid 是否一致)，然后获取对方 的 serverid(myid)，并存储到当前询问对象列表中，最后获取对方提议的 leader 相关信息 (serverid,zxid)，并将这些信息存储到当次选举的投票记录表中
>
> 4、收到所有 Server 回复以后，就计算出 id 最大的那个 Server，并将这个 Server 相关信息设 置成下一次要投票的 Server
>
> 5、线程将当前 id 最大的 Server 设置为当前 Server 要推荐的 Leader，如果此时获胜的 Server 获得 n/2 + 1 的 Server 票数， 设置当前推荐的 leader 为获胜的 Server，将根据获胜的 Server 相关信息设置自己的状态，否则，继续这个过程，直到 leader 被选举出来。

 　通过流程分析我们可以得出：要使 Leader 获得多数 Server 的支持，则 Server 总数必须是奇 数 2n+1，且存活的 Server 的数目不得少于 n+1。 每个 Server 启动后都会重复以上流程。在恢复模式下，如果是刚从崩溃状态恢复的或者刚 启动的 server 还会从磁盘快照中恢复数据和会话信息，zk 会记录事务日志并定期进行快照， 方便在恢复时进行状态恢复。 Fast Paxos 流程是在选举过程中，某 Server 首先向所有 Server 提议自己要成为 leader，当其 它 Server 收到提议以后，解决 epoch 和 zxid 的冲突，并接受对方的提议，然后向对方发送 接受提议完成的消息，重复这个流程，最后一定能选举出 Leader

## ZooKeeper 的选主机制

### 选择机制中的概念

#### 服务器ID

比如有三台服务器，编号分别是1,2,3。

> 编号越大在选择算法中的权重越大。

#### 数据ID

服务器中存放的最大数据ID.

>  值越大说明数据越新，在选举算法中数据越新权重越大。

#### 逻辑时钟

或者叫投票的次数，同一轮投票过程中的逻辑时钟值是相同的。每投完一次票这个数据就会增加，然后与接收到的其它服务器返回的投票信息中的数值相比，根据不同的值做出不同的判断。

#### 选举状态

- LOOKING，竞选状态。
- FOLLOWING，随从状态，同步leader状态，参与投票。
- OBSERVING，观察状态,同步leader状态，不参与投票。
- LEADING，领导者状态。

### 选举消息内容

在投票完成后，需要将投票信息发送给集群中的所有服务器，它包含如下内容。

- 服务器ID
- 数据ID
- 逻辑时钟
- 选举状态



### 选举流程图

因为每个服务器都是独立的，在启动时均从初始状态开始参与选举，下面是简易流程图。

 ![img](https://images2018.cnblogs.com/blog/1228818/201803/1228818-20180323185434643-1237203995.png)



### ZooKeeper 的全新集群选主

　　以一个简单的例子来说明整个选举的过程：假设有五台服务器组成的 zookeeper 集群，它们 的 serverid 从 1-5，同时它们都是最新启动的，也就是没有历史数据，在存放数据量这一点 上，都是一样的。假设这些服务器依序启动，来看看会发生什么

　　1、服务器 1 启动，此时只有它一台服务器启动了，它发出去的报没有任何响应，所以它的 选举状态一直是 LOOKING 状态

　　2、服务器 2 启动，它与最开始启动的服务器 1 进行通信，互相交换自己的选举结果，由于 两者都没有历史数据，所以 id 值较大的服务器 2 胜出，但是由于没有达到超过半数以上的服务器都同意选举它(这个例子中的半数以上是 3)，所以服务器 1、2 还是继续保持 LOOKING 状态

   3、服务器 3 启动，根据前面的理论分析，服务器 3 成为服务器 1,2,3 中的老大，而与上面不 同的是，此时有三台服务器(超过半数)选举了它，所以它成为了这次选举的 leader

　　4、服务器 4 启动，根据前面的分析，理论上服务器 4 应该是服务器 1,2,3,4 中最大的，但是 由于前面已经有半数以上的服务器选举了服务器 3，所以它只能接收当小弟的命了

　　5、服务器 5 启动，同 4 一样，当小弟

### ZooKeeper 的非全新集群选主

　　那么，初始化的时候，是按照上述的说明进行选举的，但是当 zookeeper 运行了一段时间之 后，有机器 down 掉，重新选举时，选举过程就相对复杂了。

　　需要加入数据 version、serverid 和逻辑时钟。

　　**数据 version**：数据新的 version 就大，数据每次更新都会更新 version

　　**server id**：就是我们配置的 myid 中的值，每个机器一个

　　**逻辑时钟**：这个值从 0 开始递增，每次选举对应一个值，也就是说：如果在同一次选举中， 那么这个值应该是一致的；逻辑时钟值越大，说明这一次选举 leader 的进程更新，也就是 每次选举拥有一个 zxid，投票结果只取 zxid 最新的

　　选举的标准就变成：

　　　　**1、逻辑时钟小的选举结果被忽略，重新投票**

　　　　**2、统一逻辑时钟后，数据 version 大的胜出**

　　　　**3、数据 version 相同的情况下，server id 大的胜出**

 根据这个规则选出 leader。