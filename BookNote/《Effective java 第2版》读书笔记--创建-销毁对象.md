##第一章 引言

1.本书共78个条目

2.本书大部分内容不是讨论性能的，而是关心如何编写出清晰、正确、可用、健壮、灵活和可维护的代码。

##第二章 创建/销毁对象

<font color="blue" size="4">
**第一条：用静态工厂方法代替构造器**

**1.静态工厂方法比起构造器的优势**

（1）它们有名称：

如果构造器的参数不能正确描述正被返回的对象，具有适当名称的静态方法更容易使用。

当一个类需要多个带有相同签名的构造器时，就可以用静态工厂代替构造器
例：BigInterger.probablePrime()

（2）不必每次调用它们的时候都创建新的对象：

静态工厂方法能为重复的调用返回相同的类，有助于类控制在某个时刻哪些实例应该存在，这种类被称为实例受控的类。

（3）他们可以返回原返回类型的的任何子类型的实例：

API可以返回对象，又不会使对象的类变成共有的，适合基于接口的框架。

被返回的对象由相关的借口精确指定。

共有的静态工厂方法所返回的类不仅可以是非公有的，还可以随着每次调用发生
变化，这取决于工厂方法参数值。

静态工厂方法返回的对象所属的类，在编写该静态方法时可以不必存在（留给开发者实现）

（5）在创建参数化实例的时候，它们使代码更为简单

例：

```
Map< String, List<String>> m = new Map< String, List<String> >();
Map< String, List<String>> m = HashMap.newInstance();   // 减少一次参数
```



**2.静态工厂方法的缺点**

（1）类如果不含公有或者受保护的构造器，就不能被子类化。

（2）它们与其他静态方法没有任何区别。

**3.静态工厂方法的惯用名称**

（1）ValueOf: 类型转换方法

（2）getInstance: 返回唯一的实例

（3）newInstance：保证每个返回的实例都与其他不同

（4）getType/newType：返回工厂对象的类

<font color="blue" size="4">
**第二条：遇到多个构造器参数时要考虑构建器（建造者模式）**


1.如果类的构造器或者静态工厂中具有多个参数, 设计这种类时, 选择Builder模式. 特别是大多数参数都是可选的时候. 

2.与使用传统的重载构造器模式相比, 使用Builder模式既能保证安全性，又能保证可读性。

例：

```
public class NutritionFacts {
    private final int servingSize;
    private final int serving;
    private final int calories;
    private final int fat;
    private final int sodium;
    private final int carbohydrate;
    public static class Builder {
        private int servingSize;
        private int serving;
        private int calories;
        private int fat;
        private int sodium;
        private int carbohydrate;
        public Builder(int servingSize, int serving) {
            this.servingSize = servingSize;
            this.serving = serving;
        }
        public Builder calories(int val) {
            calories = val;
            return this;
        }
        public Builder fat(int val) {
            fat = val;
            return this;
        }
        public Builder sodium(int val) {
            sodium = val;
            return this;
        }
        public Builder carbohydrate(int val) {
            carbohydrate = val;
            return this;
        }
        public NutritionFacts build() {
            return new NutritionFacts(this);
        }
    }
    public NutritionFacts(Builder builder) {
        servingSize = builder.servingSize;
        serving = builder.serving;
        calories = builder.calories;
        fat = builder.fat;
        sodium = builder.sodium;
        carbohydrate = builder.carbohydrate;
    }
    public static void main (String [] args){
        NutritionFacts coca = new Builder(240,8).calories(100).fat(5)
                .sodium(35).carbohydrate(27).build();
    }
}

```


<font color="blue" size="4">
**第三条：用私有构造器或者枚举类型强化Singleton属性**


**1.Singleton指仅仅被实例化一次的类**


例：

```
public class Elvis {
    private static Elvis ourInstance = new Elvis();
    public static Elvis getInstance() {
        return ourInstance;
    }
    private Elvis() {
    }
}
```

**2.实现Singleton的方法**

公有静态域是个final域      

公有的成员是个静态工厂方法

