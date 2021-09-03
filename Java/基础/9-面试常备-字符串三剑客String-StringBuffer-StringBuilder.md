# 面试常备，字符串三剑客 String、StringBuffer、StringBuilder

---

字符串操作毫无疑问是计算机程序设计中最常见的行为之一，在 Java 大展拳脚的 Web 系统中更是如此。

全文脉络思维导图如下：

![](https://gitee.com/veal98/images/raw/master/img/20210218215307.png)

## 1. 三剑客之首：不可变的 String

### 概述

**Java 没有内置的字符串类型**， 而是在标准 Java 类库中提供了一个**预定义类** `String`。每个用**双引号括起来的字符串都是 `String` 类的一个实例**：

```java
String e = ""; // 空串
String str = "hello";
```

看一下 `String` 的源码，**在 Java 8 中，`String` 内部是使用 `char` 数组来存储数据的**。

```java
public final class String
    implements java.io.Serializable, Comparable<String>, CharSequence {
    /** The value is used for character storage. */
    private final char value[];
}
```

> 可以看到，`String` 类是被 `final` 修饰的，因此 **`String` 类不允许被继承**。

**而在 `Java 9` 之后，`String` 类的实现改用 `byte` 数组存储字符串**，同时使用 `coder` 来标识使用了哪种编码。

```java
public final class String
    implements java.io.Serializable, Comparable<String>, CharSequence {
    /** The value is used for character storage. */
    private final byte[] value;

    /** The identifier of the encoding used to encode the bytes in {@code value}. */
    private final byte coder;
}
```

不过，无论是 Java 8 还是 Java 9，**用来存储数据的 char 或者 byte 数组 `value` 都一直是被声明为 `final` 的**，这意味着 `value  ` 数组初始化之后就不能再引用其它数组了。并且 `String `内部没有改变 `value` 数组的方法，因此我们就说 `String ` 是不可变的。

所谓不可变，就如同数字 3 永远是数字 3 —样，字符串 “hello” 永远包含字符 h、e、1、1 和 o 的代码单元序列， 不能修改其中的任何一个字符。当然， 可以修改字符串变量 str， 让它引用另外一个字符串， 这就如同可以将存放 3 的数值变量改成存放 4 一样。

我们看个例子：

```java
String str = "asdf";
String x = str.toUpperCase();
```

`toUpperCase` 用来将字符串全部转为大写字符，进入 `toUpperCase` 的源码我们发现，这个看起来会修改 `String` 值的方法，实际上最后是创建了一个全新的 `String` 对象，而最初的 `String` 对象则丝毫未动。

![](https://gitee.com/veal98/images/raw/master/img/20210218165524.png)

### 空串与 Null

空串 `""` 很好理解，就是长度为 0 的字符串。可以调用以下代码检查一个字符串是否为空：

```java
if(str.length() == 0){
    // todo
}
```

或者

```java
if(str.equals("")){
	// todo
}
```

**空串是一个 Java 对象**， 有自己的串长度（ 0 ) 和内容（空），也就是 `value` 数组为空。

`String `变量还可以存放一个特殊的值， 名为 `null`，这表示**目前没有任何对象与该变量关联**。要检查一个字符串是否为 `null`，可如下判断：

```java
if(str == null){
    // todo
}
```

有时要检查一个字符串既不是 `null `也不为空串，这种情况下就需要使用以下条件：

```java
if(str != null && str.length() != 0){
    // todo
}
```

有同学就会觉得，这么简单的条件判断还用你说？没错，这虽然简单，但仍然有个小坑，就是我们**必须首先检查 str 是否为 `null`，因为如果在一个 `null ` 值上调用方法，编译器会报错**。

### 字符串拼接

上面既然说到 `String` 是不可变的，我们来看段代码，为什么这里的字符串 a 却发生了改变？

```java
String a = "hello";
String b = "world";
a = a + b; // a = "helloworld"
```

实际上，在使用 `+` 进行字符串拼接的时候，JVM 是初始化了一个 `StringBuilder` 来进行拼接的。相当于编译后的代码如下：

```java
String a = "hello";
String b = "world";
StringBuilder builder = new StringBuilder();
builder.append(a);
builder.append(b);
a = builder.toString();
```

关于 `StringBuilder` 下文会详细讲解，大家现在只需要知道 `StringBuilder` 是可变的字符串类型就 OK 了。我们看下 `builder.toString()` 的源码：

![](https://gitee.com/veal98/images/raw/master/img/20210218174109.png)

显然，`toString`方法同样是生成了一个新的 `String` 对象，而不是在旧字符串的内容上做更改，相当于把旧字符串的引用指向的新的`String`对象。这也就是字符串 `a` 发生变化的原因。

另外，我们还需要了解一个特性，当将一个字符串与一个非字符串的值进行拼接时，后者被自动转换成字符串（**任何一个 Java 对象都可以转换成字符串**）。例如：

```java
int age = 13;
String rating = "PG" + age; // rating = "PG13"
```

这种特性通常用在输出语句中。例如：

```java
int a = 12;
System.out.println("a = " + a);
```

结合上面这两特性，我们来看个小问题，**空串和 `null` 拼接的结果是啥**？

```java
String str = null;
str = str + "";
System.out.println(str);
```

答案是 `null` 大家应该都能猜出来，但为什么是 `null` 呢？上文说过，使用 `+` 进行拼接实际上是会转换为 `StringBuilder` 使用 `append` 方法进行拼接，编译后的代码如下：

```java
String str = null;
str = str + "";
StringBuilder builder = new StringBuilder();
builder.append(str);
builder.append("");
str = builder.toString();
```

看下 `append` 的源码：

![](https://gitee.com/veal98/images/raw/master/img/20210218175045.png)

可以看出，当传入的字符串是 `null` 时，会调用 `appendNull` 方法，而这个方法会返回 `null`。

### 检测字符串是否相等

可以使用 `equals `方法检测两个字符串是否相等。比如：

```java
String str = "hello";
System.out.println("hello".equals(str)); // true
```

`equals` 其实是 `Object` 类中的一个方法，所有的类都继承于 `Object` 类。讲解 `equals` 方法之前，我们先来回顾一下运算符 `==` 的用法，它存在两种使用情况：

- 对于基本数据类型来说， `==` 比较的是值是否相同；
- 对于引用数据类型来说， `==` 比较的是内存地址是否相同。

举个例子：

```java
String str1 = new String("hello"); 
String str2 = new String("hello");
System.out.println(str1 == str2); // false
```

对 Java 中数据存储区域仍然不明白的可以先回去看看第一章《万物皆对象》。对于上述代码，str1 和 str2 采用构造函数 `new String()` 的方式新建了两个不同字符串，以 `String str1 = new String("hello");` 为例，new 出来的对象存放在堆内存中，用一个引用 str1 来指向这个对象的地址，而这个对象的引用 str1 存放在栈内存中。str1 和 str2 是两个不同的对象，地址不同，因此 `==` 比较的结果也就为 `false`。

![](https://gitee.com/veal98/images/raw/master/img/20210218203257.png)

而实际上，`Object` 类中的原始 `equals` 方法内部调用的还是运算符 `==`，**判断的是两个对象是否具有相同的引用（地址），和 `==` 的效果是一样的**：

![](https://gitee.com/veal98/images/raw/master/img/20210218194515.png)

也就是说，如果你新建的类没有覆盖 `equals` 方法，那么这个方法比较的就是对象的地址。而 `String` 方法覆盖了 `equals` 方法，我们来看下源码：

![](https://gitee.com/veal98/images/raw/master/img/20210218194256.png)

可以看出，`String` 重写的 `equals` 方法比较的是对象的内容，而非地址。

总结下 `equals()`的两种使用情况：

- 情况 1：<u>类没有覆盖 `equals()` 方法</u>。则通过 `equals()` 比较该类的两个对象时，等价于通过 `==` 比较这两个对象（比较的是地址）。
- 情况 2：<u>类覆盖了 `equals()` 方法</u>。一般来说，我们都覆盖 `equals()` 方法来判断两个对象的内容是否相等，比如 `String` 类就是这样做的。当然，你也可以不这样做。

举个例子：

```java
String a = new String("ab"); // a 为一个字符串引用
String b = new String("ab"); // b 为另一个字符串引用,这俩对象的内容一样

if (a.equals(b)) // true
    System.out.println("aEQb");
if (a == b) // false，不是同一个对象，地址不同
    System.out.println("a==b");
```

### 字符串常量池

字符串 `String` 既然作为 `Java` 中的一个类，那么它和其他的对象分配一样，需要耗费高昂的时间与空间代价，作为最基础最常用的数据类型，大量频繁的创建字符串，将会极大程度的影响程序的性能。为此，JVM 为了提高性能和减少内存开销，在实例化字符串常量的时候进行了一些优化：

- 为字符串开辟了一个**字符串常量池 String Pool**，可以理解为缓存区
- 创建字符串常量时，首先检查字符串常量池中是否存在该字符串
- **若字符串常量池中存在该字符串，则直接返回该引用实例，无需重新实例化**；若不存在，则实例化该字符串并放入池中。

举个例子：

```java
String str1 = "hello";
String str2 = "hello";

System.out.printl（"str1 == str2" : str1 == str2 ) //true 
```

对于上面这段代码，`String str1 = "hello";`， **编译器首先会在栈中创建一个变量名为 `str1` 的引用，然后在字符串常量池中查找有没有值为 "hello" 的引用，如果没找到，就在字符串常量池中开辟一个地址存放 "hello" 这个字符串，然后将引用 `str1` 指向 "hello"**。

![](https://gitee.com/veal98/images/raw/master/img/20210218204914.png)

需要注意的是，字符串常量池的位置在 JDK 1.7 有所变化：

- **JDK 1.7 之前**，字符串常量池存在于**常量存储**（Constant storage）中
- **JDK 1.7 之后**，字符串常量池存在于**堆内存**（Heap）中。

<img src="https://gitee.com/veal98/images/raw/master/img/20200906113117.png" style="zoom:;" />

另外，我们还**可以使用 `String `的 `intern()  `方法在运行过程中手动的将字符串添加到 String Pool 中**。具体过程是这样的：

当一个字符串调用 `intern()` 方法时，如果 String Pool 中已经存在一个字符串和该字符串的值相等，那么就会返回 String Pool 中字符串的引用；否则，就会在 String Pool 中添加一个新的字符串，并返回这个新字符串的引用。

看下面这个例子：

```java
String str1 = new String("hello"); 
String str3 = str1.intern();
String str4 = str1.intern();
System.out.println(str3 == str4); // true
```

对于 str3 来说，`str1.intern()` 会先在 String Pool 中查看是否已经存在一个字符串和 str1 的值相等，没有，于是，在 String Pool 中添加了一个新的值和 str1 相等的字符串，并返回这个新字符串的引用。

而对于 str4 来说，`str1.intern() `在 String Pool 中找到了一个字符串和 str1 的值相等，于是直接返回这个字符串的引用。因此 s3 和 s4 引用的是同一个字符串，也就是说它们的地址相同，所以 `str3 == str4` 的结果是 `true`。

![](https://gitee.com/veal98/images/raw/master/img/20210218204412.png)

🚩 **总结：**

- `String str = "i"` 的方式，java 虚拟机会自动将其分配到常量池中；

- `String str = new String(“i”) ` 则会被分到堆内存中。可通过 intern 方法手动加入常量池

### new String("hello") 创建了几个字符串对象

下面这行代码到底创建了几个字符串对象？仅仅只在堆中创建了一个？

```java
String str1 = new String("hello"); 
```

显然不是。对于 str1 来说，`new String("hello")` 分两步走：

- 首先，"hello" 属于字符串字面量，因此编译时期会在 String Pool 中查找有没有值为 "hello" 的引用，如果没找到，就在字符串常量池中开辟地址空间创建一个字符串对象，指向这个 "hello" 字符串字面量；
- 然后，使用 `new `的方式又会在堆中创建一个字符串对象。

因此，使用这种方式一共会创建两个字符串对象（前提是 String Pool 中还没有 "hello" 字符串对象）。

![](https://gitee.com/veal98/images/raw/master/img/20210218205909.png)

## 2. 双生子：可变的 StringBuffer 和 StringBuilder

### String 字符串拼接问题

有些时候， 需要由较短的字符串构建字符串， 例如， 按键或来自文件中的单词。采用字符串拼接的方式达到此目的效率比较低。由于 String 类的对象内容不可改变，所以每当进行字符串拼接时，总是会在内存中创建一个新的对象。既耗时， 又浪费空间。例如：

```java
String s = "Hello";
s += "World";
```

这段简单的代码其实总共产生了三个字符串，即 `"Hello"`、`"World"` 和 `"HelloWorld"`。"Hello" 和 "World" 作为字符串常量会在 String Pool 中被创建，而拼接操作 `+` 会 new 一个对象用来存放 "HelloWorld"。

使用 `StringBuilder/ StringBuffer` 类就可以避免这个问题的发生，毕竟 String 的 `+` 操作底层都是由 `StringBuilder` 实现的。`StringBuilder` 和 `StringBuffer` 拥有相同的父类：

![](https://gitee.com/veal98/images/raw/master/img/20210218211329.png)

但是，`StringBuilder` 不是线程安全的，在多线程环境下使用会出现数据不一致的问题，而 `StringBuffer` 是线程安全的。这是因为在 `StringBuffer` 类内，常用的方法都使用了`synchronized` 关键字进行同步，所以是线程安全的。而 `StringBuilder` 并没有。这也是运行速度 `StringBuilder` 大于 `StringBuffer` 的原因了。因此，如果在单线程下，优先考虑使用 `StringBuilder`。

### 初始化操作

`StringBuilder` 和 `StringBuffer` 这两个类的 API 是相同的，这里就以 `StringBuilder`  为例演示其初始化操作。

`StringBuiler/StringBuffer `不能像 `String ` 那样直接用字符串赋值，所以也不能那样初始化。它**需要通过构造方法来初始化**。首先， 构建一个空的字符串构建器：

```java
StringBuilder builder = new StringBuilder();
```

当每次需要添加一部分内容时， 就调用 `append` 方法：

```java
char ch = 'a';
builder.append(ch);

String str = "ert"
builder.append(str);
```

在需要构建字符串 `String` 时调用  `toString` 方法， 就能得到一个 `String `对象：

```java
String mystr = builder.toString(); // aert
```

## 3. String、StringBuffer、StringBuilder 比较

|               | 可变性 | 线程安全                                                     |
| :-----------: | :----: | ------------------------------------------------------------ |
|    String     | 不可变 | 因为不可变，所以是线程安全的                                 |
| StringBuffer  |  可变  | 线程安全的，因为其内部大多数方法都使用 `synchronized `进行同步。其效率较低 |
| StringBuilder |  可变  | 不是线程安全的，因为没有使用 `synchronized `进行同步，这也是其效率高于 StringBuffer  的原因。单线程下，优先考虑使用 StringBuilder。 |

关于 `synchronized` 保证线程安全的问题，我们后续文章再说。

## 📚 References

- 《Java 核心技术 - 卷 1 基础知识 - 第 10 版》
- 《Thinking In Java（Java 编程思想）- 第 4 版》