<p align="center">
    <img width="280px" src="image/konglong/m4.png" >
</p>

# HBase（5）HBase-过滤器

## 过滤器（Filter）

　　基础API中的查询操作在面对大量数据的时候是非常苍白的，这里Hbase提供了高级的查询方法：Filter。Filter可以根据簇、列、版本等更多的条件来对数据进行过滤，基于Hbase本身提供的三维有序（主键有序、列有序、版本有序），这些Filter可以高效的完成查询过滤的任务。带有Filter条件的RPC查询请求会把Filter分发到各个RegionServer，是一个服务器端（Server-side）的过滤器，这样也可以降低网络传输的压力。

　　要完成一个过滤的操作，至少需要两个参数。**一个是抽象的操作符**，Hbase提供了枚举类型的变量来表示这些抽象的操作符：LESS/LESS_OR_EQUAL/EQUAL/NOT_EUQAL等；**另外一个就是具体的比较器（Comparator）**，代表具体的比较逻辑，如果可以提高字节级的比较、字符串级的比较等。有了这两个参数，我们就可以清晰的定义筛选的条件，过滤数据。

**抽象操作符（比较运算符）**

> LESS <
>
> LESS_OR_EQUAL <=
>
> EQUAL =
>
> NOT_EQUAL <>
>
> GREATER_OR_EQUAL >=
>
> GREATER >
>
> NO_OP 排除所有

**比较器（指定比较机制）**

> BinaryComparator 按字节索引顺序比较指定字节数组，采用 Bytes.compareTo(byte[])
>
> BinaryPrefixComparator 跟前面相同，只是比较左端的数据是否相同
>
> NullComparator 判断给定的是否为空
>
> BitComparator 按位比较
>
> RegexStringComparator 提供一个正则的比较器，仅支持 EQUAL 和非 EQUAL
>
> SubstringComparator 判断提供的子串是否出现在 value 中

## HBase过滤器的分类

### 比较过滤器

#### 1、行键过滤器 RowFilter

```
Filter rowFilter = new RowFilter(CompareOp.GREATER, new BinaryComparator("95007".getBytes()));
scan.setFilter(rowFilter);
```

```java
public class HbaseFilterTest {

    private static final String ZK_CONNECT_KEY = "hbase.zookeeper.quorum";
    private static final String ZK_CONNECT_VALUE = "hadoop1:2181,hadoop2:2181,hadoop3:2181";

    private static Connection conn = null;
    private static Admin admin = null;

    public static void main(String[] args) throws Exception {

        Configuration conf = HBaseConfiguration.create();
        conf.set(ZK_CONNECT_KEY, ZK_CONNECT_VALUE);
        conn = ConnectionFactory.createConnection(conf);
        admin = conn.getAdmin();
        Table table = conn.getTable(TableName.valueOf("student"));

        Scan scan = new Scan();

        Filter rowFilter = new RowFilter(CompareOp.GREATER, new BinaryComparator("95007".getBytes()));
        scan.setFilter(rowFilter);
        ResultScanner resultScanner = table.getScanner(scan);
        for(Result result : resultScanner) {
            List<Cell> cells = result.listCells();
            for(Cell cell : cells) {
                System.out.println(cell);
            }
        }


    }
```

运行结果部分截图

