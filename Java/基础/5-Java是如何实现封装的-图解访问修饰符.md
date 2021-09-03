# Java 小白成长记 · 第 5 篇《Java 是如何实现封装的 — 图解访问修饰符》

---

## 0. 前言

> 这是一个技术疯狂迭代的时代，各种框架层出不穷，然而底层基础才是核心竞争力。博主（小牛肉）在现有的知识基础上，以上帝视角对 Java 语言基础进行复盘，汇总《Java 小白成长记》系列，力争从 0 到 1，全文无坑。

在第一篇文章 [Java 小白成长记 · 第 1 篇《万物皆对象》](https://mp.weixin.qq.com/s/W3KrCirgCrqrSiOQ8P3tAQ) 中我们就已经了解到，面向对象具有四大基本特点：

- **抽象**：对同一类型的对象的共同属性和行为进行概括，形成**类（class)** 。类是构造对象的模板或蓝图。由类构造（construct) 对象的过程称为创建类的**实例 （instance )**.
- **封装**：将抽象出的数据、代码封装在一起，隐藏对象的属性和实现细节，仅对外提供公共访问方式，将变化隔离，便于使用，提高复用性和安全性
- **继承**：在已有类的基础上，进行扩展形成新的类，提高代码复用性。继承是多态的前提
- **多态**：所谓多态就是同一函数名具有不同的功能实现方式

抽象和类的概念想必通过前面四篇文章，大家已经了解的差不多了，那么这篇文章我们就来讲解 Java 作为一种面向对象的编程语言，它是如何实现封装的。

## 1. 什么是封装

在面向对象程序设计方法中，封装（Encapsulation）是指一种将抽象性函数接口的实现细节部分包装、隐藏起来的方法。

通俗来说，可以认为封装就是一个保护屏障，防止某个类的代码和数据被外部类定义的代码随机访问。要访问该类的代码和数据，必须通过**严格的访问控制**。

**封装最主要的功能在于我们能修改自己的实现代码，而不用修改那些调用我们代码的程序片段**。

适当的封装可以让程序更容易理解与维护，也加强了程序的安全性。

OK，总结一下封装的优点：

- 良好的封装能够减少耦合
- 类内部的结构可以自由修改
- 可以对成员变量进行更精确的控制
- 隐藏信息，实现细节

## 2. Java 是如何实现封装的

上文我们提到，要访问某个被封装的类，必须通过严格的访问控制，于是 Java 就为此设计了严格的**访问修饰符**（access specifier）用于修饰被封装的类的访问权限，从“最大权限”到“最小权限”依次是：

- 公开的 - `public`
- 受保护的 - `protected`
- 包访问权限（没有关键字）
- 私有的 - `private`

首先我们需要了解，**类的权限和类中的字段或方法的权限都是可以被修饰的**：

- 对于类中的字段或方法来说：这四个访问修饰符都可以用来修饰
- 而对于类来说：只有包访问权限或 `public` 可以用来修饰（这点需要注意）

所以无论如何，万物都有某种形式的访问控制权。

## 3. 包的概念

在具体学习访问修饰符之前，我们还需要掌握包的概念，因为尽管 Java 设计了严格的访问修饰符，但是这种机制仍然不够完善，其中存在的问题就是如何将类库组件捆绑到一个内聚的类库单元中，意思就是说如何将某些有关联的类汇总到一个大的组织中进行统一管理。Java 为此引入了包的概念，通过 `package` 关键字加以控制，**类在相同包下还是在不同包下，会影响访问修饰符**。掌握包的概念之后你才能明白访问修饰符的全部含义。

顾名思义，包（package）就是用来汇聚一组类的，所以包也可以理解为类库。为了把这些类集中在一起，就需要使用关键字 `package` 来指明这些类是位于哪个包下面的。

注意：如果你使用了 `package` 语句，它必须是文件中除了注释之外的第一行代码。

💬 比如说：

```java
package hiding;

public class MyClass {
    // ...
}
```

> 关于类的访问权限会在最后进行讲解，此处大家只需要知道 `MyClass` 这个类由于被 `public` 修饰，所以可以被任何人访问

上面这段代码即意味着 `MyClass` 这个类是一个名为 `hiding` 包的一部分。任何人想要使用该类，必须指明完整的类名或者使用 `import` 关键字导入 `hiding`：

1）第一种方法：使用 `import` 关键字（推荐）

```java
import hiding.mypackage.*;

public class ImportedMyClass {
    public static void main(String[] args) {
        MyClass m = new MyClass();
    }
}    
```

2）第二种方法：使用完整的类名

```java
hiding.mypackage.MyClass m = new hiding.mypackage.MyClass();
```

由此可见，使用了包之后，不仅可以有效的聚合类，而且可以将单一的全局命名空间分隔开，从而避免名称冲突。

## 4. 访问修饰符详解

掌握了包的概念后，我们再回到本文的主题。上文我们说过：这四个访问修饰符都可以用来修饰类中的字段或方法。

也就是说如果 Java 访问权限修饰符 `public`，`protected `和 `private `位于某个类中的字段名和方法名之前，就可以控制它所修饰的对象。如果不提供访问修饰符，就意味着这个字段或方法拥有"包访问权限"。

下面我们详细解释这四个访问修饰符是如何作用于类中的方法和字段的 👇

### ① 包访问权限

我们已经了解了什么是包，那么什么是包访问权限呢？

