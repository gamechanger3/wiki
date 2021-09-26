<p align="center">
    <img width="280px" src="image/konglong/m2.png" >
</p>

# 数据分析常用的Excel函数合集

Excel是我们工作中经常使用的一种工具，对于数据分析来说，这也是处理数据最基础的工具。 本文对数据分析需要用到的函数做了分类，并且有详细的例子说明。

Excel函数分类：关联匹配类、清洗处理类、逻辑运算类、计算统计类、时间 序列类由于篇幅过长，本篇先分享关联匹配类和清洗处理类，其余三个在明 日推文第三条继续分享。

## 关联匹配类

经常性的， 需要的数据不在同一个excel表或同一个excel表不同sheet中， 数据太 多， copy麻烦也不准确， 如何整合呢？ 这类函数就是 用于多表关联或者行列比对时 的场景，而且表越复杂，用得越多。

包 含 函 数 ： VLOOKUP 、 HLOOKUP 、 INDEX 、 MATCH 、 RANK 、 Row 、 Column、Offset

### **1.** **VLOOKUP**

功能：用于查找首列满足条件的元素

语法：=VLOOKUP（要查找的值，要在其中查找值的区域，区域中包含返回值的列 号，精确匹配(0)或近似匹配(1) ）

(1) 单表查找

