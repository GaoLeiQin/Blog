###一、抽象类

1.定义：用abstract关键字修饰，允许包含未实现方法的类被称为抽象类。

2.使用场景：如果拥有一些方法，并且想要其中一部分默认实现！

3.定义格式：

```Java
[<修饰符>] abstract class <类名> {
	[<属性定义>]
	[<构造方法定义>]
	[abstract][<方法声明>]
}
```
注：
（1）抽象类不能实例化，既不能创建对象，只能作为父类用于被继承；
（2）子类继承一个抽象类后，必须要实现父类中所有的抽象方法，否则子类也要被定义为抽象类；
（3）抽象类中可以包含抽象方法，也可以不包含抽象方法；
（4）如果类中包含抽象方法，那么类必须定义成抽象类。

例：

```java
public abstract class Students {
    private String name;
    private int number;

	//抽象方法
    public abstract void getName();

    public void setNumber(int number) {
        this.number = number;
    }
}
```
<br/>

###二、接口

1.定义：是方法声明和常量值定义的集合。

2.使用场景：在有些情况下，如果某个类的所有方法都无法具体实现，此时可以使用接口定义。接口可以理解成一个标准，其他类可以遵守该标准做不同的实现。

<font color="blue">**注：接口中的属性默认是public static final的，而方法默认是public abstract的。**
</font>
3.定义格式

```Java
[<修饰符>] interface <接口名> {
	[<常量声明>]
	[<方法声明>]
}
```
注:  由于接口中的属性属于常量定义，因此在定义属性时必须显示指定初始值，不能使用默认初始化的形式。

4.规则：
（1）接口只包含方法声明和常量定义，即使定义普通属性，该属性在编译后也将变为常量；
（2）当其他类实现该接口时，接口中定义的所有方法都要实现，否则需要定义为抽象类；
（3）一个类可以实现多个接口；
（4）定义接口时可以使用继承，接口之间允许多继承。

注： 接口是方法声明和变量定义的集合，不允许包含变量。接口之间允许多继承，类之间只允许单继承。

5.特点

（1）接口中的成员变量默认都是public static final 类型的，可以省略不写；

（2）接口中的方法默认都是public abstract 类型的，可以省略不写，没有方法体，不能被实例化；

（3）不允许创建接口的实例，但允许定义接口类型的引用变量，该引用变量实现了这个接口的类的实例。

6.implements关键字
（1）定义：让一个类遵循某个接口

例：

```Java
//接口Piano 
interface Piano {
    int MAX = 50;
    void connection();
}

//接口Instrument
interface Instrument {
    int value = 5;
    void play();
}

//类Rain实现了Piano接口
class Rain implements Piano {

    @Override
    public void connection() {
        
    }
}

//类Wind实现了接口Piano和接口Instrument
class Wind implements Piano,Instrument {

    @Override
    public void connection() {

    }

    @Override
    public void play() {

    }
}
```

6.意义：接口体现了程序设计的多态和高内聚低耦合的设计思想。

<br/>
<br/>
本人才疏学浅，如有错误，请指出~
谢谢！