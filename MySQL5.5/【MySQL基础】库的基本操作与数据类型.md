**1.数据库服务器、数据库和表的关系**

<font color="blue">图解</font>
![这里写图片描述](http://img.blog.csdn.net/20170414093537772?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYmFpeWVfeGluZw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

<br/>
**2.创建数据库**

（1）普通的定义方式：

```MySQL
CREATE  DATABASE  [IF NOT EXISTS] db_name    
	[create_specification [, create_specification] ...] 

```
例：创建一个名为dream的数据库
![这里写图片描述](http://img.blog.csdn.net/20170417151817824?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYmFpeWVfeGluZw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

（2）CHARACTER SET：指定数据库采用的字符集

```
CHARACTER SET charset_name  
```
例：创建一个名为dream2的数据库，字符集为GBK
![这里写图片描述](http://img.blog.csdn.net/20170417152658101?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYmFpeWVfeGluZw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)




（3）COLLATE：指定数据库字符集的比较方式(校对规则)

```
COLLATE collation_name 
```
例：创建一个使用utf8字符集，并带 校对规则的dream3的数据库
![这里写图片描述](http://img.blog.csdn.net/20170417160025850?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYmFpeWVfeGluZw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)
<br/>

**3.查看数据库**

（1）显示所有数据库：

例：查看所有的数据库
![这里写图片描述](http://img.blog.csdn.net/20170417155139432?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYmFpeWVfeGluZw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

（2）显示数据库创建语句：

例：查看dream3数据库的创建语句
![这里写图片描述](http://img.blog.csdn.net/20170417155338683?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYmFpeWVfeGluZw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)
<br/>
**4.修改数据库**

例：修改数据库dream的字符集为GBK
![这里写图片描述](http://img.blog.csdn.net/20170417155846331?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYmFpeWVfeGluZw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)
<br/>
**5.删除数据库**

例：删除数据库dream3
![这里写图片描述](http://img.blog.csdn.net/20170417160321077?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYmFpeWVfeGluZw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

<br/>
**6.选择数据库**

（1）例：进入到数据库dream2中；
![这里写图片描述](http://img.blog.csdn.net/20170417160508641?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYmFpeWVfeGluZw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

（2）查看当前所选择的数据库；
![这里写图片描述](http://img.blog.csdn.net/20170417160623955?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYmFpeWVfeGluZw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

<br/>
**7.数据类型**

（1）整数类型


|整数类型  |  字节 |范围（有符号）|         用途|
| ------ |:-----:| :-----------:|:----------:|
|TINYINT   |1字节  |    (-128，127)         |   小整数值 |
|SMALLINT  |2字节  |   (-32 768，32 767)    |   大整数值 |
|MEDIUMINT |3字节  | (-8 388 608，8 388 607)  |      大整数值 |
|INT       |4字节  | (-2 147 483 648，2 147 483 647)| 大整数值|
|BIGINT    |8字节  | (-9233372036854775808，9223372036854775807)  |极大整数值 |

<br/>
（2）浮点类型与定点数类型

|整数类型  |  字节 |范围（有符号)| 用途|
| ------ |:-----:| :-----------:|:----------:|
|FLOAT         | 4字节 |  (-3.402 823 466 E+38，1.175 494 351 E-38)，0，(1.175 494 351 E-38，3.402 823 466 351 E+38) | 单精度浮点数值 |
|DOUBLE         |8字节 |(1.797 693 134 862 315 7 E+308，2.225 073 858 507 201 4 E-308)，0，(2.225 073 858 507 201 4 E-308，1.797 693 134 862 315 7 E+308)  |双精度浮点数值 |
|DECIMAL |对DECIMAL(M,D) ，如果M>D，为M+2否则为D+2 依赖于M和D的值 依赖于M和D的值 ||小数值|
<br/>
（3）日期与时间类型

|类型 |    大小(字节)|     范围   |            格式    |      用途 |
|:----:|:-----:|:------:|:-----:|:-----:|
| DATE       |4    |    1000-01-01/9999-12-31 |YYYY-MM-DD  |  日期值 |
| TIME      | 3       | '-838:59:59'/'838:59:59'| HH:MM:SS   | 时间值或持续时间 |
| YEAR    |   1      |   1901/2155              | YYYY     |  年份值 |
| DATETIME  | 8     |  1000-01-01 00:00:00/9999-12-31 23:59:59| YYYY-MM-DD HH:MM:SS |混合日期和时间值 |
| TIMESTAMP  |4  |     1970-01-01 00:00:00/2037 年某时| YYYYMMDD HHMMSS |混合日期和时间值，时间戳|
<br/>
（4）字符串与二进制类型

|字符串类型   |  字节大小 |        描述及存储需求|
|:----:|:-----:|:-----:|
 |   CHAR         |0-255字节  |        定长字符串 |
    | VARCHAR    |   0-255字节      |     变长字符串  |
   |  TINYBLOB    |  0-255字节     |    不超过 255 个字符的二进制字符串  |
   | TINYTEXT   |   0-255字节    |     短文本字符串 | 
    | BLOB      |    0-65535字节     |  二进制形式的长文本数据  |
    | TEXT     |     0-65535字节    |   长文本数据  |
    | MEDIUMBLOB  |  0-16 777 215字节 | 二进制形式的中等长度文本数据  |
     |MEDIUMTEXT  |  0-16 777 215字节  |中等长度文本数据  |
  |   LOGNGBLOB   |  0-4 294 967 295字节  |二进制形式的极大文本数据  |
    | LONGTEXT     | 0-4 294 967 295字节 | 极大文本数据 |
 |    VARBINARY(M)   |   0-M |可变长度的二进制数据|
    | BINARY(M)    | M    |  固定长度的二进制数据   |

<br/>
<br/>
本人才疏学浅，如有错误，请指出 
谢谢！