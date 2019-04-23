**1、定义**

Serialization（序列化）为了保存在内存中的各种对象的状态，并且可以把保存的对象状态再读出来，即将对象以一连串的字节描述；反序列化deserialization是一种将这些字节重建成一个对象的过程。



**2、需要实现序列化的几种情况** 

（1）想把内存中的对象保存到一个文件中或者数据库中时候；

（2）想用套接字在网络上传送对象的时候；

（3）想通过RMI传输对象的时候；


**3、实现序列化的方式与反序列化**

（1）实现Serializable接口

即将需要序列化的类实现Serializable接口就可以了，Serializable接口中没有任何方法，可以理解为一个标记，即表明这个类可以序列化。

例：

```
public class SerializableTest {
    public static void main(String[] args) throws Exception {
        File file = new File("G:\\剑指BAT\\作业\\person2.txt");

        ObjectOutputStream out = new ObjectOutputStream(new FileOutputStream(file));
        Person2 person = new Person2("Taylor Swift", 28);
        out.writeObject(person);
        out.flush();
        out.close();
		
		ObjectInputStream in = new ObjectInputStream(new FileInputStream(file));
        Person2 newPerson = (Person2) in.readObject();
        System.out.println(newPerson);
        in.close();
    }
}

class Person2 implements Serializable {

    private String name;
    private int age;


    public Person2(String name, Integer age) {
        this.name = name;
        this.age = age;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public Integer getAge() {
        return age;
    }

    public void setAge(Integer age) {
        this.age = age;
    }
}

```

输出结果：

```
[Taylor Swift, 28]

```

（2）自定义序列化

需要自定义序列化和反序列化的类中需要提供以下方法：

```
private void writeObject(ObjectOutputStream out)
private void readObject(ObjectInputStream in)
private void readObjectNoData()

```

注：ObjectOutputStream先通过反射在要被序列化的对象的类中查找有无自定义的writeObject方法，若有，则会优先调用自定义的writeObject方法。因为查找反射方法时使用的是getPrivateMethod，所以自定以的writeObject方法的作用域要被设置为private。通过自定义writeObject和readObject方法可以完全控制对象的序列化与反序列化。

例：

```
public class SerializableTest {
    public static void main(String[] args) throws Exception {
        File file = new File("G:\\剑指BAT\\作业\\person2.txt");

        ObjectOutputStream out = new ObjectOutputStream(new FileOutputStream(file));
        Person2 person = new Person2("Taylor Swift",28);
        out.writeObject(person);
        out.flush();
        out.close();

        ObjectInputStream in = new ObjectInputStream(new FileInputStream(file));
        Person2 newPerson = (Person2) in.readObject();
        System.out.println(newPerson);
        in.close();
    }
}

class Person2 implements Serializable {

    private String name;
    private int age;


    public Person2(String name, Integer age) {
        this.name = name;
        this.age = age;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public Integer getAge() {
        return age;
    }

    public void setAge(Integer age) {
        this.age = age;
    }

    private void writeObject(ObjectOutputStream out) {
        getAge();
    }
    private void readObject(ObjectInputStream in) {
        setAge(25);
    }

    @Override
    public String toString() {
        return "[" + name + ", " + age +  "]";
    }
}
```

输出结果：

```
[null, 25]
```


（3）实现接口Externalizable

注：Externalizable接口继承自Serializable接口，但他们的序列化机制是完全不同的，使用Serializable的所有方式，在反序列化是不会调用任何的序列化对象构造器，而使用Externalizable是会调用一个无参构造方法的，原因如下：
Externalizable序列化的过程：使用Externalizable序列化时，在进行反序列化的时候，会重新实例化一个对象，然后再将被反序列化的对象的状态全部复制到这个新的实例化对象当中去，这也就是为什么会调用构造方法啦，也因此必须有一个无参构造方法供其调用，并且权限是public。

源码如下：

