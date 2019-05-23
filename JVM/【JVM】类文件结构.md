###**Class文件的定义**

* 一组以8字节为基础单位的二进制流，

* 各个数据项目严格按照顺序紧凑排列在class文件中，

* 中间没有任何分隔符，这使得class文件中存储的内容几乎是全部程序运行的程序。
 
注：Java虚拟机规范规定，Class文件格式采用类似C语言结构体的伪结构来存储数据，这种结构只有两种数据类型：无符号数和表。

###**无符号数**

属于基本数据类型，主要可以用来描述数字、索引符号、数量值或者按照UTF-8编码构成的字符串值，大小使用u1、u2、u4、u8分别表示1字节、2字节、4字节和8字节。

###**表**
 
**1.定义**

* 由多个无符号数或者其他表作为数据项构成的复合数据类型，所有的表都习惯以“_info”结尾。

* 表主要用于描述有层次关系的复合结构的数据，比如方法、字段。需要注意的是class文件是没有分隔符的，所以每个的二进制数据类型都是严格定义的。

整个class文件本质上就是一张表，具体内容如下图所示

![这里写图片描述](http://img.blog.csdn.net/20170626122127980?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYmFpeWVfeGluZw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

由上表可以看出：class文件主要由魔数、Class文件的版本号、常量池、访问标志、类索引（还包括父类索引和接口索引集合）、字段表集合、方法表集合、属性表集合组成。

**2.实例**

```
public class TestClass {
        private int m;
                                                                                                                                                    
        public int inc(){
                return m+1;
        }
}
```

TestClass.class的二进制结构如下图所示

![这里写图片描述](http://img.blog.csdn.net/20170626122900194?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYmFpeWVfeGluZw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

根据这张图，我接下来进行详细分析

**3.魔数** 

![这里写图片描述](http://img.blog.csdn.net/20170626123555354?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYmFpeWVfeGluZw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

* 每个Class文件的头4个字节称为魔数（Magic Number）

* 唯一作用是用于确定这个文件是否为一个能被虚拟机接受的Class文件。

* Class文件魔数的值为0xCAFEBABE。如果一个文件不是以0xCAFEBABE开头，那它就肯定不是Java class文件

注： 很多文件存储标准中都使用魔数来进行身份识别，譬如图片格式，如gif或jpeg等在文件头中都存有魔数

**4.Class文件的版本号**

![这里写图片描述](http://img.blog.csdn.net/20170626123636784?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYmFpeWVfeGluZw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

（1）紧接着魔数的4个字节是Class文件版本号，而版本号又分为：

* 次版本号(minor_version): 前2字节用于表示次版本号

* 主版本号(major_version): 后2字节用于表示主版本号。

（2）Java的版本号是从45开始的。如果Class文件的版本号超过虚拟机版本，将被拒绝执行。

|十六进制版本号|  十进制版本号| 编译器版本|
|:-------|:------|
|  0X0034|50|JDK1.8  |    
|  0X0033|50|JDK1.7   |   
|  0X0032|50|JDK1.6    |  
|  0X0031|49|JDK1.5　　|
|  0X0030|48|JDK1.4　　|
|  0X002F|47|JDK1.3　　|
|0X002E|46|JDK1.2 |

<br>
**5.常量池**

![这里写图片描述](http://img.blog.csdn.net/20170626124420182?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYmFpeWVfeGluZw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

（1）紧接着魔数与版本号之后的是常量池入口，即class文件的资源从库，它的特点如下：

* Class文件结构中与其它项目关联最多的数据类型

* 占用Class文件空间最大的数据项目之一

* 在文件中第一个出现的表类型数据项目。

（2）  常量池之中主要存放两大类常量：

* 字面量: 就是常量，如文本字符串、被声明为final的常量值等

* 符号引用: ，包括类和接口的权限定名、字段的名称和描述符、方法的名称和描述符  

* 直接引用可以是直接指向引用目标的指针、相对偏移量或者是一个能够间接定位到目标的句柄。直接引用是和虚拟机的内存布局有关的，同一个符号引用在不同的虚拟机上翻译的直接引用一般是不同的。如果有了直接引用，那么引用的目标必定是存在内存中的。

（3）常量池又分为两种

* constant_pool_count：占2字节，本例为0x0016，转化为十进制为22，即说明常量池中有22个常量（只有常量池的计数是从1开始的，其它集合类型均从0开始），索引值为1~22。第0项常量具有特殊意义，如果某些指向常量池索引值的数据在特定情况下需要表达“不引用任何一个常量池项目”的含义，这种情况可以将索引值置为0来表示

*  constant_pool：表类型数据集合，即常量池中每一项常量都是一个表，共有14种(JDK1.7前只有11种)结构各不相同的表结构数据。这14种表都有一个共同的特点，即均由一个u1类型的标志位开始，可以通过这个标志位来判断这个常量属于哪种常量类型，常量类型及其数据结构

通过javap -verbose TestClass 即可得到所有常量池中的常量，如图（部分截图）

![这里写图片描述](http://img.blog.csdn.net/20170626132129076?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYmFpeWVfeGluZw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

**6.访问标志**
![这里写图片描述](http://img.blog.csdn.net/20170626132435508?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYmFpeWVfeGluZw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

（1） 常量池之后的数据结构是访问标志(access_flags),用于识别一些类或接口层次的访问信息，主要包括：

* 这个Class是类还是接口

* 是否定义public

* 是否定义abstract类型

* 如果是类的话是否被声明为final等

具体的标志位见下图

![这里写图片描述](http://img.blog.csdn.net/20170626132711344?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYmFpeWVfeGluZw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

**7.类索引、父类索引和接口索引集合**

![这里写图片描述](http://img.blog.csdn.net/20170626132810229?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYmFpeWVfeGluZw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

（1）这三项数据主要用于确定这个类的继承关系，类索引(this_class)和父类索引(super_class)都是一个u2类型的数据，而接口索引(interface)集合是一组u2类型的数据。

（2）索引的含义

* 类索引(this_class)，用于确定这个类的全限定名，占2字节

* 父类索引(super_class)，用于确定这个类父类的全限定名（Java语言不允许多重继承，故父类索引只有一个。除了java.lang.Object类之外所有类都有父类，故除了java.lang.Object类之外，所有类该字段值都不为0），占2字节

* 接口索引计数器(interfaces_count)，占2字节。如果该类没有实现任何接口，则该计数器值为0，并且后面的接口的索引集合将不占用任何字节，
 
* 接口索引集合(interfaces)，一组u2类型数据的集合。用来描述这个类实现了哪些接口，这些被实现的接口将按implements语句（如果该类本身为接口，则为extends语句）后的接口顺序从左至右排列在接口的索引集合中

**8.字段表集合**

![这里写图片描述](http://img.blog.csdn.net/20170626133041860?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYmFpeWVfeGluZw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

（1）fields_count：字段表计数器，即字段表集合中的字段表数据个数，占2字节。本测试类其值为0x0001，即只有一个字段表数据，也就是测试类中只包含一个变量（不算方法内部变量）

（2）fields：字段表集合，一组字段表类型数据的集合。字段表用于描述接口或类中声明的变量，包括类级别（static）和实例级别变量，不包括在方法内部声明的变量

**9.方法表集合**

（1） methods_count：方法表计数器，即方法表集合中的方法表数据个数。占2字节，其值为0x0002，即测试类中有2个方法

（2）methods：方法表集合，一组方法表类型数据的集合。方法表结构和字段表结构一样：

**10.属性表集合**

![这里写图片描述](http://img.blog.csdn.net/20170626133449591?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYmFpeWVfeGluZw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

分析：

* 起始2位为0x0001，说明有一个类属性。
 
* 接下来2位为属性的名称，0x0010，指向常量池中第16个常量：SourceFile。

* 接下来4位为0x00000002，说明属性体长度为2字节。

* 最后2个字节为0x0011，指向常量池中第27个常量：TestClass.java，即这个Class文件的源码文件名为TestClass.java


<br>
<br>
本人才疏学浅，若有错，请指出 
谢谢！