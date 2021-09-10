<p align="center">
    <img width="280px" src="image/konglong/m16.png" >
</p>

# Spark（16）SparkSQL的自定义函数UDF

在Spark中，也支持Hive中的自定义函数。自定义函数大致可以分为三种：

- UDF(User-Defined-Function)，即最基本的自定义函数，类似to_char,to_date等
- UDAF（User- Defined Aggregation Funcation），用户自定义聚合函数，类似在group by之后使用的sum,avg等
- UDTF(User-Defined Table-Generating Functions),用户自定义生成函数，有点像stream里面的flatMap

自定义一个UDF函数需要继承UserDefinedAggregateFunction类，并实现其中的8个方法

示例

```scala
import org.apache.spark.sql.Row
import org.apache.spark.sql.expressions.{MutableAggregationBuffer, UserDefinedAggregateFunction}
import org.apache.spark.sql.types.{DataType, StringType, StructField, StructType}

object GetDistinctCityUDF extends UserDefinedAggregateFunction{
  /**
    * 输入的数据类型
    * */
  override def inputSchema: StructType = StructType(
    StructField("status",StringType,true) :: Nil
  )
  /**
    * 缓存字段类型
    * */
  override def bufferSchema: StructType = {
    StructType(
      Array(
        StructField("buffer_city_info",StringType,true)
      )
    )
  }
/**
  * 输出结果类型
  * */
  override def dataType: DataType = StringType
/**
  * 输入类型和输出类型是否一致
  * */
  override def deterministic: Boolean = true
/**
  * 对辅助字段进行初始化
  * */
  override def initialize(buffer: MutableAggregationBuffer): Unit = {
    buffer.update(0,"")
  }
/**
  *修改辅助字段的值
  * */
  override def update(buffer: MutableAggregationBuffer, input: Row): Unit = {
    //获取最后一次的值
    var last_str = buffer.getString(0)
    //获取当前的值
    val current_str = input.getString(0)
    //判断最后一次的值是否包含当前的值
    if(!last_str.contains(current_str)){
      //判断是否是第一个值，是的话走if赋值，不是的话走else追加
      if(last_str.equals("")){
        last_str = current_str
      }else{
        last_str += "," + current_str
      }
    }
    buffer.update(0,last_str)

  }
/**
  *对分区结果进行合并
  * buffer1是机器hadoop1上的结果
  * buffer2是机器Hadoop2上的结果
  * */
  override def merge(buffer1: MutableAggregationBuffer, buffer2: Row): Unit = {
    var buf1 = buffer1.getString(0)
    val buf2 = buffer2.getString(0)
    //将buf2里面存在的数据而buf1里面没有的数据追加到buf1
    //buf2的数据按照，进行切分
    for(s <- buf2.split(",")){
      if(!buf1.contains(s)){
        if(buf1.equals("")){
          buf1 = s
        }else{
          buf1 += s
        }
      }
    }
    buffer1.update(0,buf1)
  }
/**
  * 最终的计算结果
  * */
  override def evaluate(buffer: Row): Any = {
    buffer.getString(0)
  }
}
```

注册自定义的UDF函数为临时函数

```scala
def main(args: Array[String]): Unit = {
    /**
      * 第一步 创建程序入口
      */
    val conf = new SparkConf().setAppName("AralHotProductSpark")
    val sc = new SparkContext(conf)
    val hiveContext = new HiveContext(sc)　　//注册成为临时函数
    hiveContext.udf.register("get_distinct_city",GetDistinctCityUDF)
　　//注册成为临时函数
    hiveContext.udf.register("get_product_status",(str:String) =>{
      var status = 0
      for(s <- str.split(",")){
        if(s.contains("product_status")){
          status = s.split(":")(1).toInt
        }
      }
    })
}
```