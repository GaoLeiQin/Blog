1.定义：
将一个类的定义放在另一个类的定义内部，与之对应包含内部类的类被称为外部类

2.内部类的作用：

（1）内部类提供了更好的封装，可以把内部类隐藏在外部类之内，不允许同一个包中的其他类访问该类；
（2）内部类的方法可以直接访问外部类的所有数据，包括私有的数据；
（3）内部类所实现的功能使用外部类同样可以实现，只是有时使用内部类更方便；
（4）每个内部类都能独立的继承一个（接口）实现，无论外部类是否已经继承了个（接口）实现，对于内部类没有影响；
（5）内部类不能被覆盖，但可通过继承实现。


**3.普通内部类（成员内部类）**

（1）使用方法：
* 内部类定义在外部类的内部，相当于外部类的一个成员变量的位置，内部类可以使用任意访问控制符，如 public 、 protected 、 private 等；

*内部类中定义的方法可以直接访问外部类中的数据，而不受访问控制符的影响

*定义了成员内部类后，必须使用外部类对象来创建内部类对象，而不能直接去 new 一个内部类对象，

即：内部类 对象名 = 外部类对象.new 内部类( );

例：

```java
Outer o = new Outer();           //创建外部类对象
Inner io = o.new Inner();        //创建内部类对象
```

*编译上面的程序后，会发现产生了两个 .class ⽂文件；外部类对应的class文件是：外部类名 .class ，内部类对应的class文件是：外部类名$内部类名.class

（2）注意：

*外部类是不能直接使用内部类的成员和方法，可先创建内部类的对象，然后通过内部类的对象来访问其成员变量和方法。

*如果外部类和内部类具有相同的成员变量或方法，内部类默认访问自己的成员变量量或方法，如果要访问外部类的成员变量量，可以使用 this 关键字。

例：

```Java
public class Employee {
    private String name = "Redis";
    
    class Mannger {
        private String name = "Netty";

        public void getName(){
            String name = "Lee";
            System.out.println("局部变量：" + name);
            System.out.println("内部类变量：" + this.name);
            System.out.println("外部类变量：" + Employee1.this.name);
        }
    }

    public static void main(String[] args) {
        Employee e = new Employee();
        Mannger m = e.new Mannger();
        m.getName();
    }
}

```
运行结果：

```Java
局部变量：Lee
内部类变量：Netty
外部类变量：Redis
```
<br/>

**4.静态内部类**

（1）定义：由static关键字修饰的内部类。

（2）特点：

*静态内部类不能直接访问外部类的非静态成员，但可以通过new 外部类().成员 的方式访问

*如果外部类的静态成员与内部类的成员名称相同，可通过“类名.静态成员”访问外部类的静态成员；若不同，则可通过“成员名”直接调.用外部类的静态成员

*创建静态内部类的对象时，不需要外部类的对象，可以直接创建  
即：内部类 对象名= new 内部类();

例：

```Java
public class Employee2 {
    private static String name = "Redis";
    static class Mannger {
        private String name = "Netty";

        public String getName(){
            return this.name;
        }
    }

    public static void main(String[] args) {
        Mannger m = new Mannger();
        System.out.println(m.getName());
    }
}
```
运行结果：

```
Netty

```
<br/>
**5.方法内部类**

(1)定义：
内部类定义在外部类的方法中，方法内部类只在该方法的内部可见，即只在该方法内可以使用。

（2）局限：
由于方法内部类不能在外部类的方法以外的地方使用，因此方法内部类不能使用访问控制符和 static 修饰符。

例：

```java
public class Parcel {
    public String destination() {
        String label = "top";
        class Destination {
        
            public void readLabel() {
                System.out.println("方法内部类！");
            }
        }
        new Destination().readLabel();
        return label;
    }

    public static void main(String[] args) {
        Parcel p = new Parcel();
        System.out.println(p.destination());
    }
}
```
运行结果:

```java
方法内部类！
top
```
<br/>
**6.匿名内部类**

（1）定义：
只创建内部类的一个对象，没有名称。

（2） 注意：

*使用匿名内部类时，我们必须是继承一个类或者实现一个接口，但是两者不可兼得，同时也只能继承一个类或者实现一个接口。

*匿名内部类中是不能定义构造函数的。

*匿名内部类中不能存在任何的静态成员变量和静态方法。

*匿名内部类为局部内部类，所以局部内部类的所有限制同样对匿名内部类生效。

*匿名内部类不能是抽象的，它必须要实现继承的类或者实现的接口的所有抽象方法。

例：

```java
public class Bird {
    public Goose getName(int n){
        return new Goose (){
            int number = 6 + n;
            public int getNumber(){
                return number;
            }
        };
    }

    public static void main(String[] args) {
        Bird bird = new Bird();
        Goose goose = bird.getName(3);
        System.out.println(goose.getNumber());
    }
}

interface Goose {
    int getNumber();
}

//输出结果：   9
```

<br/>
本人才疏学浅，若有错误，请指出~
谢谢！