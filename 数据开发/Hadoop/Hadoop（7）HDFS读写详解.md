<p align="center">
    <img width="280px" src="image/konglong/m7.png" >
</p>

# Hadoop（七）HDFS读写详解

## HDFS的写操作



### 《HDFS权威指南》图解HDFS写过程

![img](https://images2018.cnblogs.com/blog/1228818/201803/1228818-20180312131601322-859729566.png)

​    ![img](https://images2018.cnblogs.com/blog/1228818/201803/1228818-20180312133050238-544764862.png)



### 详细文字说明（术语）

1、使用 HDFS 提供的客户端 Client，向远程的 namenode 发起 RPC 请求

2、namenode 会检查要创建的文件是否已经存在，创建者是否有权限进行操作，成功则会 为文件创建一个记录，否则会让客户端抛出异常；

3、当客户端开始写入文件的时候，客户端会将文件切分成多个 packets，并在内部以数据队列“data queue（数据队列）”的形式管理这些 packets，并向 namenode 申请 blocks，获 取用来存储 replicas 的合适的 datanode 列表，列表的大小根据 namenode 中 replication 的设定而定；

4、开始以 pipeline（管道）的形式将 packet 写入所有的 replicas 中。客户端把 packet 以流的 方式写入第一个 datanode，该 datanode 把该 packet 存储之后，再将其传递给在此 pipeline 中的下一个 datanode，直到最后一个 datanode，这种写数据的方式呈流水线的形式。

5、最后一个 datanode 成功存储之后会返回一个 ack packet（确认队列），在 pipeline 里传递 至客户端，在客户端的开发库内部维护着"ack queue"，成功收到 datanode 返回的 ack packet 后会从"data queue"移除相应的 packet。

6、如果传输过程中，有某个 datanode 出现了故障，那么当前的 pipeline 会被关闭，出现故 障的 datanode 会从当前的 pipeline 中移除，剩余的 block 会继续剩下的 datanode 中继续 以 pipeline 的形式传输，同时 namenode 会分配一个新的 datanode，保持 replicas 设定的 数量。

7、客户端完成数据的写入后，会对数据流调用 close()方法，关闭数据流；

8、只要写入了 dfs.replication.min（最小写入成功的副本数）的复本数（默认为 1），写操作 就会成功，并且这个块可以在集群中异步复制，直到达到其目标复本数（dfs.replication 的默认值为 3），因为 namenode 已经知道文件由哪些块组成，所以它在返回成功前只需 要等待数据块进行最小量的复制。



### 详细文字说明（口语）

1、客户端发起请求：hadoop fs -put hadoop.tar.gz /　

> **客户端怎么知道请求发给那个节点的哪个进程？**
>
> 因为客户端会提供一些工具来解析出来你所指定的HDFS集群的主节点是谁，以及端口号等信息，主要是通过URI来确定，
>
> url：hdfs://hadoop1:9000
>
> 当前请求会包含一个非常重要的信息： 上传的数据的总大小

2、namenode会响应客户端的这个请求

> **namenode的职责：**
>
> 1 管理元数据（抽象目录树结构）
>
> 用户上传的那个文件在对应的目录如果存在。那么HDFS集群应该作何处理，不会处理
>
> 用户上传的那个文件要存储的目录不存在的话，如果不存在不会创建
>
> 2、响应请求
>
> 真正的操作：做一系列的校验，
>
> 1、校验客户端的请求是否合理
> 2、校验客户端是否有权限进行上传

3、如果namenode返回给客户端的结果是 通过， 那就是允许上传

> namenode会给客户端返回对应的所有的数据块的多个副本的存放节点列表，如：
>
> file1_blk1 hadoop02，hadoop03，hadoop04
> file1_blk2 hadoop03，hadoop04，hadoop05

4、客户端在获取到了namenode返回回来的所有数据块的多个副本的存放地的数据之后，就可以按照顺序逐一进行数据块的上传操作

5、对要上传的数据块进行逻辑切片

> 切片分成两个阶段:
>
> 1、规划怎么切
> 2、真正的切
>
> 物理切片： 1 和 2
>
> 逻辑切片： 1
>
> file1_blk1 ： file1:0:128
> file1_blk2 ： file1:128:256

　　逻辑切片只是规划了怎么切

　　![img](https://images2018.cnblogs.com/blog/1228818/201803/1228818-20180312133938697-2120240662.png)

6、开始上传第一个数据块

7、客户端会做一系列准备操作

> 1、依次发送请求去连接对应的datnaode
>
> pipline : client - node1 - node2 - node3
>
> 按照一个个的数据包的形式进行发送的。
>
> 每次传输完一个数据包，每个副本节点都会进行校验，依次原路给客户端
>
> 2、在客户端会启动一个服务：
>
> 用户就是用来等到将来要在这个pipline数据管道上进行传输的数据包的校验信息
>
> 客户端就能知道当前从clinet到写node1,2,3三个节点上去的数据是否都写入正确和成功

8、clinet会正式的把这个快中的所有packet都写入到对应的副本节点

> 1、block是最大的一个单位，它是最终存储于DataNode上的数据粒度，由**dfs.block.size**参数决定，2.x版本默认是**128M**；注：这个参数由客户端配置决定；如：System.out.println(conf.get("dfs.blocksize"));//结果是134217728
>
> 2、packet是中等的一个单位，它是数据由DFSClient流向DataNode的粒度，以**dfs.write.packet.size**参数为参考值，默认是**64K**；注：这个参数为参考值，是指真正在进行数据传输时，会以它为基准进行调整，调整的原因是一个packet有特定的结构，调整的目标是这个packet的大小刚好包含结构中的所有成员，同时也保证写到DataNode后当前block的大小不超过设定值；
>
> 如：System.out.println(conf.get("dfs.write.packet.size"));//结果是65536
>
> 3、chunk是最小的一个单位，它是DFSClient到DataNode数据传输中进行数据校验的粒度，由io.bytes.per.checksum参数决定，默认是**512B**；注：事实上一个chunk还包含**4B的校验值**，因而chunk写入packet时是**516B**；数据与检验值的比值为128:1，所以对于一个128M的block会有一个1M的校验文件与之对应；
>
> 如：System.out.println(conf.get("io.bytes.per.checksum"));//结果是512

9、clinet进行校验，如果校验通过，表示该数据块写入成功

10、重复7 8 9 三个操作，来继续上传其他的数据块

11、客户端在意识到所有的数据块都写入成功之后，会给namenode发送一个反馈，就是告诉namenode当前客户端上传的数据已经成功。



##  HDFS读操作



### 《HDFS权威指南》图解HDFS读过程

![img](https://images2018.cnblogs.com/blog/1228818/201803/1228818-20180313104157141-1682640973.png)

![img](https://images2018.cnblogs.com/blog/1228818/201803/1228818-20180313104237694-682511475.png)



### 数据读取

1、客户端调用FileSystem 实例的open 方法，获得这个文件对应的输入流InputStream。

2、通过RPC 远程调用NameNode ，获得NameNode 中此文件对应的数据块保存位置，包括这个文件的副本的保存位置( 主要是各DataNode的地址) 。

3、获得输入流之后，客户端调用read 方法读取数据。选择最近的DataNode 建立连接并读取数据。

4、如果客户端和其中一个DataNode 位于同一机器(比如MapReduce 过程中的mapper 和reducer)，那么就会直接从本地读取数据。

5、到达数据块末端，关闭与这个DataNode 的连接，然后重新查找下一个数据块。

6、不断执行第2 - 5 步直到数据全部读完。

7、客户端调用close ，关闭输入流DF S InputStream。

## 问题



- **上传100M的文件，上传到50M，突然断了,HDFS会怎么处理？**

> 如果向DataNode写入数据失败了怎么办？
>
> 如果这种情况发生，那么就会执行一些操作：
>
> ① Pipeline数据流管道会被关闭，ACK queue中的packets会被添加到data queue的前面以确保不会发生packets数据包的丢失
>
> ② 在正常的DataNode节点上的以保存好的block的ID版本会升级——这样发生故障的DataNode节点上的block数据会在节点恢复**正常后被删除**，失效节点也会被从Pipeline中删除
> ③ 剩下的数据会被写入到Pipeline数据流管道中的其他两个节点中



- **Client 在写入过程中，有 DataNode 挂了**

> 当 Client 在写入过程中，有 DataNode 挂了。写入过程不会立刻终止（如果立刻终止，易用性和可用性都太不友好），取而代之 HDFS 尝试**从流水线中摘除挂了的 DataNode** 并恢复写入，这个过程称为 pipeline recovery

