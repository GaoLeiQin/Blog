**1.定义**

（1）指程序可以访问，检测和修改它本身状态或行为的一种能力，并能根据自身行为的状态和结果，调整或修改应用所描述行为的状态和相关的语义，

（2）反射使代码能够得到装载到JVM中的类的内部信息，允许执行程序时才得到需要类的内部信息，而不是在编写代码的时候就必须要知道所需类的内部信息，这使反射成为构建灵活的应用的主要工具。

**2..功能**

（1）运行时判断任意一个对象所属的类。
（2）运行时构造任意一个类的对象。
（3）运行时判断任意一个类所具有的成员变量和方法。
（4）运行时调用任意一个对象的方法
（5）生成动态代理

**3.常用类**

（1）Class ： 类对象（核心类，实现反射的基础）
（2）Constructor ： 类的构造器对象
（3）Field ： 类的属性对象
（4）Method ： 类的方法对象，

**4.常用方法**


（1）获取构造器的方法

|方法名|作用|
|:-----:|:----:|
|Constructor getConstructor(Class[] params) | 返回一个 Constructor 对象，它反映此 Class 对象所表示的类的指定公共构造方法。|
|Constructor[] getConstructors()| 获   返回一个包含某些 Constructor 对象的数组，获取类的所有公共构造方法。|
|Constructor getDeclaredConstructor(Class[] params) | 返回一个 Constructor 对象，该对象反映此 Class 对象所表示的类或接口的指定构造方法。)|
|Constructor[] getDeclaredConstructors() | 返回 Constructor 对象的一个数组，获取类声明的所有构造方法。|

<br/>
（2）获得字段的方法

|方法名|作用|
|:-----:|:----:|
|Field getField(String name) |获得命名的公共字段|
|Field[] getFields() |获得类的所有公共字段|
|Field getDeclaredField(String name) |获得类声明的命名的字段|
|Field[] getDeclaredFields()| 获得类声明的所有字段 |

<br/>
（3）获得方法信息的方法

|方法名|作用|
|:-----:|:----:|
|Method getMethod(String name, Class[] params) |使用特定的参数类型，获得命名的公共方法|
|Method[] getMethods()| 获得类的所有公共方法|
|Method getDeclaredMethod(String name, Class[] params) |使用特写的参数类型，获得类声明的命名的方法|
|Method[] getDeclaredMethods() | 获得类声明的所有方法|
<br/>


**5.应用反射机制的实例**

例1：Customer类（以下例子需要用到的类）

```
class Customer {
    private int id;
    private String name;
    private int age;

    public Customer() {
    }

    public Customer(String name, int age) {
        this.name = name;
        this.age = age;
    }

    public Customer(int id) {
        this.id = id;
    }

    public int getId() {
        return id;
    }

    public void setId(int id) {
        this.id = id;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public int getAge() {
        return age;
    }

    public void setAge(int age) {
        this.age = age;
    }

    @Override
    public String toString() {
        return  "name= " + this.name+  ",age= " + this.age+ ", id= "+this.id;
    }
}
```


（1）实例化对象

例2：

```
public void acquireConstructor() throws Exception {
        //获取类
        Class clazz = Class.forName("blog.Customer");
        System.out.println("获取类名称： "+clazz.getName());

        //1.通过无参构造实例化对象
        System.out.println("无参构造器实例化对象");
        Customer customer = (Customer)clazz.newInstance();
        customer.setAge(22);
        customer.setName("高拉特");
        customer.setId(10);
        System.out.println(customer);

        //2.通过有参构造器实例化对象
        System.out.println("有参构造器实例化对象");
        Constructor<?>[] cons = clazz.getConstructors();
        Customer c1 = (Customer)cons[2].newInstance();
        Customer c2 = (Customer)cons[1].newInstance("武磊",27);
        Customer c3 = (Customer)cons[0].newInstance(7);

        System.out.println("顾客1 "+c1);
        System.out.println("顾客2 "+c2);
        System.out.println("顾客3 "+c3);
    }
```
输出结果

```
获取类名称： blog.Customer
无参构造器实例化对象
name= 高拉特,age= 22, id= 10
有参构造器实例化对象
顾客1 name= null,age= 0, id= 0
顾客2 name= 武磊,age= 27, id= 0
顾客3 name= null,age= 0, id= 7
```

（2）获取全部属性

例3：

```
public void acquireAllProperty() throws ClassNotFoundException {
        Class<?> clazz = Class.forName("blog.Customer");
        // 获取全部属性
        Field[] field = clazz.getDeclaredFields();
        for (int i = 0; i < field.length; i++) {
            // 属性的修饰符
            int modifiers = field[i].getModifiers();
            String priv = Modifier.toString(modifiers);
            // 属性的类型
            Class<?> type = field[i].getType();
            System.out.println(priv + " " + type.getName() + " " + field[i].getName());
        }

        System.out.println("实现的接口或者父类的属性");
        Class<?> c = Class.forName("java.lang.String");
        // 取得实现的接口或者父类的属性
        Field[] filed1 = c.getFields();
        for (int j = 0; j < filed1.length; j++) {
            // 权限修饰符
            int mo = filed1[j].getModifiers();
            String priv = Modifier.toString(mo);
            // 属性类型
            Class<?> type = filed1[j].getType();
            System.out.println(priv + " " + type.getName() + " " + filed1[j].getName() + ";");
        }
    }
```
输出结果：

