**一、HashSet概述：**

 * HashSet实现Set接口，由哈希表（实际上是一个HashMap实例）支持。它不保证set 的迭代顺序；特别是它不保证该顺序恒久不变。

 * 此类允许使用null元素，是非线程安全的。
 * 由于HashSet是基于HashMap实现的，HashSet底层使用HashMap来保存所有元素，因此HashSet 的实现比较简单，相关HashSet的操作，基本上都是直接调用底层HashMap的相关方法来完成，
 * 如果你已经阅读过HashMap的源码，我相信你读HashSet的源码将非常轻松，如果你没有阅读过HashMap的源码，请不要失望，请浏览我的上一篇博客：[HashMap的实现原理与源码剖析（一）](http://blog.csdn.net/baiye_xing/article/details/71915662)  及 [HashMap源码剖析（二）](http://blog.csdn.net/baiye_xing/article/details/71941366)


**二、HashSet源码剖析（JDK8）**

**1.类的继承关系**

```
public class HashSet<E>
    extends AbstractSet<E>
    implements Set<E>, Cloneable, java.io.Serializable
```

分析：HashSet继承自父类（AbstractSet），实现了Set、Cloneable、java.io.Serializable接口，Serializable接口表示HashSet实现了序列化，即可以将HashSet对象保存至本地，之后可以恢复状态。

类关系图解：
![这里写图片描述](http://img.blog.csdn.net/20170514131838403?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYmFpeWVfeGluZw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

注：绿色虚线表示实现的接口，黄色实线表示继承的类，绿色实线表示接口间的继承。

**2.类的属性**

```
	//序列号
   static final long serialVersionUID = -5024744406713321676L;  
 
   // 底层使用HashMap来保存HashSet中所有元素。  
   private transient HashMap<E,Object> map;  
     
   // 定义一个虚拟的Object对象作为HashMap的value，将此对象定义为static final。  
   private transient HashMap<E,Object> map;
   // Dummy value to associate with an Object in the backing Map
   private static final Object PRESENT = new Object();
```

 **3.类的构造器**

（1）无参构造器

``` 
 public HashSet() {
	//默认的无参构造器，构造一个空的HashSet,实际底层会初始化一个空的HashMap，并使用默认初始容量为16和加载因子0.75。    
   map = new HashMap<E,Object>();  
   }  
``` 
 （2）HashSet(Collection< ? extends E> c)类型构造器 

```
	//构造一个包含指定collection中的元素的新set，实际底层使用默认的加载因子0.75和足以包含指定 collection中所有元素的初始容量来创建一个HashMap。
	public HashSet(Collection<? extends E> c) {  
	   map = new HashMap<E,Object>(Math.max((int) (c.size()/.75f) +     1, 16));  
	   addAll(c);  
    }   
```

注：c 中的元素将存放在此set中的collection。  

（3）HashSet(int float)类型构造器

 

```
   //以指定的initialCapacity和loadFactor构造一个空的HashSet，实际底层以相        应的参数构造一个空的HashMap。 
   public HashSet(int initialCapacity, float loadFactor) {  
	   map = new HashMap<E,Object>(initialCapacity, loadFactor);  
   }  
```

 （4）HashSet（int）类型构造器
   
```
	//以指定的initialCapacity构造一个空的HashSet，实际底层以相应的参数及加载因子 loadFactor为0.75构造一个空的HashMap。  
    public HashSet(int initialCapacity) {  
	   map = new HashMap<E,Object>(initialCapacity);  
    }  
```

 （5）HashSet(int,float,boolean)类型构造器

```
   //以指定的initialCapacity和loadFactor构造一个新的空链接哈希集合（dummy 标记）
   HashSet(int initialCapacity, float loadFactor, boolean dummy) {  
	   map = new LinkedHashMap<E,Object>(initialCapacity, loadFactor);  
   }  
```

注：此构造函数为包访问权限，不对外公开，实际只是是对LinkedHashSet的支持。 实际底层会以指定的参数构造一个空LinkedHashMap实例来实现。 

**4.类的方法**

（1）iterator方法（返回对此set中元素进行迭代的迭代器。返回元素的顺序并不是特定的）
  

```
   //底层实际调用HashMap的keySet来返回所有的key。  
   public Iterator<E> iterator() {  
	   return map.keySet().iterator();  
   }  
```

注：由源码可知HashSet中的元素，只是存放在了底层HashMap的key上， value使用一个static final的Object对象标识。 对此set中元素进行迭代的Iterator。  

（2）size方法（返回此set中的元素的数量）

 

```
 //底层实际调用HashMap的size()方法返回Entry的数量，就得到该Set中元素的个数。   
   public int size() {  
	   return map.size();  
   }  
```

 （3） isEmpty方法（若此set不包含任何元素，则返回true） 
 

```
  //底层实际调用HashMap的isEmpty()判断该HashSet是否为空  
   public boolean isEmpty() {  
	   return map.isEmpty();  
   }  
```

 （4）contains方法（若此set包含指定元素，则返回true）
  
  

```

//底层实际调用HashMap的containsKey判断是否包含指定key（o 是此set中的存在已得到测试的元素） 
   public boolean contains(Object o) {  
	   return map.containsKey(o);  
   }  
```

注：当此set包含一个满足(o==null ? e==null : o.equals(e)) 的e元素时，返回true。


 （5）add方法（若此set中尚未包含指定元素，则添加指定元素）
 
  

```
   //底层实际将将该元素作为key放入HashMap。 
   public boolean add(E e) {  
	   return map.put(e, PRESENT)==null;  
   }  
```

注： 如果此 set 没有包含满足(e==null ? e2==null : e.equals(e2)) 的元素e2，则向此set 添加指定的元素e。  如果此set已包含该元素，则该调用不更改set并返回false。

由于HashMap的put()方法添加key-value对时，当新放入HashMap的Entry中key 
与集合中原有Entry的key相同（hashCode()返回值相等，通过equals比较也返回true）， 新添加的Entry的value会将覆盖原来Entry的value，但key不会有任何改变， 因此如果向HashSet中添加一个已经存在的元素时，新添加的集合元素将不会被放入HashMap中， 原来的元素也不会有任何改变，这也就满足了Set中元素不重复的特性。

 （6）remove方法（若指定元素存在于此set中，则将其移除）
```
   //底层实际调用HashMap的remove方法删除指定Entry。  
   public boolean remove(Object o) {  
	   return map.remove(o)==PRESENT;  
   }  
```

注：如果此set包含一个满足(o==null ? e==null : o.equals(e))的元素e， 则将其移除。如果此set已包含该元素，则返回true ，或是此set因调用而发生更改，则返回true。（一旦调用返回，则此set不再包含该元素）。 

（7）clear方法（从此set中移除所有元素）


```
   //底层实际调用HashMap的clear方法清空Entry中所有元素。  
   public void clear() {  
	   map.clear();  
   }
```

  

注：此调用返回后，该set将为空

（8） clone方法（返回此HashSet实例的浅表副本）

 

```
 //底层实际调用HashMap的clone()方法，获取HashMap的浅表副本，并设置到HashSet中。 
   public Object clone() {  
       try {  
           HashSet<E> newSet = (HashSet<E>) super.clone();  
           newSet.map = (HashMap<E, Object>) map.clone();  
           return newSet;  
       } catch (CloneNotSupportedException e) {  
           throw new InternalError();  
       }  
   }  
}
```

注：clone方法并没有复制这些元素本身。

**5.java API中的方法摘要**

|方法   |返回值|
|:--------:|:-------:|
| boolean add(E e) |如果此 set 中尚未包含指定元素，则添加指定元素。| 
| void clear()  |从此 set 中移除所有元素。 |
| Object clone() |返回此 HashSet 实例的浅表副本：并没有复制这些元素本身。 |
| boolean contains(Object o) | 如果此 set 包含指定元素，则返回 true。| 
| boolean isEmpty() |如果此 set 不包含任何元素，则返回 true。| 
| Iterator<E> iterator() |返回对此 set 中元素进行迭代的迭代器。 |
| boolean remove(Object o) |如果指定元素存在于此 set 中，则将其移除。| 
 |int size() |  返回此 set 中的元素的数量（set 的容量）。 |

<br/>
<br/>
本人才疏学浅，若有错，请指出
谢谢！