所谓**包访问权限**，就是如果不对这个成员（类、字段、方法）提供访问修饰符，那么这个成员就可以被**同一个包中的所有方法**访问，但是这个包之外的成员无法访问。包访问权限也称**默认访问权限**。

💬 举个例子：

```java
package java.awt;

public class Window { 
    String warningString; 
} 
```

> `Cookie` 类声明为 `public`，表示所有类都可以访问它

这意味着 `java.awt` 包中的所有类的方法都可以访问变量 `warningString`， 并将它设置为任意值。

画个图帮助大家直观的理解（假设所有类都是 `public`）：

![](https://gitee.com/veal98/images/raw/master/img/20210124203259.png)

由此可以看出，包访问权限允许将包内所有相关的类组合起来，以使得它们彼此之间可以轻松的相互使用。**构建包访问权限机制是将类聚集在包中的重要原因之一**。

### ② public 接口访问权限

当你使用关键字 `public`，就意味着紧随 `public` 后声明的成员（类、字段、方法）对于每个人都是可用的。

💬 例如：

```java
package A.B; // A.B 包

public class Cookie{
	public Cookie(){ // public 接口访问权限
        System.out.println("Cookie constructor");
    }	
    void bite(){ //  包访问权限
        System.out.println("bite");
    }
}
```

在另一个类中使用它：

```java
package A; // A 包
import A.B.Cookie.*;
    
public class Dinner{
    public static void main(String[] args){
        Cookie x = new Cookew();
        // x.bite(); can't access
    }
}
```

这里定义的 `Cookie` 类的构造函数是 `public ` 的，所有的类都能访问它。而 `bite `方法未声明访问修饰符，具有包访问权限，即它只给在 `A.B` 包中的类提供访问权，所以 `bite()` 方法对于在 `A` 包中的 `Dinner  `类来说是无法访问的。

画个图帮助大家直观的理解（假设所有类都是 `public`）：

![](https://gitee.com/veal98/images/raw/master/img/20210124203504.png)

### ③ private 私有访问权限

关键字 `private` 意味着**除了包含该成员的类，其他任何类都无法访问这个成员**。同一包中的其他类无法访问 `private` 成员，因此这等于说是自己隔离自己。

💬 举个例子：

```java
class Cookie{
	private Cookie(){ 
        System.out.println("Cookie constructor");
    }	
    static Cookie makeACookie(){
        return new Cookie();
    }
}

public class Dinner{
    public static void main(String[] args){
        // Cookie x = new Cookie(); Can't access
        Cookie x = Cookie.makeACookie(); // 可通过static 方法调用
    }
}
```

画个图帮助大家直观的理解（假设所有类都是 `public`）：

![](https://gitee.com/veal98/images/raw/master/img/20210124204254.png)

### ④ protected 继承访问权限

`protected` 是这四种访问权限中相对来说比较复杂的一个，并且涉及继承相关的东西，如果对继承毫不了解的小伙伴可以暂时先略过这段内容。

首先，`protected `也提供包访问权限，也就是说相同包内的其他类可以访问 `protected `元素，而其他包无法访问。但是有一点例外，即不同于包访问权限的是：**即使父类和子类不在同一个包下，子类也可以访问父类中具有 `protected` 访问权限的成员**。（而对于包访问权限来说，如果子类和父类不在一个包下，子类是无法访问父类中具有包访问权限的成员的）

```java
package A.B; // A.B 包

public class Cookie{
    protected void bite(){ 
        System.out.println("bite");
    }
}
```

子类继承 `Cookie`：

```java
package C; // C 包
import A.B.Cookie.*;

public class ChocolateChip extends Cookie{
    public void test(){
        bite(); // protected method
    }
}
```

对比包访问权限的例子，如果 `bite `具有的是包访问权限，显然是无法跨包访问的。而如果将其声明为 `protected`，那么对于所有继承自 `Cookie ` 的方法都可以使用它。

画个图帮助大家直观的理解（假设所有类都是 `public`）：

![](https://gitee.com/veal98/images/raw/master/img/20210124205831.png)

### ⑤ 总结

四个访问修饰符介绍完毕，其实无非就是**类控制着哪些代码有权访问自己的成员**。其他包中的代码不能一上来就说"嗨，我是 **Bob** 的朋友！"，然后想看到 **Bob** 的 `protected`、包访问权限和 `private` 成员。取得对成员的访问权的唯一方式是：

1. 使成员成为 `public`。那么无论是谁，无论在哪，都可以访问它。
2. 赋予成员默认包访问权限，不用加任何访问修饰符，然后将其他类放在相同的包内。这样，其他类就可以访问该成员。
3. 继承的类既可以访问 `public` 成员，也可以访问 `protected` 成员（但不能访问 `private` 成员）。只有当两个类处于同一个包内，它才可以访问包访问权限的成员。
4. 提供访问器（accessor）和修改器（mutator）方法（也称为"get/set" 方法），从而读取和改变值。

## 5. 基于类的访问权限

上文已经基本讲完了基于类的访问权限，无非就是只能使用包访问权限和 `public` 修饰，效果都是一样的，这里就画两张图帮助大家再回顾一下吧：

```java
package A;

public class x{
    // ...
}
```

![](https://gitee.com/veal98/images/raw/master/img/20210124211820.png)

```java
package A;

class y{
    // ...
}
```

![](https://gitee.com/veal98/images/raw/master/img/20210124211757.png)