```
public interface Externalizable extends java.io.Serializable {
    /**
     * The object implements the writeExternal method to save its contents
     * by calling the methods of DataOutput for its primitive values or
     * calling the writeObject method of ObjectOutput for objects, strings,
     * and arrays.
     *
     * @serialData Overriding methods should use this tag to describe
     *             the data layout of this Externalizable object.
     *             List the sequence of element types and, if possible,
     *             relate the element to a public/protected field and/or
     *             method of this Externalizable class.
     *
     * @param out the stream to write the object to
     * @exception IOException Includes any I/O exceptions that may occur
     */
    void writeExternal(ObjectOutput out) throws IOException;

    /**
     * The object implements the readExternal method to restore its
     * contents by calling the methods of DataInput for primitive
     * types and readObject for objects, strings and arrays.  The
     * readExternal method must read the values in the same sequence
     * and with the same types as were written by writeExternal.
     *
     * @param in the stream to read data from in order to restore the object
     * @exception IOException if I/O errors occur
     * @exception ClassNotFoundException If the class for an object being
     *              restored cannot be found.
     */
    void readExternal(ObjectInput in) throws IOException, ClassNotFoundException;
}

```

例：

```
public class SerializableTest {
    public static void main(String[] args) throws Exception {
        File file = new File("G:\\剑指BAT\\作业\\person3.txt");

		//调用无参构造方法
        ObjectOutputStream out = new ObjectOutputStream(new FileOutputStream(file));
        Person3 person = new Person3();
        out.writeObject(person);
        out.flush();
        out.close();

		ObjectInputStream in = new ObjectInputStream(new FileInputStream(file));
        Person3 newPerson = (Person3) in.readObject(); 
        System.out.println(newPerson);
        in.close();
    }
}

class Person3 implements Externalizable{

    private int age;
    private String name;
    private String sex;

    public int getAge() {
        return age;
    }

    public void setAge(int age) {
        this.age = age;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public String getSex() {
        return sex;
    }

    public void setSex(String sex) {
        this.sex = sex;
    }

    public Person3() {
        System.out.println("调用了Person3的构造方法");
    }

    @Override
    public void writeExternal(ObjectOutput out) throws IOException {
        getName();
        getAge();
    }

    @Override
    public void readExternal(ObjectInput in) throws IOException, ClassNotFoundException {
        setName("Taylor");
        setAge(28);
    }

	@Override
    public String toString() {
        return "[" + name + ", " + age +  "]";
    }
}
```

输出结果：

```
调用了Person3的构造方法
调用了Person3的构造方法
[Taylor, 28]
```

**4.序列化ID**

（1）生成策略：当我们一个实体类中没有显示的定义一个名为“serialVersionUID”、类型为long的变量时，Java序列化机制会根据编译时的class自动生成一个serialVersionUID作为序列化版本比较，这种情况下，只有同一次编译生成的class才会生成相同的serialVersionUID。例如，当我们编写一个类时，随着时间的推移，我们因为需求改动，需要在本地类中添加其他的字段，这个时候再反序列化时便会出现serialVersionUID不一致，导致反序列化失败。那么如何解决呢？便是在本地类中添加一个“serialVersionUID”变量，值保持不变，便可以进行序列化和反序列化。

例：

```
private static final long serialVersionUID = 2L;
```

（2）作用：它决定着是否能够成功反序列化，Java的序列化机制是通过在运行时判断类的serialVersionUID来验证版本一致性的。在进行反序列化时，JVM会把传来的字节流中的serialVersionUID与本地实体类中的serialVersionUID进行比较，如果相同则认为是一致的，便可以进行反序列化，否则就会报序列化版本不一致的异常。


**5.transient关键字**

当某个字段被声明为transient后，默认序列化机制就会忽略该字段

例：

```
transient private int age = 28; 
```

**6.静态变量的序列化**

例：

```
class TestSerializable implements Serializable {
    private static final long serialVersionUID = 1L;
    public static int staticVar = 6;

    public static void main(String[] args) throws Exception {
            ObjectOutputStream out = new ObjectOutputStream(
                    new FileOutputStream("serializable.txt"));
            out.writeObject(new TestSerializable());
            out.close();
            //序列化后修改为2
            TestSerializable.staticVar = 2;
            ObjectInputStream oin = new ObjectInputStream(new FileInputStream("serializable.txt"));
            TestSerializable t = (TestSerializable) oin.readObject();
            oin.close();
            System.out.println(t.staticVar);

    }
}
```
输出结果：

```
2
```

原因：因为序列化保存的是对象的状态，静态变量属于类的状态，因此 序列化并不保存静态变量。

**7.小结**

（1）当一个父类实现序列化，子类自动实现序列化，不需要显式实现Serializable接口；

（2）当一个对象的实例变量引用其他对象，序列化该对象时也把引用对象进行序列化；

（3） static,transient后的变量不能被序列化；

<br/>
<br/>
本人才疏学浅，若有错，请指出
谢谢！
