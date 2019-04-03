1.定义：

一个引用类型在多种情况下的多种状态
即：对象的多种形态（继承是多态实现的基础）

2.引用多态

（1）父类的引用可以指向本类的对象
（2）父类的引用可以指向子类的对象，但不能调用子类独有的成员。

3.方法多态

（1） 创建本类对象时，调用的方法为本类的方法；
（2）创建子类对象时，调用的方法为子类重写的方法或者继承的方法。

4.备注

（1）对象的初始化就是对象的引用指向本类或者指向子类。如果指向本类，调用对象的引用的方法时，就调用的是父类的方法，如果指向子类，就是调用子类的方法。
（2）父类的引用可以指向子类，但子类的引用不不能指向父类。

5.方法调用的绑定

（1）定义：将一个方法调用同一个方法主体关联起来。

（2）前期绑定：在程序执行前进行绑定。
例如： static方法  、final方法（private方法属于final方法）

（3）后期绑定（动态绑定）：在运行时根据对象的类型进行绑定。
例如： 除static方法和final方法之外的其他方法。

6.缺陷：

（1）覆盖私有方法：只有非private才能被覆盖；
（2）域和静态方法：
*当子类对象转型为父类引用时，任何域访问操作都将由编译器解析，因此不是多态；
*若某个方法是静态的，它的行为就不具有多态性；

7.构造器与多态

（1）关系：
构造器并不具有多态性，他们实际上是static方法，只不过该static声明是隐式的。
（2）构造器的调用顺序
*调用基类构造器，不断反复的递归，直到最低层的导出类；
*按声明顺序调用成员的初始化方法；
*调用导出类的构造器主体。

8.继承与清理

（1）若遇到清理问题，必须在导出类中覆盖清理方法。
（2）当覆盖被继承类的清理方法时，务必牢记调用基类版本清理方法，否则基类的清理动作不会发生。

9.构造器内部的多态方法的行为

（1）初始化的实际过程
*在其他任何事物发生之前，将分配给对象的存储空间初始化成二进制的0
*调用基类构造器，需在调用子类构造器之前调用；
*按照声明的顺序调用成员的初始化方法
*调用导出类的构造主体。

（2）用尽可能的方法使对象进入正常状态；如果可以避免调用其他方法。

10.多态存在的三个必要条件
（1）要有继承；
（2）要有重写；
（3）父类引用指向子类对象（向上转型）。

11.多态的优点：

（1）可替换性（substitutability）。多态对已存在代码具有可替换性。
（2）可扩充性（extensibility）。多态对代码具有可扩充性。增加新的子类不影响已存在类的多态性、继承性，以及其他特性的运行和操作。实际上新加子类更容易获得多态功能。
（3）接口性（interface-ability）。多态是超类通过方法签名，向子类提供了一个共同接口，由子类来完善或者覆盖它而实现的子类，
（4）灵活性（flexibility）。它在应用中体现了灵活多样的操作，提高了使用效率。
（5）简化性（simplicity）。多态简化对应用软件的代码编写和修改过程，尤其在处理大量对象的运算和操作时，这个特点尤为突出和重要。

例：

```Java
class Taylor {
    public void life() {
        System.out.println("Good Life");
    }

    public static void food() {
        System.out.println("父类-鱼");
    }
    public void song1(){
        System.out.println("Taylor -- I known you were trouble");
        song2();
    }

    public void song2(){
        System.out.println("Taylor -- You belong with me");
    }
}

class Swift extends Taylor{
    //动态绑定，调用此方法
    public void life() {
        System.out.println("Bad Life");
    }

    //静态方法没有运行时多态（动态绑定），不能引用此方法。
    public static void food() {
        System.out.println("子类-虫");
    }
    //(重载)向上转型后，父类不能引用该方法!
    public void song1(String song){
        System.out.println("Swift -- wildest dreams");
    }

    //(重写)父类引用调用song2时，调用该方法
    @Override
    public void song2() {
        System.out.println("Swift -- You belong with me");
    }

    public static void main(String[] args) {
        Taylor taylor = new Swift();
        taylor.life();
        taylor.food();
        taylor.song1();
    }
}

```

运行结果：

```
Bad Life
父类-鱼
Taylor -- I known you were trouble
Swift -- You belong with me
```
<br/>
<br/>

12.多态的提高：

例：

```Java
class A {
    public String show(D obj) {
        return ("A and D");
    }

    public String show(A obj) {
        return ("A and A");
    }

}

class B extends A{
    public String show(B obj){
        return ("B and B");
    }

    public String show(A obj){
        return ("B and A");
    }
}
class C extends B{

}

class D extends B{

}

class Test {
    public static void main(String[] args) {
        A a1 = new A();
        A a2 = new B();
        B b = new B();
        C c = new C();
        D d = new D();

        System.out.println("1--" + a1.show(b));
        System.out.println("2--" + a1.show(c));
        System.out.println("3--" + a1.show(d));
        System.out.println("4--" + a2.show(b));
        System.out.println("5--" + a2.show(c));
        System.out.println("6--" + a2.show(d));
        System.out.println("7--" + b.show(b));
        System.out.println("8--" + b.show(c));
        System.out.println("9--" + b.show(d));
    }
}
```
运行结果：

```
1--A and A
2--A and A
3--A and D
4--B and A
5--B and A
6--A and D
7--B and B
8--B and B
9--A and D
```

思考这个例子的结果，详解请参考链接： [深入理解java多态性](http://blog.csdn.net/thinkGhoster/article/details/2307001)

<br/>
<br/>
本人才疏学浅，如有错误，请指出~
谢谢！