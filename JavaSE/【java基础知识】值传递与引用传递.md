1.值传递：
就是在方法调用的时候，实参将自己的一份拷贝赋给形参，在方法内，对该参数值的修改不影响原来实参。

2.引用传递
在方法调用的时候，实参将自己的地址传递的形参，此时方法内对该参数值的改变，就是对该实参的实际操作。

下面举例说明

例1：基本数据类型

```
   
public class Test1 {   
    public static void main(String[] args) {   
        int  m =  1 ;   
        int  n =  6 ;   
        changeValue(m, n);  
        System.out.println( "m="  + m);   
        System.out.println( "n="  + n);   
    } 
      
    public static void changeValue( int  a,  int  b) {   
        int  temp = a;   
        a = b;   
        b = temp;   
    }   
}   
  
//输出结果：  m=1
//          n=6   
 
```

问题1：为什么m、n的值没有被交换呢？
答：因为参数中传递的是基本类型 m 和 n 的拷贝,在函数中交换的也是拷贝的值 ,而非数据自己的值。 

</br>
例2： 字符串String与字符char

```java
public class Test2 {
	public static void main (String[] args) {
		String str = "I know you were trouble";
		char[] c = {'r','e','d'};
		changeValue(str,c);
		System.out.println(str);
		System.out.println(c);

	}

	public static void changeValue(String str, char[] c) {
		str = "You belong with me";
		c[0] = 'R';
	}
}

//输出结果：  I know you were trouble
//          Red
```

问题2：为什么String值未变而char值改变了？
答：因为String是final的（查看源码得知），所以不可变，而字符数组char[]属于对象类型，即Object的子类，若在方法中修改了它的成员的值，那个修改是生效的，方法调用结束后，它的成员是新的值，但是如果你把它指向一个其它的对象，方法调用结束后，原来对它的引用并没用指向新的对象。 

</br>
例3：引用数据类型

```java
public class Test3 {
    public static void changeValue(int[] arr) {
        arr[0] = 6;
    }

    public static void main(String[] args) {
        int[] data = {1, 2, 3, 4, 5};
        changeValue(data);
        System.out.println(data[0]);
    }
}

//输出结果： 6
```
问题3：为什么数组的值会发生改变？
答：因为在方法中传递引用数据类型 int 数组，实际上传递的是其引用data的拷贝，他们都指向数组对象，在方法中可以改变数组对象的内容。即:对复制的引用所调用的方法更改的是同一个对象。  
</br>
</br>
例4：对象

```
public class Test4 {

    static class Dog {
        int count = 0;
    }

    public static void addValue(Dog dog) {
        dog = new Dog();
        dog.count++;
    }

    public static void main(String args[]) {
        Dog dog = new Dog();
        addValue(dog);
        System.out.println(dog.count);
    }
}

//输出结果:  0  
```
问题4：为什么count的没有改变呢？
答:因为在方法中传递的是一份副本指向了一个新的Dog对象，并对其进行操作，   
而原来的Dog对象并没有发生任何变化。 引用指向的是还是原来的Dog对象。

</br>

3.简单类型变量和引用类型变量的区别：

（1）存储机制：简单类型变量是直接在栈内存中开辟空间存储存储变量值。引用类型变量是由引用空间和存储空间两部分组成，引用空间是在栈内存中，存储空间是在堆内存中，存储空间存储的是变量值，而引用空间存储的是存储空间的首地址。

（2）变量传递：当变量与变量之间进行赋值时，引用类型变量和简单类型变量都属于值传递，但是简单类型变量传递的是内容本身，而引用类型变量传递的是引用的地址。

或许，你还有所疑问，比如 例2中字符数组 和 例3，他们似乎有点像引用传递，可以改变对象中某个属性的值。但是不要被这个假象所蒙蔽，实际上这个传入函数的值是对象引用的拷贝，即传递的是引用的地址值，所以还是按**值传递**。
</br>
</br>
综上所述：<font size="5" color="blue">**java中只存在一种传递方式：值传递**</font>

</br>
</br>

本人才疏学浅，如有错误，请指出~ 
谢谢！