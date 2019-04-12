1.字符串的定义  
 
 字符串是一种使用频率最高的数据类型，为了加强程序的运行速度，java设计了两种不同的方法来生成字符串对象，一种是调用String类的构造函数，一种是使用双引号。
 
 例:
 

```java
String str1 = "Dream it possible";
String str2 = new String ("we don't talk anymore");
```

注：Java为String类型提供了缓冲池机制，当使用双引号定义对象时，Java环境首先去字符串缓冲池寻找内容相同的字符串，如果存在就拿出来使用，否则就创建一个新的字符串放在缓冲池中。

</br>
2.字符串的操作
（1）用+号连接两个字符串（效率低）
注： 由于不能修改Java字符串中的字符，所以在Java文档中将String类对象称为不可变字符串

</br>
3.检测字符串是否相等

（1）equals方法实现了等价关系：自反性，对称性，传递性，一致性。
（2）对于任何非null的引用值x， x.equals(null)必须返回false。
（3）“==”与“equals（）”区别

**“==”检测内存地址是否相等**
**“equals（）”比较的是内容是否相等**

例：

```Java
String a = "dream";
String b = "dream";
String c = "d" + "ream";
String d = new String ("dream");
String e = "d" + new String ("ream");

System.out.println(a == b);       // true
System.out.println(a == d);       // false
System.out.println(a.equals(d));  // true
System.out.println(c == e);       // false
System.out.println(c.equals(e));  // true


```


4.空串与null

（1）定义：
空串： “ ”  是长度为0的字符串
null  ：  表示目前没有任何对象与该变量关联
（2）避免NPE

```java
//正例：
if (str != null && str.length() != 0)

//反例（若str为空，会抛出NPE）
if (str.length() != 0 && str != null)  

```

</br>
5. String 、StringBuffer与StringBuilder的区别

（1）String的长度是不可变的
（2）StringBuffer的长度是可变的，如果对字符串中的内容经常进行操作，特别是要修改内容时，就可以使用StringBuffer，且它是线程安全的。它的底层是一个char数组，可以自动扩容，默认初始化容量是16，最好在创建StringBuffer之前预测存储字符数量，就可以采用指定初始化容量的方式创建，这样减少了底层的数组拷贝，提高了效率。
（3）StringBuilder的长度是可变的，且是单线程的非线程安全（效率高，常用），默认初始化容量是16。

</br>
本人才疏学浅，如有错误，请指出~ 
谢谢！