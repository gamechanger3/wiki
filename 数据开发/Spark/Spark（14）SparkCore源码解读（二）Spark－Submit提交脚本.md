<p align="center">
    <img width="280px" src="image/konglong/m14.png" >
</p>

# Spark（14）SparkCore源码解读（二）Spark－Submit提交脚本

## 一、概述

上一篇主要是介绍了spark启动的一些脚本，这篇主要分析一下Spark源码中提交任务脚本的处理逻辑，从spark-submit一步步深入进去看看任务提交的整体流程,首先看一下整体的流程概要图：
![img](https://images2018.cnblogs.com/blog/1228818/201805/1228818-20180502085945492-811482545.png)

## 二、源码解读

2.1　spark-submit

```shell
# -z是检查后面变量是否为空（空则真） shell可以在双引号之内引用变量，单引号不可
#这一步作用是检查SPARK_HOME变量是否为空，为空则执行then后面程序
#source命令： source filename作用在当前bash环境下读取并执行filename中的命令
#$0代表shell脚本文件本身的文件名，这里即使spark-submit
#dirname用于取得脚本文件所在目录 dirname $0取得当前脚本文件所在目录
#$(命令)表示返回该命令的结果
#故整个if语句的含义是：如果SPARK_HOME变量没有设置值，则执行当前目录下的find-spark-home脚本文件，设置SPARK_HOME值
if [ -z "${SPARK_HOME}" ]; then
  source "$(dirname "$0")"/find-spark-home
fi

# disable randomized hash for string in Python 3.3+
export PYTHONHASHSEED=0
#执行spark-class脚本，传递参数org.apache.spark.deploy.SparkSubmit 和"$@"
#这里$@表示之前spark-submit接收到的全部参数
exec "${SPARK_HOME}"/bin/spark-class org.apache.spark.deploy.SparkSubmit "$@"
```

所以spark-submit脚本的整体逻辑就是： 
首先 检查SPARK_HOME是否设置；if 已经设置 执行spark-class文件 否则加载执行find-spark-home文件 

### 2.2　find-spark-home

```shell
#定义一个变量用于后续判断是否存在定义SPARK_HOME的python脚本文件
FIND_SPARK_HOME_PYTHON_SCRIPT="$(cd "$(dirname "$0")"; pwd)/find_spark_home.py"

# Short cirtuit if the user already has this set.
##如果SPARK_HOME为不为空值，成功退出程序
if [ ! -z "${SPARK_HOME}" ]; then
   exit 0
# -f用于判断这个文件是否存在并且是否为常规文件，是的话为真，这里不存在为假，执行下面语句，给SPARK_HOME变量赋值
elif [ ! -f "$FIND_SPARK_HOME_PYTHON_SCRIPT" ]; then
  # If we are not in the same directory as find_spark_home.py we are not pip installed so we don't
  # need to search the different Python directories for a Spark installation.
  # Note only that, if the user has pip installed PySpark but is directly calling pyspark-shell or
  # spark-submit in another directory we want to use that version of PySpark rather than the
  # pip installed version of PySpark.
  export SPARK_HOME="$(cd "$(dirname "$0")"/..; pwd)"
else
  # We are pip installed, use the Python script to resolve a reasonable SPARK_HOME
  # Default to standard python interpreter unless told otherwise
  if [[ -z "$PYSPARK_DRIVER_PYTHON" ]]; then
     PYSPARK_DRIVER_PYTHON="${PYSPARK_PYTHON:-"python"}"
  fi
  export SPARK_HOME=$($PYSPARK_DRIVER_PYTHON "$FIND_SPARK_HOME_PYTHON_SCRIPT")
fi
```

可以看到，如果事先用户没有设定SPARK_HOME的值，这里程序也会自动设置并且将其注册为环境变量，供后面程序使用

当SPARK_HOME的值设定完成之后，就会执行Spark-class文件，这也是我们分析的重要部分，源码如下：

### 2.3　spark-class

```shell
#!/usr/bin/env bash
#依旧是检查设置SPARK_HOME的值
if [ -z "${SPARK_HOME}" ]; then
  source "$(dirname "$0")"/find-spark-home
fi
#执行load-spark-env.sh脚本文件，主要目的在于加载设定一些变量值
#设定spark-env.sh中的变量值到环境变量中，供后续使用
#设定scala版本变量值
. "${SPARK_HOME}"/bin/load-spark-env.sh

# Find the java binary
#检查设定java环境值
#-n代表检测变量长度是否为0，不为0时候为真
#如果已经安装Java没有设置JAVA_HOME,command -v java返回的值为${JAVA_HOME}/bin/java
if [ -n "${JAVA_HOME}" ]; then
  RUNNER="${JAVA_HOME}/bin/java"
else
  if [ "$(command -v java)" ]; then
    RUNNER="java"
  else
    echo "JAVA_HOME is not set" >&2
    exit 1
  fi
fi

# Find Spark jars.
#-d检测文件是否为目录，若为目录则为真
#设置一些关联Class文件
if [ -d "${SPARK_HOME}/jars" ]; then
  SPARK_JARS_DIR="${SPARK_HOME}/jars"
else
  SPARK_JARS_DIR="${SPARK_HOME}/assembly/target/scala-$SPARK_SCALA_VERSION/jars"
fi

if [ ! -d "$SPARK_JARS_DIR" ] && [ -z "$SPARK_TESTING$SPARK_SQL_TESTING" ]; then
  echo "Failed to find Spark jars directory ($SPARK_JARS_DIR)." 1>&2
  echo "You need to build Spark with the target \"package\" before running this program." 1>&2
  exit 1
else
  LAUNCH_CLASSPATH="$SPARK_JARS_DIR/*"
fi

# Add the launcher build dir to the classpath if requested.
if [ -n "$SPARK_PREPEND_CLASSES" ]; then
  LAUNCH_CLASSPATH="${SPARK_HOME}/launcher/target/scala-$SPARK_SCALA_VERSION/classes:$LAUNCH_CLASSPATH"
fi

# For tests
if [[ -n "$SPARK_TESTING" ]]; then
  unset YARN_CONF_DIR
  unset HADOOP_CONF_DIR
fi

# The launcher library will print arguments separated by a NULL character, to allow arguments with
# characters that would be otherwise interpreted by the shell. Read that in a while loop, populating
# an array that will be used to exec the final command.
#
# The exit code of the launcher is appended to the output, so the parent shell removes it from the
# command array and checks the value to see if the launcher succeeded.
#执行类文件org.apache.spark.launcher.Main，返回解析后的参数
build_command() {
  "$RUNNER" -Xmx128m -cp "$LAUNCH_CLASSPATH" org.apache.spark.launcher.Main "$@"
  printf "%d\0" $?
}

# Turn off posix mode since it does not allow process substitution
#将build_command方法解析后的参数赋给CMD
set +o posix
CMD=()
while IFS= read -d '' -r ARG; do
  CMD+=("$ARG")
done < <(build_command "$@")

COUNT=${#CMD[@]}
LAST=$((COUNT - 1))
LAUNCHER_EXIT_CODE=${CMD[$LAST]}

# Certain JVM failures result in errors being printed to stdout (instead of stderr), which causes
# the code that parses the output of the launcher to get confused. In those cases, check if the
# exit code is an integer, and if it's not, handle it as a special error case.
if ! [[ $LAUNCHER_EXIT_CODE =~ ^[0-9]+$ ]]; then
  echo "${CMD[@]}" | head -n-1 1>&2
  exit 1
fi

if [ $LAUNCHER_EXIT_CODE != 0 ]; then
  exit $LAUNCHER_EXIT_CODE
fi

CMD=("${CMD[@]:0:$LAST}")
#执行CMD中的某个参数类org.apache.spark.deploy.SparkSubmit
exec "${CMD[@]}"
```

spark-class文件的执行逻辑稍显复杂，总体上应该是这样的：

检查SPARK_HOME的值----》执行load-spark-env.sh文件，设定一些需要用到的环境变量，如scala环境值，这其中也加载了spark-env.sh文件-------》检查设定java的执行路径变量值-------》寻找spark jars,设定一些引用相关类的位置变量------》执行类文件org.apache.spark.launcher.Main，返回解析后的参数给CMD-------》判断解析参数是否正确（代表了用户设置的参数是否正确）--------》正确的话执行org.apache.spark.deploy.SparkSubmit这个类

### 2.4　SparkSubmit 

2.1最后提交语句，D:\src\spark-2.3.0\core\src\main\scala\org\apache\spark\deploy\SparkSubmit.scala

```
exec "${SPARK_HOME}"/bin/spark-class org.apache.spark.deploy.SparkSubmit "$@"
```

```scala
override def main(args: Array[String]): Unit = {
    // Initialize logging if it hasn't been done yet. Keep track of whether logging needs to
    // be reset before the application starts.
    val uninitLog = initializeLogIfNecessary(true, silent = true)
    //拿到submit脚本传入的参数
    val appArgs = new SparkSubmitArguments(args)
    if (appArgs.verbose) {
      // scalastyle:off println
      printStream.println(appArgs)
      // scalastyle:on println
    }
    //根据传入的参数匹配对应的执行方法
    appArgs.action match {
        //根据传入的参数提交命令
      case SparkSubmitAction.SUBMIT => submit(appArgs, uninitLog)
        //只有standalone和mesos集群模式才触发
      case SparkSubmitAction.KILL => kill(appArgs)
      //只有standalone和mesos集群模式才触发
      case SparkSubmitAction.REQUEST_STATUS => requestStatus(appArgs)
    }
  }
```

#### 2.4.1　submit十分关键，主要分为两步骤

（1）调用prepareSubmitEnvironment

（2）调用doRunMain

![img](https://images2018.cnblogs.com/blog/1228818/201805/1228818-20180502185922203-694329056.png)

 