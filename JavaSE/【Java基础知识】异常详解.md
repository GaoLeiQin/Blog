1.概念：
当出现程序无法控制的外部环境问题，Java就会用异常对象来描述。

2.异常的分类

![Java异常类层次结构图：](http://img.blog.csdn.net/20170411211557266?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYmFpeWVfeGluZw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

（1）未受检异常：派生于Error类或RuntimeException类的所有异常，

*Error异常
一般很少见，也很难通过程序解决，他可能源于程序的bug，但更可能源于环境问题，
例：内存耗尽

*RuntimeException（运行时异常）
指程序存在bug
例：数组越界、0被除、错误的类型转换。访问空指针。
*几种常见的运行异常
IllegalArgumentException
IllegalStateException
NullPointerException
ArrayIndexOutOfBoundsException
ClassCastException


（2）已受检异常

*定义：程序正确，但因为外在的环境条件不满足引发。
例： IO错误
Java编译器强制要求处理这类异常，如果不捕获这类异常，程序将不能编译。

3.处理异常的方法

（1）在发生异常的地方直接处理（推荐）
例：try { } catch() { } 语句块

（2）将异常抛给调用者，让调用者处理
例：

```
public class ThrowException {
    int a ;
    //1.在方法名后抛出异常
    public void f() throws Exception{
        if (a == -1) {
	        //2.在语句中抛出异常
            throw new NullPointerException("空指针异常！");
        }
    }
}
```
<br/>
4.捕获异常

（1）使用try/catch语句块

*若在try语句块中的任何代码抛出一个在catch子句中说明的异常类，那程序将跳过try语句块的其余代码，再执行catch子句中的处理器代码。

*若在try语句块中的代码没有抛出任何异常，那么程序将跳过catch子句。

（2）捕获多个异常
利用 try { } catch ( ) { }  catch ( ) { } ...

（3）再次抛出异常与异常链
*异常链指在捕获一个异常后抛出另一个异常，并希望把原始异常的信息保存下来。

（4）finally子句

*无论异常是否被抛出，finally子句总能被执行；
*当要把除内存之外的资源恢复到他们的初始状态时，用finally子句；
*finally中的return会覆盖try和catch中的return值。

例：

```java
public class ReturnException {

    public static int number(int a) {
        try {
            if (a < 0) {
                throw new Exception();
            }
            System.out.println("try语句块！");
            return 1;
        }catch(Exception e) {
            System.out.println("catch语句块！异常");
            return -1;
        }finally {
            System.out.println("finally语句块！");
            return 5;
        }
    }
    
    public static void main(String[] args) {
        System.out.println("当 a = 3 时：");
        System.out.println(number(3));
        System.out.println("----------");
        System.out.println("当 a = -2 时：");
        System.out.println(number(-2));
    }
}
```
运行结果：

```java
当 a = 3 时：
try语句块！
finally语句块！
5
----------
当 a = -2 时：
catch语句块！异常
finally语句块！
5
```
结果分析：finally语句总会被执行，若finally语句中有return子句，则会覆盖try语句和catch语句中的return值。
<br/>
5.堆栈跟踪
*是一个方法调用过程的列表，它包含了程序执行过程中方法调用的特定位置。

例：

```java
try {
     //code   
    }catch(Exception e) {
        e.printStackTrace();
    }
```

6.创建自定义异常

（1）背景：Java提供的异常体系不可能遇见所有期望加以报告的错误，这就需要自定义异常。

（2）要求：必须从已有的异常继承。


7.使用异常机制的技巧

（1）异常处理不能代替简单的测试；
（2）不要过分的细化异常；
（3）利用异常层次结构；
（4）不要压制异常；
（5）检查错误时，“苛刻”要比放任好；
（6）不要羞于传递异常（早抛出，晚捕获）

<br/>
<br/>
本人才疏学浅，若有错误，请指出~
谢谢！