![img](https://images2018.cnblogs.com/blog/1228818/201803/1228818-20180331142700638-33411889.png)

#### 2、列簇过滤器 FamilyFilter

```
Filter familyFilter = new FamilyFilter(CompareOp.EQUAL, new BinaryComparator("info".getBytes()));
scan.setFilter(familyFilter);
```

```java
public class HbaseFilterTest {

    private static final String ZK_CONNECT_KEY = "hbase.zookeeper.quorum";
    private static final String ZK_CONNECT_VALUE = "hadoop1:2181,hadoop2:2181,hadoop3:2181";

    private static Connection conn = null;
    private static Admin admin = null;

    public static void main(String[] args) throws Exception {

        Configuration conf = HBaseConfiguration.create();
        conf.set(ZK_CONNECT_KEY, ZK_CONNECT_VALUE);
        conn = ConnectionFactory.createConnection(conf);
        admin = conn.getAdmin();
        Table table = conn.getTable(TableName.valueOf("student"));

        Scan scan = new Scan();

        Filter familyFilter = new FamilyFilter(CompareOp.EQUAL, new BinaryComparator("info".getBytes()));
        scan.setFilter(familyFilter);
        ResultScanner resultScanner = table.getScanner(scan);
        for(Result result : resultScanner) {
            List<Cell> cells = result.listCells();
            for(Cell cell : cells) {
                System.out.println(cell);
            }
        }


    }


}
```



![img](https://images2018.cnblogs.com/blog/1228818/201803/1228818-20180331142550951-1320138981.png)

#### 3、列过滤器 QualifierFilter

```
Filter qualifierFilter = new QualifierFilter(CompareOp.EQUAL, new BinaryComparator("name".getBytes()));
scan.setFilter(qualifierFilter);
```

```java
public class HbaseFilterTest {

    private static final String ZK_CONNECT_KEY = "hbase.zookeeper.quorum";
    private static final String ZK_CONNECT_VALUE = "hadoop1:2181,hadoop2:2181,hadoop3:2181";

    private static Connection conn = null;
    private static Admin admin = null;

    public static void main(String[] args) throws Exception {

        Configuration conf = HBaseConfiguration.create();
        conf.set(ZK_CONNECT_KEY, ZK_CONNECT_VALUE);
        conn = ConnectionFactory.createConnection(conf);
        admin = conn.getAdmin();
        Table table = conn.getTable(TableName.valueOf("student"));

        Scan scan = new Scan();

        Filter qualifierFilter = new QualifierFilter(CompareOp.EQUAL, new BinaryComparator("name".getBytes()));
        scan.setFilter(qualifierFilter);
        ResultScanner resultScanner = table.getScanner(scan);
        for(Result result : resultScanner) {
            List<Cell> cells = result.listCells();
            for(Cell cell : cells) {
                System.out.println(cell);
            }
        }


    }


}
```

 ![img](https://images2018.cnblogs.com/blog/1228818/201803/1228818-20180331142440045-1822520821.png)

#### 4、值过滤器 ValueFilter

```
Filter valueFilter = new ValueFilter(CompareOp.EQUAL, new SubstringComparator("男"));
scan.setFilter(valueFilter);
```

```java
public class HbaseFilterTest {

    private static final String ZK_CONNECT_KEY = "hbase.zookeeper.quorum";
    private static final String ZK_CONNECT_VALUE = "hadoop1:2181,hadoop2:2181,hadoop3:2181";

    private static Connection conn = null;
    private static Admin admin = null;

    public static void main(String[] args) throws Exception {

        Configuration conf = HBaseConfiguration.create();
        conf.set(ZK_CONNECT_KEY, ZK_CONNECT_VALUE);
        conn = ConnectionFactory.createConnection(conf);
        admin = conn.getAdmin();
        Table table = conn.getTable(TableName.valueOf("student"));

        Scan scan = new Scan();

        Filter valueFilter = new ValueFilter(CompareOp.EQUAL, new SubstringComparator("男"));
        scan.setFilter(valueFilter);
        ResultScanner resultScanner = table.getScanner(scan);
        for(Result result : resultScanner) {
            List<Cell> cells = result.listCells();
            for(Cell cell : cells) {
                System.out.println(cell);
            }
        }


    }


}
```

 ![img](https://images2018.cnblogs.com/blog/1228818/201803/1228818-20180331142810385-1010448988.png)

#### 5、时间戳过滤器 TimestampsFilter

```
List<Long> list = new ArrayList<>();
list.add(1522469029503l);
TimestampsFilter timestampsFilter = new TimestampsFilter(list);
scan.setFilter(timestampsFilter);
```

```java
public class HbaseFilterTest {

    private static final String ZK_CONNECT_KEY = "hbase.zookeeper.quorum";
    private static final String ZK_CONNECT_VALUE = "hadoop1:2181,hadoop2:2181,hadoop3:2181";

    private static Connection conn = null;
    private static Admin admin = null;

    public static void main(String[] args) throws Exception {

        Configuration conf = HBaseConfiguration.create();
        conf.set(ZK_CONNECT_KEY, ZK_CONNECT_VALUE);
        conn = ConnectionFactory.createConnection(conf);
        admin = conn.getAdmin();
        Table table = conn.getTable(TableName.valueOf("student"));

        Scan scan = new Scan();

        List<Long> list = new ArrayList<>();
        list.add(1522469029503l);
        TimestampsFilter timestampsFilter = new TimestampsFilter(list);
        scan.setFilter(timestampsFilter);
        ResultScanner resultScanner = table.getScanner(scan);
        for(Result result : resultScanner) {
            List<Cell> cells = result.listCells();
            for(Cell cell : cells) {
                System.out.println(Bytes.toString(cell.getRow()) + "\t" + Bytes.toString(cell.getFamily()) + "\t" + Bytes.toString(cell.getQualifier())
                + "\t" + Bytes.toString(cell.getValue()) + "\t" + cell.getTimestamp());
            }
        }


    }


}
```

![img](https://images2018.cnblogs.com/blog/1228818/201803/1228818-20180331143422193-1859430510.png)



### 专用过滤器

#### 1、单列值过滤器 SingleColumnValueFilter ----会返回满足条件的整行

```
SingleColumnValueFilter singleColumnValueFilter = new SingleColumnValueFilter(
                "info".getBytes(), //列簇
                "name".getBytes(), //列
                CompareOp.EQUAL, 
                new SubstringComparator("刘晨"));
//如果不设置为 true，则那些不包含指定 column 的行也会返回
singleColumnValueFilter.setFilterIfMissing(true);
scan.setFilter(singleColumnValueFilter);
```

```java
public class HbaseFilterTest2 {

    private static final String ZK_CONNECT_KEY = "hbase.zookeeper.quorum";
    private static final String ZK_CONNECT_VALUE = "hadoop1:2181,hadoop2:2181,hadoop3:2181";

    private static Connection conn = null;
    private static Admin admin = null;

    public static void main(String[] args) throws Exception {

        Configuration conf = HBaseConfiguration.create();
        conf.set(ZK_CONNECT_KEY, ZK_CONNECT_VALUE);
        conn = ConnectionFactory.createConnection(conf);
        admin = conn.getAdmin();
        Table table = conn.getTable(TableName.valueOf("student"));

        Scan scan = new Scan();

        SingleColumnValueFilter singleColumnValueFilter = new SingleColumnValueFilter(
                "info".getBytes(),
                "name".getBytes(),
                CompareOp.EQUAL,
                new SubstringComparator("刘晨"));
        singleColumnValueFilter.setFilterIfMissing(true);

        scan.setFilter(singleColumnValueFilter);
        ResultScanner resultScanner = table.getScanner(scan);
        for(Result result : resultScanner) {
            List<Cell> cells = result.listCells();
            for(Cell cell : cells) {
                System.out.println(Bytes.toString(cell.getRow()) + "\t" + Bytes.toString(cell.getFamily()) + "\t" + Bytes.toString(cell.getQualifier())
                + "\t" + Bytes.toString(cell.getValue()) + "\t" + cell.getTimestamp());
            }
        }


    }


}
```

![img](https://images2018.cnblogs.com/blog/1228818/201803/1228818-20180331145327073-421109265.png)

#### 2、单列值排除器 SingleColumnValueExcludeFilter 

```
SingleColumnValueExcludeFilter singleColumnValueExcludeFilter = new SingleColumnValueExcludeFilter(
                "info".getBytes(), 
                "name".getBytes(), 
                CompareOp.EQUAL, 
                new SubstringComparator("刘晨"));
singleColumnValueExcludeFilter.setFilterIfMissing(true);
        
scan.setFilter(singleColumnValueExcludeFilter);
```

```java
public class HbaseFilterTest2 {

    private static final String ZK_CONNECT_KEY = "hbase.zookeeper.quorum";
    private static final String ZK_CONNECT_VALUE = "hadoop1:2181,hadoop2:2181,hadoop3:2181";

    private static Connection conn = null;
    private static Admin admin = null;

    public static void main(String[] args) throws Exception {

        Configuration conf = HBaseConfiguration.create();
        conf.set(ZK_CONNECT_KEY, ZK_CONNECT_VALUE);
        conn = ConnectionFactory.createConnection(conf);
        admin = conn.getAdmin();
        Table table = conn.getTable(TableName.valueOf("student"));

        Scan scan = new Scan();

        SingleColumnValueExcludeFilter singleColumnValueExcludeFilter = new SingleColumnValueExcludeFilter(
                "info".getBytes(),
                "name".getBytes(),
                CompareOp.EQUAL,
                new SubstringComparator("刘晨"));
        singleColumnValueExcludeFilter.setFilterIfMissing(true);

        scan.setFilter(singleColumnValueExcludeFilter);
        ResultScanner resultScanner = table.getScanner(scan);
        for(Result result : resultScanner) {
            List<Cell> cells = result.listCells();
            for(Cell cell : cells) {
                System.out.println(Bytes.toString(cell.getRow()) + "\t" + Bytes.toString(cell.getFamily()) + "\t" + Bytes.toString(cell.getQualifier())
                + "\t" + Bytes.toString(cell.getValue()) + "\t" + cell.getTimestamp());
            }
        }


    }


}
```

 ![img](https://images2018.cnblogs.com/blog/1228818/201803/1228818-20180331145530417-602264605.png)

#### 3、前缀过滤器 PrefixFilter----针对行键

```
PrefixFilter prefixFilter = new PrefixFilter("9501".getBytes());
        
scan.setFilter(prefixFilter);
```

```java
public class HbaseFilterTest2 {

    private static final String ZK_CONNECT_KEY = "hbase.zookeeper.quorum";
    private static final String ZK_CONNECT_VALUE = "hadoop1:2181,hadoop2:2181,hadoop3:2181";

    private static Connection conn = null;
    private static Admin admin = null;

    public static void main(String[] args) throws Exception {

        Configuration conf = HBaseConfiguration.create();
        conf.set(ZK_CONNECT_KEY, ZK_CONNECT_VALUE);
        conn = ConnectionFactory.createConnection(conf);
        admin = conn.getAdmin();
        Table table = conn.getTable(TableName.valueOf("student"));

        Scan scan = new Scan();

        PrefixFilter prefixFilter = new PrefixFilter("9501".getBytes());

        scan.setFilter(prefixFilter);
        ResultScanner resultScanner = table.getScanner(scan);
        for(Result result : resultScanner) {
            List<Cell> cells = result.listCells();
            for(Cell cell : cells) {
                System.out.println(Bytes.toString(cell.getRow()) + "\t" + Bytes.toString(cell.getFamily()) + "\t" + Bytes.toString(cell.getQualifier())
                + "\t" + Bytes.toString(cell.getValue()) + "\t" + cell.getTimestamp());
            }
        }


    }


}
```

![img](https://images2018.cnblogs.com/blog/1228818/201803/1228818-20180331150009860-2063523179.png)

#### 4、列前缀过滤器 ColumnPrefixFilter

```
ColumnPrefixFilter columnPrefixFilter = new ColumnPrefixFilter("name".getBytes());
        
scan.setFilter(columnPrefixFilter);
```

```java
public class HbaseFilterTest2 {

    private static final String ZK_CONNECT_KEY = "hbase.zookeeper.quorum";
    private static final String ZK_CONNECT_VALUE = "hadoop1:2181,hadoop2:2181,hadoop3:2181";

    private static Connection conn = null;
    private static Admin admin = null;

    public static void main(String[] args) throws Exception {

        Configuration conf = HBaseConfiguration.create();
        conf.set(ZK_CONNECT_KEY, ZK_CONNECT_VALUE);
        conn = ConnectionFactory.createConnection(conf);
        admin = conn.getAdmin();
        Table table = conn.getTable(TableName.valueOf("student"));

        Scan scan = new Scan();

        ColumnPrefixFilter columnPrefixFilter = new ColumnPrefixFilter("name".getBytes());

        scan.setFilter(columnPrefixFilter);
        ResultScanner resultScanner = table.getScanner(scan);
        for(Result result : resultScanner) {
            List<Cell> cells = result.listCells();
            for(Cell cell : cells) {
                System.out.println(Bytes.toString(cell.getRow()) + "\t" + Bytes.toString(cell.getFamily()) + "\t" + Bytes.toString(cell.getQualifier())
                + "\t" + Bytes.toString(cell.getValue()) + "\t" + cell.getTimestamp());
            }
        }


    }


}
```



![img](https://images2018.cnblogs.com/blog/1228818/201803/1228818-20180331150519676-692732493.png)

#### 5、分页过滤器 PageFilter