```
private int id
private java.lang.String name
private int age
实现的接口或者父类的属性
public static final java.util.Comparator CASE_INSENSITIVE_ORDER;

```

（3）获取单个属性

例4：

```
public void acquireSingleProperty() throws Exception {
        Class<?> clazz = Class.forName("blog.Customer");
        Customer customer = (Customer)clazz.newInstance();
        // 可以直接对 private 的属性赋值
        Field field = clazz.getDeclaredField("id");
        field.setAccessible(true);
        field.set(customer, 10001);
        System.out.println(field.get(customer));
    }
```
输出结果;

```
10001
```
（4）获取全部方法

例5：

```
public void acquireAllMethod() throws Exception {
        Class<?> clazz = Class.forName("blog.Customer");
        //获取全部方法
        Method method[] = clazz.getMethods();
        for (int i = 0; i < method.length; ++i) {
            Class<?> returnType = method[i].getReturnType();
            Class<?> para[] = method[i].getParameterTypes();
            int temp = method[i].getModifiers();
            //获取修饰符
            System.out.print(Modifier.toString(temp) + " ");
            //获取返回值类型
            System.out.print(returnType.getName() + "  ");
            //获取方法名
            System.out.print(method[i].getName() + " ");
            System.out.println();
        }
    }
```

输出结果：

```
public java.lang.String  toString 
public java.lang.String  getName 
public int  getId 
public void  setName 
public void  setId 
public void  setAge 
public int  getAge 
public final void  wait 
public final void  wait 
public final native void  wait 
public boolean  equals 
public native int  hashCode 
public final native java.lang.Class  getClass 
public final native void  notify 
public final native void  notifyAll 
```

（5）获取单个方法

例6：

```
class test {
    public void acquireSingleMethod() throws Exception {
        Class<?> clazz = Class.forName("blog.Customer");
        // 调用getName方法
        Method method = clazz.getMethod("getName");
        method.invoke(clazz.newInstance());

        // 调用setName方法
        method = clazz.getMethod("setName",String.class);
        method.invoke(clazz.newInstance(), "诶尔克森");

    }
}

class Customer {
    private int id;
    private String name;
    private int age;

    public Customer() {
    }

    public Customer(String name, int age) {
        this.name = name;
        this.age = age;
    }

    public Customer(int id) {
        this.id = id;
    }

    public int getId() {
        return id;
    }

    public void setId(int id) {
        this.id = id;
    }

    public String getName() {
        System.out.println("调用了getName（）方法");
        return name;
    }

    public void setName(String name) {
        System.out.println("设置的名字为： "+name);
        this.name = name;
    }

    public int getAge() {
        return age;
    }

    public void setAge(int age) {
        this.age = age;
    }

    @Override
    public String toString() {
        return  "name= " + this.name+  ",age= " + this.age+ ", id= "+this.id;
    }
}
```

输出结果：

```
调用了getName（）方法
设置的名字为： 诶尔克森
```

<br/> 
**6.动态创建代理类**

（1）代理模式的作用：为其他对象提供一种代理以控制对这个对象的访问。

（2）动态代理类： java.lang.reflect.Proxy	

* Proxy 提供用于创建动态代理类和实例的静态方法，它还是由这些方法创建的所有动态代理类的超类

* InvocationHandler	是代理实例的调用处理程序 实现的接口，每个代理实例都具有一个关联的调用处理程序。对代理实例调用方法时，将对方法调用进行编码并将其指派到它的调用处理程序的 invoke 方法。
 
注：动态Proxy是这样的一种类:它是在运行生成的类，在生成时你必须提供一组Interface给它，然后该class就宣称它实现了这些interface。你可以把该class的实例当作这些interface中的任何一个来用。当然，这个Dynamic Proxy其实就是一个Proxy，它不会替你作实质性的工作，在生成它的实例时你必须提供一个handler，由它接管实际的工作。

例：

```
//若想要完成动态代理，需要定义一个InvocationHandler接口的子类，已完成代理的具体操作。

class MyInvocationHandler implements InvocationHandler {
    private Object obj = null;
    public Object bind(Object obj) {
        this.obj = obj;
        return Proxy.newProxyInstance(obj.getClass().getClassLoader(), obj.getClass().getInterfaces(), this);
    }
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        Object temp = method.invoke(this.obj, args);
        return temp;
    }
}
```

<br/> 
<br/> 
本人才疏学浅，若有错误，请指出
谢谢！