# 读懂框架设计的灵魂—Java反射机制

---

Java 反射机制对于小白来说，真的是一道巨大的坎儿，其他的东西吧，无非就是内容多点，多看看多背背就好了，反射真的就是不管看了多少遍不理解就还是不理解，而且学校里面的各大教材应该都没有反射这个章节，有也是一带而过。说实话，在这篇文章之前，我对反射也并非完全了解，毕竟平常开发基本用不到，不过，看完这篇文章相信你对反射就没啥疑点了。

全文脉络思维导图如下：

![](https://gitee.com/veal98/images/raw/master/img/20210221151641.png)

## 1. 抛砖引玉：为什么要使用反射

前文我们说过，接口的使用提高了代码的可维护性和可扩展性，并且降低了代码的耦合度。来看个例子：

首先，我们拥有一个接口 X 及其方法 test，和两个对应的实现类 A、B：

```java
public class Test {
    
    interface X {
    	public void test();
	}

    class A implements X{
        @Override
        public void test() {
             System.out.println("I am A");
        }
    }

    class B implements X{
        @Override
        public void test() {
            System.out.println("I am B");
    }
}
```

通常情况下，我们需要使用哪个实现类就直接 new 一个就好了，看下面这段代码：

```java
public class Test {    

    ......

	public static void main(String[] args) {
        X a = create1("A");
        a.test();
        X b = create1("B");
        b.test();
    }

    public static X create1(String name){
        if (name.equals("A")) {
            return new A();
        } else if(name.equals("B")){
            return new B();
        }
        return null;
    }

}
```

按照上面这种写法，如果有成百上千个不同的 X 的实现类需要创建，那我们岂不是就需要写上千个 if 语句来返回不同的 X 对象？

我们来看看看反射机制是如何做的：

```java
public class Test {

    public static void main(String[] args) {
		X a = create2("A");
        a.test();
        X b = create2("B");
        b.testReflect();
    }
    
	// 使用反射机制
    public static X create2(String name){
        Class<?> class = Class.forName(name);
        X x = (X) class.newInstance();
        return x;
    }
}
```

向 `create2()` 方法传入包名和类名，通过反射机制动态的加载指定的类，然后再实例化对象。

看完上面这个例子，相信诸位对反射有了一定的认识。反射拥有以下四大功能：

- 在运行时（动态编译）获知任意一个对象所属的类。
- 在运行时构造任意一个类的对象。
- 在运行时获知任意一个类所具有的成员变量和方法。
- 在运行时调用任意一个对象的方法和属性。

上述这种**动态获取信息、动态调用对象的方法**的功能称为 Java 语言的反射机制。

## 2. 理解 Class 类

要想理解反射，首先要理解 `Class` 类，因为 `Class` 类是反射实现的基础。

![](https://gitee.com/veal98/images/raw/master/img/20210220150043.png)

在程序运行期间，JVM 始终为所有的对象维护一个被称为**运行时的类型标识**，这个信息跟踪着每个对象所属的类的完整结构信息，包括包名、类名、实现的接口、拥有的方法和字段等。可以通过专门的 Java 类访问这些信息，这个类就是 `Class` 类。我们可以把 `Class` 类理解为**类的类型**，一个 `Class` 对象，称为类的类型对象，**一个 `Class` 对象对应一个加载到 JVM 中的一个 `.class` 文件**。

<u>在通常情况下，一定是先有类再有对象</u>。以下面这段代码为例，类的正常加载过程是这样的：

```java
import java.util.Date; // 先有类

public class Test {
    public static void main(String[] args) {
        Date date = new Date(); // 后有对象
        System.out.println(date);
    }
}
```

首先 JVM 会将你的代码编译成一个 `.class` 字节码文件，然后被类加载器（Class Loader）加载进 JVM 的内存中，**同时会创建一个 `Date` 类的 `Class` 对象存到堆中**（注意这个不是 new 出来的对象，而是类的类型对象）。JVM 在创建 `Date` 对象前，会先检查其类是否加载，寻找类对应的 `Class` 对象，若加载好，则为其分配内存，然后再进行初始化 `new Date()`。

![](https://gitee.com/veal98/images/raw/master/img/20210220201500.png)



需要注意的是，**每个类只有一个 `Class` 对象**，也就是说如果我们有第二条 `new Date()` 语句，JVM 不会再生成一个 `Date` 的 `Class` 对象，因为已经存在一个了。这也使得我们可以利用 `==` 运算符实现两个类对象比较的操作：

```java
System.out.println(date.getClass() == Date.getClass()); // true
```

OK，那么在加载完一个类后，堆内存的方法区就产生了一个 `Class` 对象，这个对象就包含了完整的类的结构信息，**我们可以通过这个 `Class` 对象看到类的结构**，就好比一面镜子。所以我们形象的称之为：反射。

说的再详细点，再解释一下。上文说过，在通常情况下，一定是先有类再有对象，我们把这个通常情况称为 “正”。那么反射中的这个 “反” 我们就可以理解为根据对象找到对象所属的类（对象的出处）

```java
Date date = new Date();
System.out.println(date.getClass()); // "class java.util.Date"
```

通过反射，也就是调用了 `getClass()` 方法后，我们就获得了 `Date` 类对应的 `Class` 对象，看到了 `Date` 类的结构，输出了 `Date` 对象所属的类的完整名称，即找到了对象的出处。当然，获取 `Class` 对象的方式不止这一种。

![](https://gitee.com/veal98/images/raw/master/img/20210220201817.png)

## 3. 获取 Class 类对象的四种方式

从 `Class` 类的源码可以看出，它的构造函数是私有的，也就是说只有 JVM 可以创建 `Class` 类的对象，我们不能像普通类一样直接 new 一个 `Class` 对象。

![](https://gitee.com/veal98/images/raw/master/img/20210220202541.png)

我们只能通过已有的类来得到一个 `Class `类对象，Java 提供了四种方式：

**第一种：知道具体类的情况下可以使用**：

```java
Class alunbarClass = TargetObject.class;
```

但是我们一般是不知道具体类的，基本都是通过遍历包下面的类来获取 Class 对象，通过此方式获取 Class 对象不会进行初始化。

**第二种：通过 `Class.forName() `传入全类名获取**：

```java
Class alunbarClass1 = Class.forName("com.xxx.TargetObject");
```

这个方法内部实际调用的是 `forName0`：

![](https://gitee.com/veal98/images/raw/master/img/20210220202713.png)

第 2 个 `boolean` 参数表示类是否需要初始化，<u>默认是需要初始化</u>。一旦初始化，就会触发目标对象的 `static` 块代码执行，`static` 参数也会被再次初始化。

**第三种：通过对象实例 `instance.getClass()` 获取**：

```java
Date date = new Date();
Class alunbarClass2 = date.getClass(); // 获取该对象实例的 Class 类对象
```

**第四种：通过类加载器 `xxxClassLoader.loadClass()` 传入类路径获取**

```java
class clazz = ClassLoader.LoadClass("com.xxx.TargetObject");
```

<u>通过类加载器获取 Class 对象不会进行初始化</u>，意味着不进行包括初始化等一些列步骤，静态块和静态对象不会得到执行。这里可以和 `forName` 做个对比。

## 4. 通过反射构造一个类的实例

上面我们介绍了获取 `Class` 类对象的方式，那么成功获取之后，我们就需要构造对应类的实例。下面介绍三种方法，第一种最为常见，最后一种大家稍作了解即可。

### ① 使用 Class.newInstance

举个例子：

```java
Date date1 = new Date();
Class alunbarClass2 = date1.getClass();
Date date2 = alunbarClass2.newInstance(); // 创建一个与 alunbarClass2 具有相同类类型的实例
```

创建了一个与 `alunbarClass2` 具有相同类类型的实例。 

需要注意的是，**`newInstance `方法调用默认的构造函数（无参构造函数）初始化新创建的对象。如果这个类没有默认的构造函数， 就会抛出一个异常**。

![](https://gitee.com/veal98/images/raw/master/img/20210220204343.png)

### ② 通过反射先获取构造方法再调用

由于不是所有的类都有无参构造函数又或者类构造器是 `private` 的，在这样的情况下，如果我们还想通过反射来实例化对象，`Class.newInstance` 是无法满足的。

此时，我们可以使用 `Constructor` 的 `newInstance` 方法来实现，先获取构造函数，再执行构造函数。

![](https://gitee.com/veal98/images/raw/master/img/20210220214318.png)

从上面代码很容易看出，`Constructor.newInstance` 是可以携带参数的，而 `Class.newInstance` 是无参的，这也就是为什么它只能调用无参构造函数的原因了。

> 大家不要把这两个 `newInstance` 方法弄混了。如果被调用的类的构造函数为默认的构造函数，采用`Class.newInstance()` 是比较好的选择， 一句代码就 OK；如果需要调用类的带参构造函数、私有构造函数等， 就需要采用 `Constractor.newInstance()`

`Constructor.newInstance` 是执行构造函数的方法。我们来看看获取构造函数可以通过哪些渠道，作用如其名，以下几个方法都比较好记也容易理解，返回值都通过 `Cnostructor` 类型来接收。

**批量获取构造函数**：

1）获取所有"公有的"构造方法

```java
public Constructor[] getConstructors() { }
```

2）获取所有的构造方法（包括私有、受保护、默认、公有）

```java
public Constructor[] getDeclaredConstructors() { }
```

**单个获取构造函数**：

1）获取一个指定参数类型的"公有的"构造方法

```java
public Constructor getConstructor(Class... parameterTypes) { }
```

2）获取一个指定参数类型的"构造方法"，可以是私有的，或受保护、默认、公有

```java
public Constructor getDeclaredConstructor(Class... parameterTypes) { }
```

举个例子：

```java
package fanshe;

public class Student {
	//（默认的构造方法）
	Student(String str){
		System.out.println("(默认)的构造方法 s = " + str);
	}
	// 无参构造方法
	public Student(){
		System.out.println("调用了公有、无参构造方法执行了。。。");
	}
	// 有一个参数的构造方法
	public Student(char name){
		System.out.println("姓名：" + name);
	}
	// 有多个参数的构造方法
	public Student(String name ,int age){
		System.out.println("姓名："+name+"年龄："+ age);//这的执行效率有问题，以后解决。
	}
	// 受保护的构造方法
	protected Student(boolean n){
		System.out.println("受保护的构造方法 n = " + n);
	}
	// 私有构造方法
	private Student(int age){
		System.out.println("私有的构造方法年龄："+ age);
	}
}

----------------------------------
    
public class Constructors {
	public static void main(String[] args) throws Exception {
		// 加载Class对象
		Class clazz = Class.forName("fanshe.Student");
        
		// 获取所有公有构造方法
		Constructor[] conArray = clazz.getConstructors();
		for(Constructor c : conArray){
			System.out.println(c);
		}
        
		// 获取所有的构造方法(包括：私有、受保护、默认、公有)
		conArray = clazz.getDeclaredConstructors();
		for(Constructor c : conArray){
			System.out.println(c);
		}
        
		// 获取公有、无参的构造方法
        // 因为是无参的构造方法所以类型是一个null,不写也可以：这里需要的是一个参数的类型，切记是类型
		// 返回的是描述这个无参构造函数的类对象。
		Constructor con = clazz.getConstructor(null);
		Object obj = con.newInstance(); // 调用构造方法
		
		// 获取私有构造方法
		con = clazz.getDeclaredConstructor(int.class);
		System.out.println(con);
		con.setAccessible(true); // 为了调用 private 方法/域 我们需要取消安全检查
		obj = con.newInstance(12); // 调用构造方法
	}
}
```

### ③ 使用开源库 Objenesis

Objenesis 是一个开源库，和上述第二种方法一样，可以调用任意的构造函数，不过封装的比较简洁：

```java
public class Test {
    // 不存在无参构造函数
    private int i;
    public Test(int i){
        this.i = i;
    }
    public void show(){
        System.out.println("test..." + i);
    }
}

------------------------
    
public static void main(String[] args) {
        Objenesis objenesis = new ObjenesisStd(true);
        Test test = objenesis.newInstance(Test.class);
        test.show();
    }
```

使用非常简单，`Objenesis` 由子类 `ObjenesisObjenesisStd`实现。详细源码此处就不深究了，了解即可。

## 5. 通过反射获取成员变量并使用

和获取构造函数差不多，获取成员变量也分批量获取和单个获取。返回值通过 `Field` 类型来接收。

**批量获取**：

1）获取所有公有的字段

```java
public Field[] getFields() { }
```

2）获取所有的字段（包括私有、受保护、默认的）

```java
public Field[] getDeclaredFields() { }
```

**单个获取**：

1）获取一个指定名称的公有的字段

```java
public Field getField(String name) { }
```

2）获取一个指定名称的字段，可以是私有、受保护、默认的

```java
public Field getDeclaredField(String name) { }
```

获取到成员变量之后，如何修改它们的值呢？

![](https://gitee.com/veal98/images/raw/master/img/20210220211126.png)

`set` 方法包含两个参数：

- obj：哪个对象要修改这个成员变量
- value：要修改成哪个值

举个例子:

```java
package fanshe.field;

public class Student {
	public Student(){
        
	}
	
	public String name;
	protected int age;
	char sex;
	private String phoneNum;
	
	@Override
	public String toString() {
		return "Student [name=" + name + ", age=" + age + ", sex=" + sex
				+ ", phoneNum=" + phoneNum + "]";
	}
}

----------------------------------
    
public class Fields {
    public static void main(String[] args) throws Exception {
        // 获取 Class 对象
        Class stuClass = Class.forName("fanshe.field.Student");
        // 获取公有的无参构造函数
        Constructor con = stuClass.getConstructor();
		
		// 获取私有构造方法
		con = clazz.getDeclaredConstructor(int.class);
		System.out.println(con);
		con.setAccessible(true); // 为了调用 private 方法/域 我们需要取消安全检查
		obj = con.newInstance(12); // 调用构造方法
        
        // 获取所有公有的字段
        Field[] fieldArray = stuClass.getFields();
        for(Field f : fieldArray){
            System.out.println(f);
        }

         // 获取所有的字段 (包括私有、受保护、默认的)
        fieldArray = stuClass.getDeclaredFields();
        for(Field f : fieldArray){
            System.out.println(f);
        }

        // 获取指定名称的公有字段
        Field f = stuClass.getField("name");
        Object obj = con.newInstance(); // 调用构造函数，创建该类的实例
        f.set(obj, "刘德华"); // 为 Student 对象中的 name 属性赋值


        // 获取私有字段
        f = stuClass.getDeclaredField("phoneNum");
        f.setAccessible(true); // 暴力反射，解除私有限定
        f.set(obj, "18888889999"); // 为 Student 对象中的 phoneNum 属性赋值
    }
}
```

## 6. 通过反射获取成员方法并调用

同样的，获取成员方法也分批量获取和单个获取。返回值通过 `Method` 类型来接收。

**批量获取**：

1）获取所有"公有方法"（包含父类的方法，当然也包含 `Object` 类）

```java
public Method[] getMethods() { }
```

2）获取所有的成员方法，包括私有的（不包括继承的）

```java
public Method[] getDeclaredMethods() { }
```

**单个获取**：

获取一个指定方法名和参数类型的成员方法：

```java
public Method getMethod(String name, Class<?>... parameterTypes)
```

获取到方法之后该怎么调用它们呢？

![](https://gitee.com/veal98/images/raw/master/img/20210220210903.png)

`invoke` 方法中包含两个参数：

- obj：哪个对象要来调用这个方法
- args：调用方法时所传递的实参

举个例子：

```java
package fanshe.method;
 
public class Student {
	public void show1(String s){
		System.out.println("调用了：公有的，String参数的show1(): s = " + s);
	}
	protected void show2(){
		System.out.println("调用了：受保护的，无参的show2()");
	}
	void show3(){
		System.out.println("调用了：默认的，无参的show3()");
	}
	private String show4(int age){
		System.out.println("调用了，私有的，并且有返回值的，int参数的show4(): age = " + age);
		return "abcd";
	}
}

-------------------------------------------
public class MethodClass {
	public static void main(String[] args) throws Exception {
		// 获取 Class对象
		Class stuClass = Class.forName("fanshe.method.Student");
        // 获取公有的无参构造函数
        Constructor con = stuClass.getConstructor();
        
		// 获取所有公有方法
		stuClass.getMethods();
		Method[] methodArray = stuClass.getMethods();
		for(Method m : methodArray){
			System.out.println(m);
		}
        
		// 获取所有的方法，包括私有的
		methodArray = stuClass.getDeclaredMethods();
		for(Method m : methodArray){
			System.out.println(m);
		}
        
		// 获取公有的show1()方法
		Method m = stuClass.getMethod("show1", String.class);
		System.out.println(m);
		Object obj = con.newInstance(); // 调用构造函数，实例化一个 Student 对象
		m.invoke(obj, "小牛肉");
		
		// 获取私有的show4()方法
		m = stuClass.getDeclaredMethod("show4", int.class);
		m.setAccessible(true); // 解除私有限定
		Object result = m.invoke(obj, 20);
		System.out.println("返回值：" + result);
	}
}
```

## 7. 反射机制优缺点

**优点**： 比较灵活，能够在运行时动态获取类的实例。

**缺点**：

1）性能瓶颈：反射相当于一系列解释操作，通知 JVM 要做的事情，性能比直接的 Java 代码要慢很多。

2）安全问题：反射机制破坏了封装性，因为通过反射可以获取并调用类的私有方法和字段。

## 8. 反射的经典应用场景

反射在我们实际编程中其实并不会直接大量的使用，但是实际上有很多设计都与反射机制有关，比如：

- 动态代理机制
- 使用 JDBC 连接数据库
- Spring / Hibernate 框架（实际上是因为使用了动态代理，所以才和反射机制有关）

> 为什么说动态代理使用了反射机制，下篇文章会给出详细解释。

### JDBC 连接数据库

在 JDBC 的操作中，如果要想进行数据库的连接，则必须按照以下几步完成：

- 通过 `Class.forName()` 加载数据库的驱动程序 （通过反射加载）
- 通过 `DriverManager` 类连接数据库，参数包含数据库的连接地址、用户名、密码
- 通过 `Connection` 接口接收连接
- 关闭连接

```java
public static void main(String[] args) throws Exception {  
        Connection con = null; // 数据库的连接对象  
        // 1. 通过反射加载驱动程序
        Class.forName("com.mysql.jdbc.Driver"); 
        // 2. 连接数据库  
        con = DriverManager.getConnection(
            "jdbc:mysql://localhost:3306/test","root","root"); 
        // 3. 关闭数据库连接
        con.close(); 
}
```

### Spring 框架

反射机制是 Java 框架设计的灵魂，框架的内部都已经封装好了，我们自己基本用不着写。典型的除了Hibernate 之外，还有 Spring 也用到了很多反射机制，最典型的就是 Spring 通过 xml 配置文件装载 Bean（创建对象），也就是 **Spring 的 IoC**，过程如下：

- 加载配置文件，获取 Spring 容器
- 使用反射机制，根据传入的字符串获得某个类的 Class 实例

```java
// 获取 Spring 的 IoC 容器，并根据 id 获取对象
public static void main(String[] args) {
    // 1.使用 ApplicationContext 接口加载配置文件，获取 spring 容器
    ApplicationContext ac = new ClassPathXmlApplicationContext("spring.xml");
    // 2. 使用反射机制，根据这个字符串获得某个类的 Class 实例
    IAccountService aService = (IAccountService) ac.getBean("accountServiceImpl");
    System.out.println(aService);
}
```

另外，**Spring AOP 由于使用了动态代理，所以也使用了反射机制**，这点我会在 Spring 的系列文章中详细解释。

## 📚 References

- 《Java 核心技术 - 卷 1 基础知识 - 第 10 版》
- 《Thinking In Java（Java 编程思想）- 第 4 版》
- 敬业的小马哥 — Java 基础之反射：https://blog.csdn.net/sinat_38259539/article/details/71799078