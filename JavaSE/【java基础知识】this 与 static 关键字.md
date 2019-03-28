##一、this关键字

<br/>
1.定义：java中为了解决变量的命名冲突和不确定性问题，引入了this关键字,this代表当前类的一个实例，它经常出现在方法和构造器方法中。

2.使用场景：
（1）返回调用当前方法的引用；
（2）在构造器中调用当前类中的其他构造方法；
（3）当方法参数和成员变量名相同时，用于区分参数名和成员变量名。

3.使用规则：
（1）this可以调用一个构造器，但不能调用两个；
（2）必须置于最起始处；
（3）除构造器外，编译器禁止在其他任何方法中调用构造方法。

4.分析
从内存分析上来说，this可以看作是heap里面的一个对象内存里面的一个成员变量内存，这个变量的内容指向这个对象自己。

例：


```java
public class Cat {
    private String name ;
    private int age ;

    public Cat(){
        System.out.println("无参构造器") ;
    }

    public Cat (String name) {
        this();   //调用类的无参构造方法,必须在第一行
        System.out.println("参数为name的构造器");
    }

    public Cat(String name, int age) {
        this(name);         //调用类的有参(name)构造方法,必须在第一行
        this.name = name;   //this指属性中的name
        this.age = age;     //this指属性中的age
        System.out.println("参数为name，age的构造器");
    }

    public String getCat(){
        return this.name + " " + this.age;
        //返回调用当前方法的对象的引用，this指Cat
    }

    public static void main(String args[]) {
        Cat cat = new Cat("Tom",6);
        System.out.println(cat.getCat());
    }

}


//输出结果：  无参构造器
//          参数为name的构造器
//          参数为name，age的构造器
//          Tom 6
```
<br/>
<br/>
##二、static关键字

<br/>

1.定义：在Java中，static关键字可以修饰方法、属性、自由块、和内部类，使用static修饰这些成员时，可以理解这些成员与类相关，通过“类名.成员”的形式调用，没有static修饰可以理解成这些成员与对象有关，需要通过“对象名.成员”的形式调用。

2.&nbsp;static修饰方法：
static关键字不能修饰构造方法，在static修饰的方法中不能调用没有static修饰的方法和属性，**在静态方法中不能使用this和super关键字**

思考题：为什么将main方法定义为static的呢？
答：main方法是Java应用程序的主入口方法，Java解析器在运行时会寻找该方法，但是在解析器解析该方法的时候还没有来得及创建当前类对象，所以讲main方法定义为static。

3.&nbsp;static修饰属性：
当static修饰属性时，当前这个属性属于类不属于对象（被当前类对象共享），一个对象修改静态属性值后，会影响其他对象。

注：静态成员变量占用的内存是在一个叫做data segment的地方，一旦要new一个对象的时候，除了在堆内存里面分配一个对象所占的内存之外，还要在数据区（data segment）分配一份内存，存储静态成员变量。之后无论new多少次，还会在这个堆内存继续为对象分配内存。但是不会再在数据区分配内存了。为什么会是这样呢?因为程序初始化的过程就是先初始化静态代码，分配好内存了就不再轻易改动了。“注意：字符串常量也是在“数据区”分配内存空间”，然后还是像指针一样被指向。其实即使没有new对象，在数据区也有为其分配内存，只是如果你不new对象的话，那么要调用它们必须使用类名。

4.&nbsp;static修饰自由块：
自由块是Java类中使用{}括起来的代码块，自由块中的代码在构造方法之前执行，因此可以将一部分初始化代码放在自由块中。
静态自由块：是由static修饰的自由块，通常用于初始化静态常量，**只被执行一次**

5.非static与static之间的关系
（1）static方法不能调用非static方法和属性；
（2）非static能调用static方法和属性。

例1：

```Java
class Employee {
    private static int nextId = 1;
    private String name;

    public Employee (String name) {
        this.name = name;
        System.out.println("构造器方法");
    }
    
    //匿名代码块（先于构造器执行，但在静态代码块之后执行）
    {
        System.out.println("匿名代码块！");
    }

    //静态代码块（最先执行）
    static{
        System.out.println("静态代码块！");
    }

    public String getName() {
        return name;
    }

    public static int getNextId() {
        return nextId;   //直接调用静态成员变量，不能调用非静态成员变量，例：name
    }

    public static void main(String[] args) {
        Employee e = new Employee("Harry");
        System.out.println(e.getName());  //通过创建对象引用非静态方法
        System.out.println(getNextId());  //静态方法直接调用
    }
}


//输出结果：  静态代码块！
//          匿名代码块！
//          构造器方法
//          Harry
//          1

```
例2：

```java
class Number {

    int a = 20;

    {
        a = 30;
    }
    
    public void print() {
        System.out.println("a = "+a);
    }

    public static void main(String[] args) {
        new Number().print();
    }
}

//输出结果：  a = 30 
```

例3：

```
class Number1 {
    {
        b = 3;
    }
    
    int b = 6;
    
    public void print() {
        System.out.println("b = "+b);
    }

    public static void main(String[] args) {
        new Number1().print();
    }
}

//输出结果： b = 6
```
例4：

```java
class Number2 {
    {
        c = 7;
    }
    
    static int c = 6;

    static{
        c = 5;
    }
    
    public void print() {
        System.out.println("c = "+c);
    }

    public static void main(String[] args) {
        new Number2().print();
    }
}

//输出结果：  c = 7
```
例5：

```
class Number3 {
    {
        d = 11;
    }
    
    static int d = 12;

    static{
        d = 13;
    }

    public Number3() {
        d = 14;
    }

    public void print() {
        System.out.println("d = " + d);
    }

    public static void main(String[] args) {
        new Number3().print();
    }
}

//输出结果：  d = 14
```
结论：元素初始化时，若在构造器内初始化，则会覆盖静态代码块初始化，匿名代码块初始化，属性初始化。
<br/>
<br/>
本人才疏学浅，如有错误，请指出
谢谢！