![img](https://gitee.com/gamechanger1/image/raw/master/wps3.jpg) 

把选手Tian的战队找到之后， 接下来把鼠标放到G8 单元格右下角位置， 出现十字符

号后往下拉，Excel会根据单元格的变化自动填充G9和G10单元格的公式。 (2) 跨多工作表查找

假设我有一个工资表格文件，里面每个部门有一张表，有4个部门对应的部门工资表 和一个需要查询工资的查询表，为方便说明这里的姓名取方便识别的编号，你也可以 用真正的姓名。

![img](https://gitee.com/gamechanger1/image/raw/master/wps4.jpg) 

在查询表中， 要求根据提供的姓名， 从销售~ 人事 4 个工作表中查询该员工的基本工

资。

![img](https://gitee.com/gamechanger1/image/raw/master/wps5.jpg) 

如果，我们知道A1是销售部的，那么公式可以写为：

![img](C:\Users\UML-DEV\AppData\Local\Temp\ksohtml16492\wps6.png)=VLOOKUP(A2,销售!A:C,3,0)

如果，我们知道A1可能在销售或财务表这2个表中，公式可以写为：

![img](C:\Users\UML-DEV\AppData\Local\Temp\ksohtml16492\wps7.png)=IFERROR(VLOOKUP(A2,销售!A:C,3,0),VLOOKUP(A2,财务!A:C,3,0))

意思是，如果在销售表中查找不到(用IFERROR函数判断)，则去财务表中再查找。 如果，我们知道A1可能在销售、财务或服务表中，公式可以再次改为：

![img](C:\Users\UML-DEV\AppData\Local\Temp\ksohtml16492\wps8.png)=IFERROR(VLOOKUP(A2,销售!A:C,3,0),IFERROR(VLOOKUP(A2,财 务!A:C,3,0),VLOOKUP(A2,服务!A:C,3,0)))

如果， 有更多的表， 如本例中 4 个表， 那就一层层的套用下去， 如果 4 个表都查不到 就设置为"无此人信息"：

![img](C:\Users\UML-DEV\AppData\Local\Temp\ksohtml16492\wps9.png)=IFERROR(VLOOKUP(A2,销售!A:C,3,0),IFERROR(VLOOKUP(A2,财 务!A:C,3,0),IFERROR(VLOOKUP(A2,服务!A:C,3,0),IFERROR(VLOOKUP(A2,人 事!A:C,3,0),"无此人信息"))))

 

![img](https://gitee.com/gamechanger1/image/raw/master/wps10.jpg) 

### **2.** HLOOKUP

当 查 找 的 值 位 于 查 找 范 围 的 首 行 ， 并 且 返 回 的 值 在 查 找 范 围 的 第 几 行 ， 可 以 使 用 hlookup 函数

语法：=HLOOKUP（要查找的值，查找的范围，返回的值在查找范围的第几行，精 确匹配(0)或近似匹配(1) ）

区别：HLOOKUP按行查找，返回的值与需要查找的值在同一列上，VLOOKUP按列 查找，返回的值与需要查找的值在同一行上。

![img](https://gitee.com/gamechanger1/image/raw/master/wps11.jpg) 

### **3.** INDEX

在Excel中，除了VLOOKUP函数常用来查找引用外，INDEX函数和MATCH函数组 合也可用来做查找引用工作，这组函数有效弥补了VLOOKUP函数查找目标不在查找 范围数据首列的缺陷。

功能：返回表格或区域中的值

语法：= INDEX(要返回值的单元格区域或数组,所在行,所在列)

![image](https://gitee.com/gamechanger1/image/raw/master/Image_012.jpg) 

### **4.** MATCH

功能：用于返回指定内容在指定区域（某行或者某列）的位置

语法：= MATCH ( 要查找的值， 查找的区域， 查找方式)， 查找方式 0 为等于查找 值，1为小于查找值，-1为大于查找值

![image](https://gitee.com/gamechanger1/image/raw/master/Image_013.png)

### **5.** RANK

功能：求某一个数值在某一区域内的数值排名

语法：=RANK(参与排名的数值, 排名的数值区域, 排名方式-0是降序-1是升序-默认 为0）。

![image](https://gitee.com/gamechanger1/image/raw/master/Image_014.png)

### **6.** Row

功能：返回单元格所在的行

语法：ROW()或ROW(某个单元格)

![image](https://gitee.com/gamechanger1/image/raw/master/Image_015.png)

### **7.** Column

功能：返回单元格所在的列

语法：COLUMN()或COLUMN(某个单元格)

![img](https://gitee.com/gamechanger1/image/raw/master/wps16.jpg) 

### **8.** Offset

功能：从指定的基准位置按行列偏移量返回指定的引用

语法： ＝Offset（ 指定点， 偏移多少行( 正数向下， 负数向上)， 偏移多少列( 正数向 右，负数向左)，返回多少行，返回多少列）

![image](https://gitee.com/gamechanger1/image/raw/master/Image_017.jpg) 

##  清洗处理类

数据处理之前，需要对提取的数据进行初步清洗，如清除字符串空格，合并单元格、 替换、截取字符串、查找字符串出现的位置等。

![img](https://gitee.com/gamechanger1/image/raw/master/wps19.png)清除字符串前后空格：使用Trim

![img](C:\Users\UML-DEV\AppData\Local\Temp\ksohtml16492\wps20.png)合并单元格：使用concatenate

![img](https://gitee.com/gamechanger1/image/raw/master/wps21.png)截取字符串：使用Left/Right/Mid

![img](C:\Users\UML-DEV\AppData\Local\Temp\ksohtml16492\wps22.png)![img](C:\Users\UML-DEV\AppData\Local\Temp\ksohtml16492\wps23.png)![img](C:\Users\UML-DEV\AppData\Local\Temp\ksohtml16492\wps24.png)替换单元格中内容：Replace/Substitute 查找文本在单元格中的位置：Find/Search 获取字符长度：Len/Lenb

![img](https://gitee.com/gamechanger1/image/raw/master/wps25.png)![img](https://gitee.com/gamechanger1/image/raw/master/wps26.png)筛选包含某个条件的 内容：IF+OR+COUNTIF 转换数据类型：VALUE/TEXT

### **1.** Trim

功能：主要用于把单元格内容前后的空格去掉，但并不去除字符之间的空格，如果是 想要去掉所有的空格，需要用substitute函数。

语法：=TRIM(单元格)

![img](https://gitee.com/gamechanger1/image/raw/master/wps27.png)

### **2.** concatenate

语法：=Concatenate(单元格1，单元格2……)

合 并 单 元 格 中 的 内 容 ， 还 有 另 一 种 合 并 方 式 是 & ， 需 要 合 并 的 内 容 过 多 时 ， concatenate效率更快。

![image](https://gitee.com/gamechanger1/image/raw/master/Image_028.png)

### **3.** Left

功能：从左截取字符串

语法：=Left(值所在单元格，截取长度)

![image](https://gitee.com/gamechanger1/image/raw/master/Image_029.png)

### **4.** Right

功能：从右截取字符串

语法：= Right (值所在单元格，截取长度)

![img](https://gitee.com/gamechanger1/image/raw/master/wps30.jpg) 

### **5.** Mid

功能：从中间截取字符串

语法：= Mid(指定字符串，开始位置，截取长度)

![img](https://gitee.com/gamechanger1/image/raw/master/wps31.jpg) 

Text函数表示将数值转化为自己想要的文本格式，语法：

=TEXT(value,format_text)

### **6.** Replace

功能：替换掉单元格的字符串

语法：=Replace（指定字符串，哪个位置开始替换，替换几个字符，替换成什么）

![img](https://gitee.com/gamechanger1/image/raw/master/wps32.jpg) 

### **7.** Substitute

和replace接近， 不同在于 Replace根据位置实现替换 ， 需要提供从第几位开始替 换，替换几位，替换后的新的文本。

而Substitute根据文本内容替换， 需要提供替换的旧文本和新文本， 以及替换第几 个旧文本等。 因此Replace实现固定位置的文本替换， Substitute实现固定文本替 换。

![img](https://gitee.com/gamechanger1/image/raw/master/wps33.png)

![image](https://gitee.com/gamechanger1/image/raw/master/Image_034.jpg) 



### **8.** Find

功能：查找文本位置

语法：=Find（要查找字符，指定字符串，从第几个字符开始查起

![image](https://gitee.com/gamechanger1/image/raw/master/Image_035.jpg) 

 

![image](https://gitee.com/gamechanger1/image/raw/master/Image_036.jpg) 

### **9.** Search

功能：返回一个指定字符或文本字符串在字符串中第一次出现的位置，从左到右查找

语法：=search（要查找的字符，字符所在的文本，从第几个字符开始查找）

Find和Search这两个函数功能几乎相同，实现查找字符所在的位置，区别在于Find 函数精确查找，区分大小写；Search函数模糊查找，不区分大小写。

![img](https://gitee.com/gamechanger1/image/raw/master/wps37.jpg) 

### **10.** Len

功能：返回字符串的字符数 语法：=LEN（字符串)

字符串是指包含数字、字母、符号等的一串字符。

![image](https://gitee.com/gamechanger1/image/raw/master/Image_038.jpg) 

### **11.** Lenb

功能：返回字符串的字节数

区别在于，len是按字符数计算的、lenb是按字节数计算的。数字、字母、英文、标 点符号（半角状态下输入的哦）都是按1计算的，汉字、全角状态下的标点符号，每 个字符按2计算。

![image](https://gitee.com/gamechanger1/image/raw/master/Image_039.jpg) 

综合应用：

![img](https://gitee.com/gamechanger1/image/raw/master/wps40.jpg) 

筛选内容：IF+OR+COUNTIF

=IF(OR(COUNTIF(A1,"*"&{"Python","java"}&"*")),A1,"0")

如果含有字段Python或java中的任何一个则为本身，否则为"0"，* 代表任意内容， 之后就可以通过Excel的筛选功能，把B列的"0"筛选掉。

![image](https://gitee.com/gamechanger1/image/raw/master/Image_041.jpg) 

### **12.** VALUE

功能：将所选区域转为数值类型 13.TEXT

功能：将所选区域转为文本类型

![image](https://gitee.com/gamechanger1/image/raw/master/Image_042.jpg) 



##  逻辑运算类

包括：IF、AND、OR三个函数

### **1.** IF

功能：使用逻辑函数 IF 函数时，如果条件为真，该函数将返回一个值；如果条件为 假，函数将返回另一个值。

语法：=IF(条件, true时返回值, false返回值)

![image](https://gitee.com/gamechanger1/image/raw/master/Image_045.jpg) 

### **2.** AND

功能：逻辑判断，相当于“并”,"&"

语法：全部参数为True，则返回True，经常用于多条件判断。

![img](https://gitee.com/gamechanger1/image/raw/master/wps46.png)

### **3.** OR

功能：逻辑判断，相当于“或”

语法：只要参数有一个True，则返回Ture，经常用于多条件判断。

![img](https://gitee.com/gamechanger1/image/raw/master/wps47.png)

##  计算统计类

在利用excel表格统计数据时， 常常需要使用各种excel自带的公式， 也是最常使用

的一类，重要性不言而喻，不过excel都自带快捷功能。

 

![img](C:\Users\UML-DEV\AppData\Local\Temp\ksohtml16492\wps49.png)MIN函数：找到某区域中的最小值

![img](C:\Users\UML-DEV\AppData\Local\Temp\ksohtml16492\wps50.png)MAX函数：找到某区域中的最大值

![img](https://gitee.com/gamechanger1/image/raw/master/wps51.png)AVERAGE函数：计算某区域中的平均值

![img](C:\Users\UML-DEV\AppData\Local\Temp\ksohtml16492\wps52.png)COUNT函数： 计算某区域中包含数字的单元格的数目

![img](https://gitee.com/gamechanger1/image/raw/master/wps53.png)![img](C:\Users\UML-DEV\AppData\Local\Temp\ksohtml16492\wps54.png)COUNTIF函数：计算某个区域中满足给定条件的单元格数目 COUNTIFS函数：统计一组给定条件所指定的单元格数

![img](C:\Users\UML-DEV\AppData\Local\Temp\ksohtml16492\wps55.png)![img](https://gitee.com/gamechanger1/image/raw/master/wps56.png)SUM函数：计算单元格区域中所有数值的和 SUMIF函数：对满足条件的单元格求和

![img](C:\Users\UML-DEV\AppData\Local\Temp\ksohtml16492\wps57.png)![img](https://gitee.com/gamechanger1/image/raw/master/wps58.png)SUMPRODUCT函数：返回相应的数组或区域乘积的和 STDEV函数：求标准差

![img](C:\Users\UML-DEV\AppData\Local\Temp\ksohtml16492\wps59.png)SUBTOTAL函数：汇总型函数，将平均值、计数、最大最小、相乘、标准差、 求和、方差等参数化

![img](C:\Users\UML-DEV\AppData\Local\Temp\ksohtml16492\wps60.png)INT/ROUND函数：取整函数，int向下取整，round按小数位取数

### ![img](https://gitee.com/gamechanger1/image/raw/master/wps61.png)MOD函数：取余 1. MIN

功能：找到某区域中的最小值

![img](https://gitee.com/gamechanger1/image/raw/master/wps62.jpg) 

### **2.** MAX

功能：找到某区域中的最大值

![img](https://gitee.com/gamechanger1/image/raw/master/wps63.png)

### **3.** AVERAGE

功能：计算某区域中的平均值

![img](https://gitee.com/gamechanger1/image/raw/master/wps64.png)

### **4.** COUNT

功能：计算纯数字的单元格的个数

![img](https://gitee.com/gamechanger1/image/raw/master/wps65.png)

**5.** **COUNTIF**

功能：计算某个区域中满足给定条件的单元格数目 语法：=COUNTIF(单元格1: 单元格2 ,条件)

![img](https://gitee.com/gamechanger1/image/raw/master/wps66.png)

### **6.** COUNTIFS

功能：统计一组给定条件所指定的单元格数

语法：=COUNTIFS( 第一个条件区域， 第一个对应的条件， 第二个条件区域， 第二 个对应的条件，第N个条件区域，第N个对应的条件)

![img](https://gitee.com/gamechanger1/image/raw/master/wps67.png)

### **7.** SUM

计算单元格区域中所有数值的和

![img](https://gitee.com/gamechanger1/image/raw/master/wps68.png)

**8.** **SUMIF**

功能：求满足条件的单元格和

语法：=SUMIF(单元格1: 单元格2 ,条件,单元格3: 单元格4)

![img](https://gitee.com/gamechanger1/image/raw/master/wps69.png)

### **9.** SUMPRODUCT

功能：返回相应的数组或区域乘积的和

语法：=SUMPRODUCT(单元格1: 单元格2 ,单元格3: 单元格4)

![img](https://gitee.com/gamechanger1/image/raw/master/wps70.png)

### **10.** Stdev

统计型函数，求标准差，衡量离散程度。

![img](https://gitee.com/gamechanger1/image/raw/master/wps71.jpg) 

**11.** **Subtotal**

语法：=Subtotal（参数，区域）

汇总型函数，将平均值、计数、最大最小、相乘、标准差、求和、方差等参数化，换

言之，只要会了这个函数，上面的都可以抛弃掉了。

为 1 到 11 （ 包含隐藏值） 或 101 到 111 （ 忽略隐藏值） 之间的数字， 指定使用何 种函数在列表中进行分类汇总计算。

AVERAGE（算术平均值） COUNT（数值个数）

COUNTA（非空单元格数量） MAX（最大值）

MIN（最小值）

PRODUCT（括号内所有数据的乘积） STDEV（估算样本的标准偏差）

STDEVP（返回整个样本总体的标准偏差） SUM（求和）

VAR（计算基于给定样本的方差）

VARP（计算基于整个样本总体的方差）

![img](https://gitee.com/gamechanger1/image/raw/master/wps72.jpg) 

### **12.** Int／Round

取整函数，int取整(去掉小数)，round按小数位取数(四舍五入)。 语法：ROUND(数值, 位数)

round(3.1415,2)=3.14 ;

round(3.1415,1)=3.1

![img](https://gitee.com/gamechanger1/image/raw/master/wps73.png)

![img](https://gitee.com/gamechanger1/image/raw/master/wps74.png)

![img](https://gitee.com/gamechanger1/image/raw/master/wps75.png)

MOD

![img](https://gitee.com/gamechanger1/image/raw/master/wps76.png)

## 时间序列类

专门用于处理时间格式以及转换。

TODAY函数：返回今天的日期，动态函数。 NOW函数：返回当前的时间，动态函数。

YEAR函数：返回日期的年份。

MONTH函数：返回日期的月份。

DAY函数：返回以序列数表示的某日期的天数。

WEEKDAY函数：返回对应于某个日期的一周中的第几天。 Datedif函数：计算两个日期之间相隔的天数、月数或年数。

### **1.** TODAY

功能：返回今天的日期，动态函数

语法：=TODAY()，如不显示应该是单元格格式问题，单元格格式应是常规或日期型

![img](https://gitee.com/gamechanger1/image/raw/master/wps77.jpg) 

### **2.** NOW

功能：返回当前的日期和时间，动态函数 语法：=NOW()

![img](https://gitee.com/gamechanger1/image/raw/master/wps78.jpg) 

### **3.** YEAR

功能：返回日期的年份 语法：=YEAR(日期)

![img](https://gitee.com/gamechanger1/image/raw/master/wps79.png) 

### **4.** MONTH

功能：返回日期的月份

语法：=MONTH(日期)

![img](https://gitee.com/gamechanger1/image/raw/master/wps80.jpg) 

### **5.** DAY

功能：返回以序列数表示的某日期的天数 语法：=DAY(日期)

![img](https://gitee.com/gamechanger1/image/raw/master/wps81.jpg) 

### **6.** WEEKDAY

功能：返回对应于某个日期的一周中的第几天。默认情况下， 1（星期日）到 7（星 期六）范围内的整数。

语法：=Weekday(指定时间，参数)，参数设为2，则星期一为1，星期日为7

![img](https://gitee.com/gamechanger1/image/raw/master/wps82.png)

### **7.** Datedif

功能：计算两个日期之间相隔的天数、月数或年数 语法：=Datedif（开始日期，结束日期，参数）

参数3：为所需信息的返回时间单位代码。各代码含义如下： "y"返回时间段中的整年数

"m”返回时间段中的整月数

"d"返回时间段中的天数

"md”参数1和2的天数之差，忽略年和月 "ym“参数1和2的月数之差，忽略年和日

"yd”参数1和2的天数之差，忽略年。按照月、日计算天数

![img](https://gitee.com/gamechanger1/image/raw/master/wps83.jpg) 



 