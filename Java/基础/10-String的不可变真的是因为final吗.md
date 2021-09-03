# String 的不可变真的是因为 final 吗？

---

`String` 为啥不可变？因为 `String` 中的 char 数组被 final 修饰。这套回答相信各位已经背烂了，But 这并不正确！

> - 面试官：讲讲 `String`、`StringBuilder`、`StringBuffer` 的区别
> - 我：`String` 不可变，而 `StringBuilder` 和 `StringBuffer` 可变，叭叭叭 ......
> - 面试官：`String` 为什么不可变？
> - 我：`String` 被 `final` 修饰，这说明 `String` 不可继承；并且`String` 中真正存储字符的地方是 char 数组，这个数组被 `final` 修饰，所以 `String` 不可变
> - 面试官：`String` 的不可变真的是因为 `final` 吗？
> - 我：是.....是的吧
> - 面试官：OK，你这边还有什么问题吗？
> - 我：卒......

### 什么是不可变？

《Effective Java》中对于不可变对象（Immutable Object）的定义是：**对象一旦被创建后，对象所有的状态及属性在其生命周期内不会发生任何变化**。这就意味着，一旦我们将一个对象分配给一个变量，就无法再通过任何方式更改对象的状态了。

`String` 不可变的表现就是当我们试图对一个已有的对象 "abcd" 赋值为 "abcde"，`String` 会新创建一个对象：

![](https://gitee.com/veal98/images/raw/master/img/20210402224733.png)

### String 为什么不可变？

`String` 用 final 修饰 char 数组，这个数组无法被修改，这么说确实没啥问题。

![](https://gitee.com/veal98/images/raw/master/img/20210402215240.png)

但是！！！**这个无法被修改仅仅是指引用地址不可被修改（也就是说栈里面的这个叫 value 的引用地址不可变，编译器不允许我们把 value 指向堆中的另一个地址），并不代表存储在堆中的这个数组本身的内容不可变**。举个例子：

![](https://gitee.com/veal98/images/raw/master/img/20210402215319.png)

如果我们直接修改数组中的元素，是完全 OK 的：

![](https://gitee.com/veal98/images/raw/master/img/20210402215616.png)

那既然我们说 `String` 是不可变的，那显然**仅仅靠 final 是远远不够的**：

1）首先，char 数组是 private 的，并且 `String` 类没有对外提供修改这个数组的方法，所以它初始化之后外界没有有效的手段去改变它；

2）其次，`String` 类被 final 修饰的，也就是不可继承，避免被他人继承后破坏；

3）最重要的！是因为 Java 作者在 `String` 的所有方法里面，都很小心地避免去修改了 char 数组中的数据，**涉及到对 char 数组中数据进行修改的操作全部都会重新创建一个 `String` 对象**。你可以随便翻个源码看看来验证这个说法，比如 substring 方法：

![](https://gitee.com/veal98/images/raw/master/img/20210406202957.png)

### 为什么要设计成不可变的呢？

1）首先，**字符串常量池的需要**。

我们来回顾一下字符串常量池的定义：大量频繁的创建字符串，将会极大程度的影响程序的性能。为此，JVM 为了提高性能和减少内存开销，在实例化字符串常量的时候进行了一些优化：

- 为字符串开辟了一个字符串常量池 String Pool，可以理解为缓存区
- 创建字符串常量时，首先检查字符串常量池中是否存在该字符串
- 若字符串常量池中存在该字符串，则直接返回该引用实例，无需重新实例化；若不存在，则实例化该字符串并放入池中。

如下面的代码所示，堆内存中只会创建一个 `String` 对象：

```java
String str1 = "hello";
String str2 = "hello";

System.out.println(str1 == str2) // true 
```

![](https://gitee.com/veal98/images/raw/master/img/20210218204914.png)

假设 `String` 允许被改变，那如果我们修改了 str2 的内容为 good，那么 str1 也会被修改，显然这不是我们想要看见的结果。

2）另外一点也比较容易想到，`String` 被设计成不可变就是为了**安全**。

作为最基础最常用的数据类型，`String` 被许多 Java 类库用来作为参数，如果 `String` 不是固定不变的，将会引起各种安全隐患。

举个例子，我们来看看将可变的字符串 `StringBuilder` 存入 `HashSet` 的场景：

  ![](https://gitee.com/veal98/images/raw/master/img/20210406210219.png)

我们把可变字符串 s3 指向了 s1 的地址，然后改变 s3 的值，由于 `StringBuilder` 没有像 `String` 那样设计成不可变的，所以 s3 就会直接在 s1 的地址上进行修改，导致 s1 的值也发生了改变。于是，糟糕的事情发生了，`HashSet` 中出现了两个相等的元素，破坏了 `HashSet` 的不包含重复元素的原则。

另外，在多线程环境下，众所周知，多个线程同时想要修改同一个资源，是存在危险的，而 `String` 作为不可变对象，不能被修改，并且多个线程同时读同一个资源，是完全没有问题的，所以 `String` 是线程安全的。

### String 真的不可变吗？

想要改变 `String` 无非就是改变 char 数组 value 的内容，而 value 是私有属性，那么在 Java 中有没有某种手段可以访问类的私有属性呢？

没错，就是反射，使用反射可以直接修改 char 数组中的内容，当然，一般来说我们不这么做。

看下面代码：

![](https://gitee.com/veal98/images/raw/master/img/20210406204010.png)

### 总结

总结来说，**并不是因为 char 数组是 `final` 才导致 `String` 的不可变，而是为了把 `String` 设计成不可变才把 char 数组设置为 `final` 的**。下面是一些创建不可变对象的简单策略，当然，也并非所有不可变类都完全遵守这些规则：

- 不要提供 setter 方法（包括修改字段的方法和修改字段引用对象的方法）；
- 将类的所有字段定义为 final、private 的；
- 不允许子类重写方法。简单的办法是将类声明为 final，更好的方法是将构造函数声明为私有的，通过工厂方法创建对象；
- 如果类的字段是对可变对象的引用，不允许修改被引用对象。