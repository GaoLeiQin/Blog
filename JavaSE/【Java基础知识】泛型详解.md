**1.泛型的概念：**

泛型是Java SE 1.5的新特性，泛型的本质参数化类型，即所操作的数据类型被指定为一个参数。

**2.使用泛型的优点：**

（1）安全简单，提高代码重用率，在编译的时候检查类型安全，并且所有的强制类型转化都是自动和隐式的；

（2）类型安全，向后兼容，层次清晰，性能较高。

**3.元组类库**（出处：《Thinking Java》）

（1）定义：元组指将一对对象直接打包存储于其中的一个单一对象，这个容器对象允许读取其中元素，但不允许向其中放入新的对象；

（2）元组可以具有任意长度，元组中的对象可以是任意不同类型。

例：

```java
//一个1维元组，它持有1个对象
class OneTuple<A> {
    //code
}

//一个2维元组，它持有两个对象
class TwoTuple<A,B> {
    //code
}
```
**4.定义简单泛型**

（1）泛型类：具有一个或多个类型变量的类

```java
public class Pair<T> {
    private T first;
    private T second;
    
    public Pair(T first, T second) {
        this.first = first;
        this.second = second;
    }
    
    public T getFirst() {
        return this.first;
    }
    
    public void setFirst(T newValue) {
        this.first = newValue;
    }
}
```

（2）泛型方法：


例：

```java
//普通泛型方法
public class ArrayAlg {
    //...表示可变参数
    public <T> T getMiddle(T... a) {
        return a[a.length/2];
    }
}

//集合泛型方法
class ArrayListTest {
    public <T> List<T> list() {
        return new ArrayList<T>();
    }
}
```
（3）泛型接口
例：

```java
public interface Generator<T> {
    T next();
}
```

（4）泛型应用于匿名内部类
例：

```java
public class Customer {
    public  Generator<Customer> clerk() {
        return new Generator<Customer>() {
            @Override
            public Customer next() {
                return new Customer();
            }
        };
    }
}
```

**5.泛型的擦除**

（1）定义;
java字节码中不包含有泛型的类型信息。编译器在编译阶段去掉泛型信息，在运行的时候加上类型信息。

（2）不同类型的擦除

例：

```java
public class ErasedTypeEquairalence{
    public static void main(String[] args) {
        Class c1 = new ArrayList<String>().getClass();
        Class c2 = new ArrayList<Integer>().getClass();
        System.out.println(c1 == c2);
    }
}

//输出结果：  true
```

（3）重载中的类型擦除

例：

```
class HoldItem <T,V>{
    
    public void f(List<T> t) {} // error 编译器报错
    
    public void f(List<V> v) {}
    
    public void f(List<T> t,List<V> v) {}
}
```
**6.泛型的边界**

（1）限定类型：< T extends Comparable & Serializable>

由于泛型有擦除行为，所以只能调用Object的toString( ),equals( ),hashCode( )......方法，但若加上限制，
例：< T extends Dog> 就可以使用Dog类中的方法了。

（2）自限定类型： < T extends Access < T >>

若定义一个类F 
class F extends Access< u > { } 
这个U必须是Access的子类（即：u必须与Access有关）

**7.通配符**

（1）有限制的通配符

List < ? extends Fruit> 子类型通配符
List < ? super class>  超类型通配符

*生产者消费者模型
助记符：PESC (produce - extends   consumer - super)

* 如果想从列表中读取T类型的元素，则需要将这个列表声明成< ? extends T>，之后就不能往该列表中添加任何元素。

* 如果想把T类型的元素加入到列表中，则需要把这个列表声明成< ? super T>，之后就不能从中读取元素。

* 如果一个列表即要生产，又要消费，你不能使用泛型通配符声明列表，比如List < Integer >。

例：

```
	public <T extends Dog> void get1(List<T extends Dog> list){
        list.get(0);
    }
	
	//error
    public <T extends Dog> void set1(List<T extends Dog> list, Dog dog){
        list.add(dog);
    }
	
	//error
    public <T super Cat> void get2(List<T super Cat> list){
        list.get(0);
    }

    public <T super Cat> void set2(List<T super Cat> list, Cat cat){
        list.add(cat);
    }
```

例2：Collections类的copy方法就是典型的PECS模型，下面是copy方法的源码

```
/**
     * Copies all of the elements from one list into another.  After the
     * operation, the index of each copied element in the destination list
     * will be identical to its index in the source list.  The destination
     * list must be at least as long as the source list.  If it is longer, the
     * remaining elements in the destination list are unaffected. <p>
     *
     * This method runs in linear time.
     *
     * @param  <T> the class of the objects in the lists
     * @param  dest The destination list.
     * @param  src The source list.
     * @throws IndexOutOfBoundsException if the destination list is too small
     *         to contain the entire source List.
     * @throws UnsupportedOperationException if the destination list's
     *         list-iterator does not support the <tt>set</tt> operation.
     */
    
    public static <T> void copy(List<? super T> dest, List<? extends T> src) {
        int srcSize = src.size();
        if (srcSize > dest.size())
            throw new IndexOutOfBoundsException("Source does not fit in dest");

        if (srcSize < COPY_THRESHOLD ||
            (src instanceof RandomAccess && dest instanceof RandomAccess)) {
            for (int i=0; i<srcSize; i++)
                dest.set(i, src.get(i));
        } else {
            ListIterator<? super T> di=dest.listIterator();
            ListIterator<? extends T> si=src.listIterator();
            for (int i=0; i<srcSize; i++) {
                di.next();
                di.set(si.next());
            }
        }
    }
```

（2）无界通配符 < ? >

**8.约束与局限性**


（1）不能用基本数据类型实例化类型参数
（2）运行时类型查询只适用于原始类型
（3）不能创建参数化类型的数组

例： 

```java
Pair<String>[] table = new Pair<String> [10];   //error
```
解决方法：
*使用ArrayList< Pair < String > >
*使用工具类 Arrays.asList()

（4）Varargs警告

抑制警告：@SuppressWarnings（“unchecked”） 或 safe Varargs

（5）不能实例化类型变量
（6）泛型类的静态上下文中类型变量无效
（7）不能抛出或捕获泛型类的实例
（8）防范擦除后的冲突
<br/>
<br/>
本人才疏学浅，如有错误，请指出~
谢谢！