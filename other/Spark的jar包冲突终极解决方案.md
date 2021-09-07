<p align="center">
    <img width="280px" src="image/konglong/m2.png" >
</p>

# Spark的jar包冲突终极解决方案

## Spark 依赖包来源

我们知道Spark application运行加载依赖有三个地方：

- SystemClasspath -- Spark安装时候提供的依赖包   【SystemClassPath】
- Spark-submit --jars 提交的依赖包                【UserClassPath】
- Spark-submit app.jar或者shadowJar打的jar        【UserClassPath】

 

## Spark 依赖包默认优先级

通过测试发现class的默认加载顺序如下：

\1. SystemClasspath -- Spark安装时候提供的依赖包

\2. UserClassPath  -- Spark-submit --jars 提交的依赖包 或用户的app.jar

 

SystemClasspath 系统安装的包，默认优先使用环境的包，这样更加稳定安全。

spark-submit --jars 在默认spark环境里没有需要的包时，自己上传提供。

 

*观察方法： spark-submit 的时候添加参数*

 

*测试环境：* 

> SPARK2-2.2.0.cloudera1-1.cdh5.12.0.p0.142354
>
> 1.8.0_171 (Oracle Corporation)

 

查看环境包依赖情况： Environment tab里搜索想要的依赖包。

![img](https://img-blog.csdnimg.cn/2020111117383756.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2Fkb3JlY2hlbg==,size_16,color_FFFFFF,t_70)

 

#  

## 依赖包冲突解决方案

 

- spark.{driver/executor}.userClassPathFirst  

该配置的意思是优先使用用户路径下的依赖包。通常是系统默认提供的依赖包版本较低，需要使用用户自己提供的依赖包，使用该选项可以很好的解决90%的问题。

 

- spark.{driver/executor}.extraClassPath 

某些时候同一个类的两个不同版本被加载进ClassLoader，link的时候发生错误。这种冲突就需要使用extraClassPath手动指定版本。该选项指定的依赖包具有最高优先级。缺点是只能指定一个包。

 

注意：extraClassPath=jarName 不用给path。 例如：

> --jars snappy-java-version.jar  \
>  --conf "spark.driver.extraClassPath=snappy-java-version.jar" \
>  --conf "spark.executor.extraClassPath=snappy-java-version.jar" \

 

比如曾经的例子 snappy-java包冲突：

https://blog.csdn.net/adorechen/article/details/80109625

使用userClassPathFirst后，报如下错误：

> org.xerial.snappy.SnappyNative.maxCompressedLength(I)I
>  java.lang.UnsatisfiedLinkError: org.xerial.snappy.SnappyNative.maxCompressedLength(I)I

通过 “--driver-java-options -verbose:class”  查看加载的类：SnappyNative分别从两个包snappy-java-1.1.4.jar [UserClassPath],  snappy-java-1.0.4.1.jar [SystemClassPath] 加载进入ClassLoader，

![img](https://img-blog.csdnimg.cn/20201112153340856.png)

![img](https://img-blog.csdnimg.cn/20201112153306550.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2Fkb3JlY2hlbg==,size_16,color_FFFFFF,t_70)

link的时候不能确定使用哪个版本就报上面的错误。这种情况就必须使用“extraClassPath”来解决。

## 总结

A）在我们提交一个spark2 程序时，系统没有的包--jars 提交；

B）在需要使用系统中已有的包的不同版本时，使用spark.{driver/executor}.userClassPathFirst/extraClassPath来指定。

