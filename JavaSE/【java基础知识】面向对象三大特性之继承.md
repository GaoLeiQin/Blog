1.复用类

（1）组合：把一个类当做对象放到另一个类里面去使用
（2）继承：基于已存在的类构造一个新类。
*已存在的类称为父类、基类或超类；
*新类称为子类、派生类、或孩子类。
（3）继承与组合的关系
*继承表现为一般——特殊的关系，子类是一个特殊的父类，是is-a的关系。父类具有所有子类的一般特性。
*组合表现为整体——部分关系，即has-a关系。在组合中，通过将“部分”单独抽取出来，形成自己的类定义，并且在“整体”

2.继承的定义：
实现类重复利用的重要手段，通过继承，子类可以具有父类中定义的方法和属性，子类还可以根据需求添加自己的方法和属性，因此在定义父类和子类类型时，只要通过继承关键字指明两者属于父子关系，那么子类就可以通过继承机制拥有父类的行为和特征。

3.继承定义格式：

```Java
[<修饰符>] class <子类名> extends <父类名> {
	[<属性定义>]
	[<构造方法定义>]
	[<方法声明>]
}

```
注意：定义类只能使用单继承，且子类无法继承父类的构造方法，同时父类中private关键字修饰的属性和方法，子类是无法继承的。如果子类和父类不在同一个包下，default状态的属性和方法也无法继承。

4.几种关键字的介绍

（1）extends关键字：
紧跟在类名后面用于继承父类

（2）super关键字
*用于调用父类中的构造函数；
*调用父类中的方法和属性。

（3）final关键字
*可以在类、成员变量和方法之前进行修饰；
*final修饰类：表示该类不能被继承；
*final修饰成员变量：表示该类是一个常量；
*final修饰方法：表示该方法在类中不能被重写.

（4）instanceof关键字
*一个二元操作符，其作用是判断某个对象是否为某个类或接口类型
*当instanceof左侧操作数为右侧操作数指定的类型或者子类型时返回true，否则返回false.

例：

```Java
public class Person{
    String name;
    int age;

    public Person(String name,int age) {
        this.name = name;
        this.age = age;
    }

    public String getName(){
        return this.name;
    }

    //final方法不能被继承
    public final void setAge(int age) {
        this.age = age;
    }
}

class Student extends Person{
    int number;
    private final int ID_CARD = 612236; //ID_CARD的值不能改变

    public Student(String name, int age, int number){
        super(name,age);  //调用父类的构造函数
        this.number = number;
    }

    public static void main(String[] args){
        Student t = new Student("Lee",21,1001);
		//因为Student是Person的子类，所以返回true
		System.out.println(t instanceof Person);   
        //getName()是从父类继承而来
        System.out.println(t.getName());
    }
}

```
<br/>
<br/>
本人才疏学浅，如有错误，请指出~
谢谢！