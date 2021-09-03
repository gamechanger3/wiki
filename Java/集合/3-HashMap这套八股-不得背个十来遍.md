# HashMap 这套八股，不得背个十来遍？

---

HashMap、HashTable、ConcurrentHashMap 这一套感觉今年面试都不怎么问了，场景题越来越多，求职的门槛越来越高，这种常见的面试题问出来大概率就是要送波分了。

### 1. 讲讲 HashMap 的底层结构和原理

HashMap 就是以 Key-Value 的方式进行数据存储的一种数据结构嘛，在我们平常开发中非常常用，它在 JDK 1.7 和 JDK 1.8 中底层数据结构是有些不一样的。总体来说，JDK 1.7 中 HashMap 的底层数据结构是数组 + 链表，使用 `Entry` 类存储 Key 和 Value；JDK 1.8 中 HashMap 的底层数据结构是数组 + 链表/红黑树，使用 `Node` 类存储 Key 和 Value。当然，这里的 `Entry` 和 `Node` 并没有什么不同，我们来看看 `Node` 类的源码：

```java
// HashMap 1.8 内部使用这个数组存储所有键值对
transient Node<K,V>[] table;
```

![](https://gitee.com/veal98/images/raw/master/img/20210327225137.png)

每一个节点都会保存自身的 hash、key 和 value、以及下个节点。咱先画个简略的 HashMap 示意图：

![](https://gitee.com/veal98/images/raw/master/img/20210327234553.png)

因为 HashMap 本身所有的位置都为 null 嘛，所以在插入元素的时候即 `put` 操作时，会根据 key 的 hash 去计算出一个 index 值，也就是这个元素将要插入的位置。

举个例子：比如 `put("小牛肉"，20)`，我插入了一个 key 为 "小牛肉" value 为 20 的元素，这个时候我们会通过哈希函数计算出这个元素将要插入的位置，假设计算出来的结果是 2：

![](https://gitee.com/veal98/images/raw/master/img/20210327234630.png)

我们刚刚还提到了链表，Node 类里面也确实定义了一个 `next` 属性，那么为啥需要链表呢？

首先，数组的长度是有限的对吧，在有限的数组上使用哈希，那么哈希冲突是不可避免的，很有可能两个元素计算得出的 index 是相同的，那么如何解决哈希冲突呢？**拉链法**。也就是把 hash 后值相同的元素放在同一条链表上。比如说：

![](https://gitee.com/veal98/images/raw/master/img/20210327235235.png)

当然这里还有一个问题，那就是当 Hash 冲突严重时，在数组上形成的链表会变的越来越长，由于链表不支持索引，要想在链表中找一个元素就需要遍历一遍链表，那显然效率是比较低的。为此，JDK 1.8 引入了红黑树，**当链表的长度大于 8 的时候就会转换为红黑树，不过，在转换之前，会先去查看 table 数组的长度是否大于 64，如果数组的长度小于 64，那么 HashMap 会优先选择对数组进行扩容 `resize`，而不是把链表转换成红黑树**。

![](https://gitee.com/veal98/images/raw/master/img/20210327232720.png)

看下 JDK 1.8 下 HashMap 的完整示意图，应该画的比较清晰了：

![](https://gitee.com/veal98/images/raw/master/img/20210327234953.png)

### 2. 新的 Entry/Node 节点在插入链表的时候，是怎么插入的？

在 JDK 1.7 的时候，采用的是头插法，看下图：

![](https://gitee.com/veal98/images/raw/master/img/20210327235519.png)

不过 JDK 1.8 改成了尾插法，这是为什么呢？因为 **JDK 1.7 中采用的头插法在多线程环境下可能会造成循环链表问题**。

首先，我们之前提到，数组容量是有限的，如果数据多次插入并到达一定的数量就会进行数组扩容，也就是`resize` 方法。什么时候会进行 `resize` 呢？与两个因素有关：

1）`Capacity`：HashMap 当前最大容量/长度

2）`LoadFactor`：负载因子，默认值0.75f

![](https://gitee.com/veal98/images/raw/master/img/20210328003223.png)

如果当前存入的数据数量大于 Capacity * LoadFactor 的时候，就会进行数组扩容 `resize`。就比如当前的 HashMap 的最大容量大小为 100，当你存进第 76 个的时候，判断发现需要进行 resize了，那就进行扩容。当然，HashMap 的扩容不是简单的扩大点容量这么简单的。

**扩容 `resize` 分为两步**：

1）扩容：创建一个新的 Entry/Node 空数组，长度是原数组的 **2 倍**

2）ReHash：遍历原 Entry/Node 数组，把所有的 Entry/Node 节点重新 Hash 到新数组

为什么要 ReHash 呢？直接复制到新数组不行吗？

显然是不行的，因为数组的长度改变以后，Hash 的规则也随之改变。index 的计算公式是这样的：

- index = HashCode(key) & (Length - 1)

比如说数组原来的长度（Length）是 4，Hash 出来的值是 2 ，然后数组长度翻倍了变成 16，显然 Hash 出来的值也就会变了。画个图解释下：

![](https://gitee.com/veal98/images/raw/master/img/20210328004757.png)

OK，说完扩容机制我们言归正传，为啥 JDK 1.7 使用头插法，JDK 1.8 之后改成尾插法了呢？

我们来看 1.7 的 resize 方法：

![](https://gitee.com/veal98/images/raw/master/img/20210328113533.png)

newTable 就是扩容后的新数组，`transfer` 方法是 `resize` 的核心，它的的功能就是 ReHash，然后将原数组中的数据迁移到新数据。我们先来把 transfer 代码简化一下，方便下文的理解：

![](https://gitee.com/veal98/images/raw/master/img/20210328120118.png)

先来看看单线程情况下，正常的 resize 的过程。假设我们原来的数组容量为 2，记录数为 3，分别为：[3,A]、[7,B]、[5,C]，并且这三个 Entry 节点都落到了第二个桶里面，新数组容量会被扩容到 4。

> 下面的图画的可能会有点不严谨，不过能够方便大家理解其中意思就好

![](https://gitee.com/veal98/images/raw/master/img/20210328121026.png)

OK，那现在如果我们有两个线程 Thread1 和 Thread2，假设线程 Thread1 执行到了 transfer 方法的 `Entry next = e.next` 这一句，然后时间片用完了被挂起了。随后线程 Thread2 顺利执行并完成 resize 方法。于是我们有下面这个样子：

![](https://gitee.com/veal98/images/raw/master/img/20210328122358.png)

注意，Thread1 的 e 指向了 [3,A]，next 指向了 [7,B]，**而在线程 Thread2 进行 ReHash后，e 和 next 指向了线程 Thread2 重组后的链表**。我们可以看到链表的顺序被反转了。

OK，这个时候线程 Thread1 被重新调度执行，先是执行 `newTalbe[i] = e`，i 就是 ReHash 后的 index 值：

![](https://gitee.com/veal98/images/raw/master/img/20210328123217.png)

然后执行 `e = next`，导致了 e 指向了 [7,B]，而下一次循环执行到 `next = e.next` 时导致了 next 指向了 [3,A]

![](https://gitee.com/veal98/images/raw/master/img/20210328123241.png)

然后，线程 Thread1 继续执行。把旧数组的 [7,B] 摘下来，放到 newTable[i] 的第一个，然后把 e 和 next 往下顺移：

![](https://gitee.com/veal98/images/raw/master/img/20210328123321.png)

OK，Thread1 再进入下一步循环，执行到 `e.next = newTable[i] `，导致 [3,A].next 指向了 [7,B]，循环链表出现！！！

![](https://gitee.com/veal98/images/raw/master/img/20210328124102.png)

**由于 JDK 1.7 中 HashMap 使用头插会改变链表上元素的的顺序，在旧数组向新数组转移元素的过程中修改了链表中节点的引用关系，因此 JDK 1.8 改成了尾插法，在扩容时会保持链表元素原本的顺序，避免了链表成环的问题**。

### 3. HashMap 的默认初始数组长度是多少？为什么是这么多？

默认数组长度是 16，其实只要是 2 的次幂都行，至于为啥是 16 呢，我觉得应该是个经验值问题，Java 作者是觉得 16 这个长度最为常用。

那为什么数组长度得是 2 的次幂呢？

首先，一般来说，我们常用的 Hash 函数是这样的：index = HashCode(key) % Length，但是因为位运算的效率比较高嘛，所以 HashMap 就相应的改成了这样：index = HashCode(key) & (Length - 1)。

那么**为了保证根据上述公式计算出来的 index 值是分布均匀的，我们就必须保证 Length 是 2 的次幂**。

解释一下：2 的次幂，也就是 2 的 n 次方，它的二进制表示就是 1 后面跟着 n 个 0，那么 2 的 n 次方 - 1 的二进制表示就是 n 个 1。而对于 & 操作来说，任何数与 1 做 & 操作的结果都是这个数本身。也就是说，index 的结果等同于 HashCode(key) 后 n 位的值，只要 HashCode 本身是分布均匀的，那么我们这个 Hash 算法的结果就是均匀的。

### 4. 以 HashMap 为例，解释一下为什么重写 equals 方法的时候还需要重写 hashCode 方法呢？

既然讲到 `equals` 了，那就先顺便回顾下运算符 `==` 的吧，它存在两种使用情况：

- 对于基本数据类型来说， == 比较的是值是否相同；
- 对于引用数据类型来说， == 比较的是内存地址是否相同。

`equals()`也存在两种使用情况：

- 情况 1：没有重写 equals() 方法。则通过 equals() 比较该类的两个对象时，等价于通过 `==` 比较这两个对象（比较的是地址）。
- 情况 2：重写  equals() 方法。一般来说，我们都会重写 equals() 方法来判断两个对象的内容是否相等，比如 String 类就是这样做的。当然，你也可以不这样做。

另外，我们还需要明白，**如果我们不重写 hashCode()，那么任何对象的 hashCode() 值都不会相等**。

OK，回到问题，为什么重写 equals 方法的时候还需要重写 hashCode 方法呢？

以 HashMap 为例，HashMap 是通过 hashCode(key) 去计算寻找 index 的，如果多个 key 哈希得到的 index 一样就会形成链表，那么如何在这个具有相同 hashCode 的对象链表上找到某个对象呢？

那就是通过重写的 `equals` 比较两个对象的值。

总体来说，HashMap 中`get(key)` 一个元素的过程是这样的，先比较 key 的 hashcode() 是否相等，若相等再通过 equals() 比较其值，若 equals() 相等则认为他们是相等的。若 equals() 不相等则认为他们不相等。

如果只重写 equals 没有重写 hashCode()，就会导致相同的对象却拥有不同的 hashCode，也就是说在判断的第一步 HashMap 就会认为这两个对象是不相等的，那显然这是错误的。

### 5. HashMap 线程不安全的表现有哪些？

关于 JDK 1.7 中 HashMap 的线程不安全，上面已经说过了，就是会出现环形链表。虽然 JDK 1.8 采用尾插法避免了环形链表的问题，但是它仍然是线程不安全的，我们来看看 JDK 1.8 中 HashMap 的 `put` 方法：

![](https://gitee.com/veal98/images/raw/master/img/20210328162309.png)

注意上图我圈出来的代码，如果没有发生 Hash 冲突就会直接插入元素。

假设线程 1 和线程 2 同时进行 put 操作，恰好这两条不同的数据的 hash 值是一样的，并且该位置数据为null，这样，线程 1 和线程 2 都会进入这段代码进行插入元素。假设线程 1 进入后还没有开始进行元素插入就被挂起，而线程 2 正常执行，并且正常插入数据，随后线程 1 得到 CPU 调度进行元素插入，这样，线程 2 插入的数据就被覆盖了。

总结一下 HashMap 在 JDK 1.7 和 JDK 1.8 中为什么不安全：

- JDK 1.7：由于采用头插法改变了链表上元素的的顺序，并发环境下扩容可能导致循环链表的问题
- JDK 1.8：由于 put 操作并没有上锁，并发环境下可能发生某个线程插入的数据被覆盖的问题

### 6. 如何保证 HashMap 线程安全？

这个问题留到下篇文章再做讲解，这里先笼统概括下，主要有三种方式：

1）使用 java.util.Collections 类的 `synchronizedMap` 方法包装一下 HashMap，得到线程安全的 HashMap，其原理就是对所有的修改操作都加上 synchronized。方法如下：

```java
public static <K,V> Map<K,V> synchronizedMap(Map<K,V> m) 
```

2）使用线程安全的 `HashTable` 类代替，该类在对数据操作的时候都会上锁，也就是加上 synchronized

3）使用线程安全的 `ConcurrentHashMap` 类代替，该类在 JDK 1.7 和 JDK 1.8 的底层原理有所不同，JDK 1.7 采用数组 + 链表存储数据，使用分段锁 Segment 保证线程安全；JDK 1.8 采用数组 + 链表/红黑树存储数据，使用 CAS + synchronized 保证线程安全。

