# Hashtable 渐渐被人们遗忘了，只有面试官还记得，感动

---

本来准备这篇文章一口气写完 `Hashtable` 和 `ConcurrentHashMap` 的，后来发现 `Hashtable` 就已经很多了，考虑各位的阅读体验，所以 `ConcurrentHashMap` 就放在下篇文章吧。

OK，继续上篇文章 [HashMap 这套八股，不得背个十来遍？](https://mp.weixin.qq.com/s/JuwcUAfvxVQmJdhEBAekUw) 最后提出的问题来讲：

### 1. 如何保证 HashMap 线程安全？

一般有三种方式来代替原生的线程不安全的 `HashMap`：

1）使用 java.util.Collections 类的 `synchronizedMap` 方法包装一下 `HashMap`，得到线程安全的 `HashMap`，其原理就是对所有的修改操作都加上 synchronized。方法如下：

```java
public static <K,V> Map<K,V> synchronizedMap(Map<K,V> m) 
```

2）使用线程安全的 `Hashtable` 类代替，该类在对数据操作的时候都会上锁，也就是加上 synchronized

3）使用线程安全的 `ConcurrentHashMap` 类代替，该类在 JDK 1.7 和 JDK 1.8 的底层原理有所不同，JDK 1.7 采用数组 + 链表存储数据，使用分段锁 Segment 保证线程安全；JDK 1.8 采用数组 + 链表/红黑树存储数据，使用 CAS + synchronized 保证线程安全。

不过前两者的线程并发度并不高，容易发生大规模阻塞，所以一般使用的都是 `ConcurrentHashMap`，他的性能和效率明显高于前两者。

### 2. synchronizedMap 具体是怎么实现线程安全的？

> 这个问题应该很容易被大家漏掉吧，面经中也确实不常出现，也没啥好问的。不过为了保证知识的完整性，这里还是解释一下吧。

一般我们会这样使用 `synchronizedMap` 方法来创建一个线程安全的 Map：

```java
Map m = Collections.synchronizedMap(new HashMap(...));
```

`Collections` 中的这个静态方法 `synchronizedMap` 其实是创建了一个内部类的对象，这个内部类就是 `SynchronizedMap`。在其内部维护了一个普通的 Map 对象以及**互斥锁 mutex**，如下图所示：

![](https://gitee.com/veal98/images/raw/master/img/20210331172454.png)

可以看到 `SynchronizedMap` 有两个构造函数，如果你传入了互斥锁 mutex 参数，就使用我们自己传入的互斥锁。如果没有传入，则将互斥锁赋值为 this，也就是将调用了该构造函数的对象作为互斥锁，即我们上面所说的 Map。

创建出 `SynchronizedMap` 对象之后，通过源码可以看到对于这个对象的所有操作全部都是上了悲观锁 `synchronized` 的：

![](https://gitee.com/veal98/images/raw/master/img/20210331173542.png)

由于多个线程都共享同一把互斥锁，导致同一时刻只能有一个线程进行读写操作，而其他线程只能等待，所以虽然它支持高并发，但是并发度太低，多线程情况下性能比较低下。

而且，大多数情况下，业务场景都是读多写少，多个线程之间的读操作本身其实并不冲突，所以`SynchronizedMap` 极大的限制了读的性能。

所以多线程并发场景我们很少使用 `SynchronizedMap` 。

### 3. 那 Hashtable 呢？

和 `SynchronizedMap` 一样，`Hashtable` 也是非常粗暴的给每个方法都加上了悲观锁 `synchronized`，我们随便找几个方法看看：

![](https://gitee.com/veal98/images/raw/master/img/20210331174332.png)

### 4. 除了这个之外 Hashtable 和 HashMap 还有什么不同之处吗？

**`Hashtable` 是不允许 key 或 value 为 null 的，`HashMap` 的 key 和 value 都可以为 null** ！！！

先解释一下 `Hashtable` 不支持 null key 和 null value 的原理：

如果我们 put 了一个 value 为 null 进入 Map，`Hashtable`  会直接抛空指针异常：

![](https://gitee.com/veal98/images/raw/master/img/20210331175329.png)

2）如果我们 put 了一个 key 为 null 进入 Map，当程序执行到下图框出来的那行代码时就会抛出空指针异常，因为 key 为 null，我们拿了一个 null 值去调用方法：

![](https://gitee.com/veal98/images/raw/master/img/20210331180319.png)

OK，讲完了 Hashtable，再来解释一下 `HashMap` 支持 null key 和 null value 的原理：

1）`HashMap` 相比 `Hashtable` 做了一个特殊的处理，如果我们 put 进来的 key 是 null，`HashMap` 在计算这个 key 的 hash 值时，会直接返回 0：

![](https://gitee.com/veal98/images/raw/master/img/20210331175610.png)

也就是说 `HashMap` 中 key为 null 的键值对的 hash 为 0。因此一个 `HashMap` 对象中只会存储一个 key 为 null 的键值对，因为它们的 hash 值都相同。

2）如果我们 put 进来的 value 是 null，由于 `HashMap` 的 put 方法不会对 value 是否为 null 进行校验，因此一个 `HashMap` 对象可以存储多个 value 为 null 的键值对：

![](https://gitee.com/veal98/images/raw/master/img/20210331180722.png)

不过，这里有个小坑需要注意，我们来看看 `HashMap` 的 get 方法：

![](https://gitee.com/veal98/images/raw/master/img/20210331180936.png)

如果 Map 中没有查询到这个 key 的键值对，那么 get 方法就会返回 null 对象。但是我们上面刚刚说了，`HashMap` 里面可以存在多个 value 为 null 的键值对，也就是说，通过 get(key) 方法返回的结果为 null 有两种可能：

- `HashMap` 中不存在这个 key 对应的键值对
- `HashMap` 中这个 key 对应的 value 为 null

因此，一般来说我们不能使用 get 方法来判断 `HashMap` 中是否存在某个 key，而应该使用 `containsKey` 方法。

### 5. 那到底为什么 Hashtable 不允许 key 和 value 为 null 呢？为什么这么设计呢？

不止是 `Hashtable` 不允许 key 为 null 或者 value 为 null，`ConcurrentHashMap` 也是不允许的。作为支持并发的容器，如果它们像 `HashMap` 一样，允许 null key 和 null value 的话，在多线程环境下会出现问题。

假设它们允许 null key 和 null value，我们来看看会出现什么问题：当你通过 get(key) 获取到对应的 value 时，如果返回的结果是 null 时，你无法判断这个 key 是否真的存在。为此，我们需要调用 containsKey 方法来判断这个 key 到底是 value = null 还是它根本就不存在，如果 containsKey 方法返回的结果是 true，OK，那我们就可以调用 map.get(key) 获取 value。

上面这段逻辑对于单线程的 `HashMap` 当然没有任何问题。在单线程中，当我们得到的 value 是 null 的时候，可以用 map.containsKey(key) 方法来区分二义性。

但是！由于 `Hashtable` 和 `ConcurrentHashMap` 是支持多线程的容器，在调用 map.get(key) 的这个时候 map 对象可能已经不同了。

我们假设此时某个线程 A 调用了 map.get(key) 方法，它返回为 value = null 的真实情况就是因为这个 key 不能存在。当然，线程 A 还是会按部就班的继续用 map.containsKey(key)，我们期望的结果是返回 false。

但是，在线程 A 调用 map.get(key) 方法之后，map.containsKey 方法之前，另一个线程 B 执行了 map.put(key,null) 的操作。那么线程 A 调用的 map.containsKey 方法返回的就是 true 了。这就与我们的假设的真实情况不符合了。

所以，**出于并发安全性的考虑**，`Hashtable` 和 `ConcurrentHashMap` 不允许 key 和 value 为 null。

### 6. Hashtable 和 HashMap 的不同点说完了吗？

除了 `Hashtable` 不允许 null key 和 null value 而 `HashMap` 允许以外，它俩还有以下几点不同：

1）**初始化容量不同**：`HashMap` 的初始容量为 16，`Hashtable` 初始容量为 11。两者的负载因子默认都是 0.75；

2）**扩容机制不同**：当现有容量大于总容量 * 负载因子时，`HashMap` 扩容规则为当前容量翻倍，`Hashtable` 扩容规则为当前容量翻倍 + 1；

3）**迭代器不同**：首先，`Hashtable` 和 `HashMap` 有一个相同的迭代器 Iterator，用法：

```java
Iterator iterator = map.keySet().iterator();
```

`HashMap` 的 Iterator 是 **快速失败 fail-fast** 的，那自然 `Hashtable` 的 Iterator 也是 fail-fast 的。`Hashtable` 是 fail-fast 机制这点很明确，JDK 1.8 的官方文档就是这么写的：

![](https://gitee.com/veal98/images/raw/master/img/20210331211406.png)

但是！！！`Hashtable` 还有另外一个迭代器 Enumeration，这个迭代器是 **失败安全 fail-safe** 的。网络上很多博客提到 `Hashtable` 就说它是 fail-safe 的，这是不正确的、是存在歧义的！

### 7. 介绍下 fail-safe 和 fail-fast 机制

fail-safe 和 fail-fast 是一种思想，一种机制，属于**系统设计范畴**，并非 Java 集合所特有，各位如果熟悉 Dubbo 的话，一定记得 Dubbo 的集群容错策略中也有这俩。

当然，这两种机制在 Java 集合和 Dubbo 中的具体表现肯定是不一样的，本文我们就只说在 Java 集合中，这两种机制的具体表现。

1）**快速失败 fail-fast**：一种快速发现系统故障的机制。一旦发生异常，立即停止当前的操作，并上报给上层的系统来处理这些故障。

举一个最简单的 fail-fast 的例子：

![](https://gitee.com/veal98/images/raw/master/img/20210331214737.png)

这样做的好处就是可以预先识别出一些错误情况，一方面可以避免执行复杂的其他代码，另外一方面，这种异常情况被识别之后也可以针对性的做一些单独处理。

**java.util 包下的集合类都是 fail-fast 的**，比如 `HashMap` 和 `HashTable`，官方文档是这样解释 fail-fast 的：

> The iterators returned by all of this class's "collection view methods" are *fail-fast*: if the map is structurally modified at any time after the iterator is created, in any way except through the iterator's own `remove` method, the iterator will throw a [`ConcurrentModificationException`](https://docs.oracle.com/javase/8/docs/api/java/util/ConcurrentModificationException.html). Thus, in the face of concurrent modification, the iterator fails quickly and cleanly, rather than risking arbitrary, non-deterministic behavior at an undetermined time in the future.

大体意思就是说当 Iterator 这个迭代器被创建后，除了迭代器本身的方法 remove 可以改变集合的结构外，其他的因素如若改变了集合的结构，都将会抛出 `ConcurrentModificationException` 异常。

所谓**结构上的改变**，集合中元素的插入和删除就是结构上的改变，但是对集合中修改某个元素并不是结构上的改变。我们以 `Hashtable` 来演示下 fail-fast 机制抛出异常的实例：

![](https://gitee.com/veal98/images/raw/master/img/20210331221409.png)

分析下这段代码：第一次循环遍历的时候，我们删除了集合 key = "a" 的元素，集合的结构被改变了，所以第二次遍历迭代器的时候，就会抛出异常。

另外，这里多提一嘴，使用 for-each 增强循环也会抛出异常，for-each 本质上依赖了 Iterator。

OK，我们接着往下看官方文档：

> Note that the fail-fast behavior of an iterator cannot be guaranteed as it is, generally speaking, impossible to make any hard guarantees in the presence of unsynchronized concurrent modification. Fail-fast iterators throw `ConcurrentModificationException` on a best-effort basis. Therefore, it would be wrong to write a program that depended on this exception for its correctness: *the fail-fast behavior of iterators should be used only to detect bugs.*

意思就是说：迭代器的 fail-fast 行为是不一定能够得到 100% 得到保证的。但是 fail-fast 迭代器会做出最大的努力来抛出 `ConcurrentModificationException`。因此，程序员编写依赖于此异常的程序的做法是不正确的。迭代器的 fail-fast 行为应该仅用于检测程序中的 Bug。

2）**失败安全 fail-safe**：在故障发生之后会维持系统继续运行。

顾名思义，和 fail-fast 恰恰相反，**当我们对集合的结构做出改变的时候，fail-safe 机制不会抛出异常**。

**java.util.concurrent 包下的容器都是 fail-safe 的，比如 `ConcurrentHashMap`，可以在多线程下并发使用，并发修改。同时也可以在 for-each 增强循环中进行 add/remove**。

不过有个例外，那就是 `java.util.Hashtable`，上面我们说到 `Hashtable` 还有另外一个迭代器 Enumeration，这个迭代器是 fail-safe 的。

`HashTable` 中有一个 keys 方法可以返回 Enumeration 迭代器：

![](https://gitee.com/veal98/images/raw/master/img/20210331222820.png)

至于为什么 fail-safe 不会抛出异常呢，这是因为，当集合的结构被改变的时候，fail-safe 机制会复制一份原集合的数据，然后在复制的那份数据上进行遍历。因此，虽然 fail-safe 不会抛出异常，但存在以下缺点：

- 不能保证遍历的是最新内容。也就是说迭代器遍历的是开始遍历那一刻拿到的集合拷贝，在遍历期间原集合发生的修改迭代器是不知道的；
- 复制时需要额外的空间和时间上的开销。

### 8. 讲讲 fail-fast 的原理是什么

从源码我们可以发现，迭代器在执行 next() 等方法的时候，都会调用 `checkForComodification` 这个方法，查看 modCount 和 expectedModCount 是否相等，如果不相等则抛出异常终止遍历，如果相等就返回遍历。

![](https://gitee.com/veal98/images/raw/master/img/20210331230228.png)

expectedModcount 这个值在对象被创建的时候就被赋予了一个固定的值即 modCount，也就是说 expectedModcount 是不变的，但是 modCount 在我们对集合的元素的个数做出改变（删除、插入）的时候会被改变（修改操作不会）。那如果在迭代器下次遍历元素的时候，发现 modCount 这个值发生了改变，那么走到这个判断语句时就会抛出异常。