编写一个包含单个元素的枚举类型：更加简洁，无偿提供序列化机制，而且绝对防止多次实例化，是目前最好的方法。

例：

```
public enum A {
	INSTANCE;
    public void invoke(){}
}

```

<font color="blue" size="4">
**第四条：通过私有构造器强化不可被实例化的类的特性**    


1.有时可能需要编写只包含静态方法和静态域的类,  最好给其提供一个私有构造器, 否则编译器会为其生成一个默认无参构造函数. 

例：

```
public class UtilityClass{
    private UtilityClass(){
        throw new AssertionError();
    }
}
```
<font color="blue" size="4">
**第五条：避免创建不必要的对象**


**1.使用静态的初始器（静态块只被执行一次）**

例：

```
public class Person{
    private final Date birthDate;
    
    public Person(Date birthDate){
        this.birthDate = birthDate;
    }
    
    private static final Date BOOM_START;
    private static final Date BOOM_END;
    
    static{
        Calendar gmtCal = Calendar.getInstance(TimeZone,getTimeZone("GMT"));
        gmtCal.set(1946,Calendar.JANUARY,1,0,0,0);
        BOOM_START = gmtCal.getTime();
        gmtCal.set(1965,Calendar.JANUARY,1,0,0,0);
        BOOM_END = gmtCal.getTime();
    }
    
    public boolean isBadyBoomer(){
        return birthDate.compareTo(BOOM_START) >= 0 &&
                birthDate.compareTo(BOOM_END) < 0;
    }
}
```



**2.优先使用基本类型而不是装箱基本类型**

（1）装箱基本类型
例1：
Integer  i = 10;   //装箱过程
自动将基本数据类型转换为包装器类型，自动调用ValueOf(int)方法

例2：
int  n = i ;       //拆箱过程
自动将包装器类型转换为基本数据类型，自动调用intValue方法

<font color="blue" size="4">
**第六条：消除过期对象的引用**

**1.栈内部维护着对过期对象的过期引用，永远也不会被解除。导致内存泄露**

修复方法：清空这些引用（清空引用对象应该是一种例外, 而不是一种规范行为）

例：

```
public Object pop(){
        if(size==0){
	        throw new EemptyStackException();
        }
        
        Object result = elements[--size];
        elements[size] = null;//Eliminate obsolete reference
        return result;
}

```

**2.只要类自己管理内存，就应该警惕内存泄露。**

**3.缓存**

（1）可以考虑使用WeakHashMap来代表缓存

（2）使用后台线程来定期清理

（3）对于更复杂的缓存，使用java.lang.ref

**4.监听器和回调**

（1）只保留回调的弱引用

例：只将他们保存成WeakHashMap中的键。


<font color="blue" size="4">
**第七条：避免使用终结方法finalize()**

**1.终结方法不可预测，很危险**

（1）不能保证会及时被执行，甚至不能保证执行：

（2）不能保证在所有的JVM上都得到同样的体验

（3）可能随意延迟其实例的回收过程

（4）不应该依赖终结方法来更新重要的持久状态，例如分布式系统的永久锁

（5）不要使用System.gc()等函数

**2.如果在终结方法执行时抛出未捕获异常，则此异常会被忽略，终结方法也将终止**

**3.有非常严重的性能损失**

**4.使用显式的终止方法，并要求客户端在每个实例不再有用时调用这个方法。**

例：InputStream的close函数， Timer的cancel函数等，应该在try-catch块的finally部分调用这个显式终止方法

例：

```
		Foo foo = new Foo( );
        try{  } 
        finally{
        foo.terminate();//Explicity termination method
        }

```



**5.终结方法的用途**

（1）在对象的所有者忘记调用显式终止方法时，充当安全网函数。

（2）对象的本地对等体：终结方法释放本地对等体的重要资源

子类覆盖终结方法时，需要显式调用父类的终结方法

终结方法守卫者：匿名内部类，终结它的外围实例。

注：对于每一个带有自定义终结方法的非final公有类，都应该考虑使用这个方法。

<br/>
<br/>
本人才疏学浅，若有错误，请指出
谢谢！