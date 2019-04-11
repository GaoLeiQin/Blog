###1.重载

（1）定义：在一个类中具有多个方法名相同而参数列表不同的方法

（2）规则：
*方法名必须相同
*参数列表必须不同
*可以有不同返回值的类型
*可以有不同的访问修饰符；
*可以抛出不同的异常；
*重载方法可以通过this关键字相互调用

例：

```java
public class Animal {
    String name;
    int id;

    public void getName() {
        System.out.println("无参数方法");
    }

    public String getName(String name) {
        return this.name = name;
    }

    int getName(int id) {
        return this.id = id;
    }

    private void getName(String name, int id) {
        System.out.println("有参数方法1");
    }

    private void getName(int id, String name) {
        System.out.println("有参数方法2");
    }

    public static void main(String[] args) {
        Animal animal = new Animal();
        animal.getName();
        System.out.println(animal.getName("goose"));
        System.out.println(animal.getName(6));
        animal.getName("Dog",3);
        animal.getName(5,"Cat");
    }
}

//输出结果：  无参数方法
//          goose
//          6
//          有参数方法1
//          有参数方法2
```
<br/>
###2.重写

（1）定义：当一个子类继承一个父类时，它同是继承了父类的属性和方法。子类可以直接调用父类的属性和方法，如果父类的方法不能满足子类的需求，则可以在子类中对父类的方法进行重写（或覆盖）。重写后，子类对象使用该方法时，会执行子类中重写的方法。

（2）规则：
*重写方法名、参数和返回类型必须与父类方法定义一致；
*重写方法的修饰符不能比父类方法严格；（public>protected>default>private）
*被重写的方法不能为private，否则在其子类中只是新定义了一个方法，并没s有对其进行重写。
*静态方法不能被重写为非静态的方法（会编译出错）
*重写方法如果有throws定义，那么重写方法throws的异常类型可以是父类方法throws的异常类型及其子类类型。

例：

```java
public class Animals {
    String food;

    public void setFood(String food) {
        this.food = food;
    }
}

class Dog extends Animal {
    @Override
    public void setFood(String food) {
        System.out.println("重写此方法");
        this.food = "meat";
    }

    public static void main(String[] args) {
        Dog dog = new Dog();
        dog.setFood("grass");
        System.out.println("食物： "+dog.food);
    }
}

//输出结果： 重写此方法
//         食物： meat
```
<br/>
###3.重载与重写的区别

（1）重载是在同一个类中，方法名相同，但参数个数、参数类型、返回值类型可能不同，而重写是子类重写父类的方法，方法名、参数个数、参数类型、返回值类型均必须相同。
（2）重写多态性起作用，对调用被重载过的方法可以大大减少代码的输入量
<br/>
<br/>
本人才疏学浅，如有错误，请指出~ 
谢谢