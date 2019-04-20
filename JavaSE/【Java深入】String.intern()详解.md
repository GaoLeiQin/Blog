####**1.引入**

* String不属于8种基本数据类型，String是一个对象。

* 因为对象的默认值是null，所以String的默认值也是null；但它又是一种特殊的对象，有其它对象没有的一些特性。

* String类型的常量池方法有两种：
 * 直接使用双引号声明出来的String对象会直接存储在常量池中。
 * 如果不是用双引号声明的String对象，可以使用String提供的intern方法。intern 方法会从字符串常量池中查询当前字符串是否存在，若不存在就会将当前字符串放入常量池中

####**2.JDK6与JDK7版本的intern区别**

（1）例：

```
public static void main(String[] args) {
    String s = new String("1");
    s.intern();
    String s2 = "1";
    System.out.println(s == s2);

    String s3 = new String("1") + new String("1");
    s3.intern();
    String s4 = "11";
    System.out.println(s3 == s4);
}
```

输出结果：
jdk6 ：false false
jdk7 ：false true

（2）原因

*  jdk6中的常量池是放在 Perm 区中的，Perm 区和正常的 JAVA Heap 区域是完全分开的。 JAVA Heap 区域的对象地址和字符串常量池的对象地址进行比较肯定是不相同的，即使调用String.intern方法也是没有任何关系的。

*   jdk7 的版本中，字符串常量池已经从 Perm 区移到正常的 Java Heap 区域了。s3引用对象内容是"11"，但此时常量池中是没有 “11”对象的。s3.intern()就将 s3中的“11”字符串放入 String 常量池中，因为此时常量池中不存在“11”字符串，在常量池中生成一个 "11" 的对象，可以直接存储堆中的引用。该引用指向 s3 引用的对象。 也就是说引用地址是相同的。
**图解**
![这里写图片描述](http://img.blog.csdn.net/20170714163822419?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYmFpeWVfeGluZw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)


####**3.JDK8的实例**

```
public class InternString {
    public static void main(String[] args) {
        //s1常量池中
        String s1 = "ab123" ;
        //s2在堆中
        String s2 = new String( "ab123" ) ;
        System.out.println( s1 == s2 );
        //s3到常量池中取s1的引用
        String s3 = s2.intern() ;
        System.out.println( s1 == s3 ) ;
        //而s2在堆中，并没有在常量池中
        System.out.println(s2 == s3);

        //new会创建两个对象，两个1在堆上，而另两个1在常量池中，
        // intern指向常量池,寻找11并没有，就在常量池中新建11，s4就指向s6，而s6==s5
        String s5 = new String("1") + new String("1");
        s5.intern();
        String s4 = "11";
        System.out.println(s5 == s4);

        //new会创建两个对象，一个9在堆上，而另一个在常量池中，
        // intern指向常量池，并没有在常量池中新建9,ss指向常量池,
        String s = new String("9");
        s.intern();
        String ss = "9";
        //s在堆上，而ss在常量池中
        System.out.println(s == ss);
    }
}

```

输出结果

```
false
true
false
true
false
```

####**4.String 的几个面试题**

（1）如何比较两个字符串？使用“==”还是equals()方法？

答：“==”测试的是两个对象的引用是否相同，而equals()比较的是两个字符串的值是否相等。除非你想检查的是两个字符串是否是同一个对象，否则你应该使用equals()来比较字符串。

（2）为什么针对安全保密高的信息，char[]比String更好?

答：因为String是不可变的，就是说它一旦创建，就不能更改了，直到垃圾收集器将它回收走。而字符数组中的元素是可以更改的（译者注：这就意味着你就可以在使用完之后将其更改，而不会保留原始的数据）。所以使用字符数组的话，安全保密性高的信息(如密码之类的)将不会存在于系统中被他人看到。

（3）可以对字符串使用switch条件语句吗？

答：从JDK 7开始可以，但在JDK 6或者之前的版本，不能使用switch条件语句。

（4）常用方法

```
//将字符串转化成int
Integer.parseInt("10")
//将字符串用空白字符分割开
String[] str = String.split("\\s+");
//重复一个字符串
String repeated = StringUtils.repeat(str,3);
//将字符串转换成时间
Date date = new SimpleDateFormat("MMMM d, yy", Locale.ENGLISH).parse(str);
//计算一个字符串某个字符的出现次数
int n = StringUtils.countMatches("11112222", "1");
```
（5）substring()方法到底做了什么？

答：JDK 6中, 该方法用一个字符数组来表示现存的字符串，然后给这个字符数组提供一个“窗口”，但实际并没有创建一个新的字符数组。但在JDK 7中，substring()会创建新的字符数组，而不是使用现存的字符数组。这样更快，因为垃圾收集器会收集不用的长字符串，而仅保存要使用的子字符串。

（6）String vs StringBuilder vs StringBuffer

答：String是不可变的，而StringBuilder和StringBuffer是可变的，但StringBuffer是synchronized的,它是线程安全的的，而StringBuilder是非线程安全的，但是StringBuffer比StringBuilder慢。



<br>
<br>
本人才疏学浅，若有错，请指出
谢谢！

参考资料：[深入解析String#intern](https://tech.meituan.com/in_depth_understanding_string_intern.html)