**1.定义**

java.util.concurrent.atomic这个包里面提供了一组原子类。其基本的特性就是在多线程环境下，当有多个线程同时执行这些类的实例包含的方法时，具有排他性，即当某个线程进入方法，执行其中的指令时，不会被其他线程打断，而别的线程就像自旋锁一样，一直等到该方法执行完成，才由JVM从等待队列中选择一个另一个线程进入，这只是一种逻辑上的理解。实际上是借助硬件的相关指令（CAS）来实现的，不会阻塞线程(或者说只是在硬件级别上阻塞了)

**2.分类**

（1）基本类型

类AtomicBoolean、AtomicInteger、AtomicLong 、（AtomicReference ）各自提供对相应类型单个变量的访问和更新。每个类也为该类型提供适当的实用工具方法。

（2）数组类型

类AtomicIntegerArray、AtomicLongArray 和AtomicReferenceArray 进一步扩展了原子操作，对这些类型的数组提供了支持。为其数组元素提供volatile 访问语义，这对于普通数组来说是不受支持的。

（3）对象的属性修改类型:

AtomicIntegerFieldUpdater, AtomicLongFieldUpdater 和 AtomicReferenceFieldUpdater 

**3.核心方法：**

（1）compareAndSet方法

源码如下

```
    public final boolean compareAndSet(int expect, int update) {
        return unsafe.compareAndSwapInt(this, valueOffset, expect, update);
    }
```

（2）compareAndSwapInt方法

```
    public final native boolean compareAndSwapInt(Object var1, long var2, int var4, int var5);
```
（3）compareAndSwapInt的native实现（c代码）

```
        UNSAFE_ENTRY(jboolean, Unsafe_CompareAndSwapInt(JNIEnv *env, jobject unsafe, jobject obj, jlong offset, jint e, jint x))
        UnsafeWrapper("Unsafe_CompareAndSwapInt");
        oop p = JNIHandles::resolve(obj);
        jint* addr = (jint *) index_oop_from_field_offset_long(p, offset);
        return (jint)(Atomic::cmpxchg(x, addr, e)) == e;
```

（4）Atomic的cmpxchg方法

```
inline jint     Atomic::cmpxchg    (jint     exchange_value, volatile jint*     dest, jint     compare_value) {
  // alternative for InterlockedCompareExchange
  int mp = os::is_MP();
  __asm {
    mov edx, dest
    mov ecx, exchange_value
    mov eax, compare_value
    LOCK_IF_MP(mp)
    cmpxchg dword ptr [edx], ecx
  }
}
```

**4.常用类**

1.AtomicInteger

（1）定义：一个提供原子操作的Integer类，通过线程安全的方式操作加减，它提供原子操作来进行Integer的使用，因此十分适合高并发情况下的使用。

（2）方法

|方法|功能|
|:------|:------|
| int addAndGet(int delta) | 以原子方式将给定值与当前值相加|
 |boolean compareAndSet(int expect, int update) |如果当前值 == 预期值，则以原子方式将该值设置为给定的更新值|
| int decrementAndGet() | 以原子方式将当前值减 1|
 |double doubleValue() | 以 double 形式返回指定的数值|
 |float floatValue() | 以 float 形式返回指定的数值|
 |int get() | 获取当前值|
 |int getAndAdd(int delta) | 以原子方式将给定值与当前值相加|
| int getAndDecrement() |以原子方式将当前值减 1|
 |int getAndIncrement() | 以原子方式将当前值加 1|
 |int getAndSet(int newValue) | 以原子方式设置为给定值，并返回旧值| 
| int incrementAndGet() | 以原子方式将当前值加 1| 
| int intValue() | 以 int 形式返回指定的数值| 
| void lazySet(int newValue) |最后设置为给定值|
 |long longValue() | 以 long 形式返回指定的数值|
 |void set(int newValue) |设置为给定值|
 |String toString() | 返回当前值的字符串表示形式|
| boolean weakCompareAndSet(int expect, int update) |如果当前值 == 预期值，则以原子方式将该设置为给定的更新值|


<br>
(3)例：

```
public class AtomicTest {
    public static void main(String[] args) {
        AtomicDemo ad = new AtomicDemo();

        for (int i = 0; i < 5; i++) {
            Thread t = new Thread(ad);
            t.start();
        }

    }
}

class AtomicDemo implements Runnable{

//	private volatile int serialNumber = 0;
    private AtomicInteger serialNumber = new AtomicInteger(0);


    @Override
    public void run() {
        try {
            Thread.sleep(200);
        } catch (InterruptedException e) {
        }
        //结果具有随机性
        System.out.println(Thread.currentThread().getName()+"  "+getSerialNumber());

    }

    public int getSerialNumber(){
	    //每次+1
        serialNumber.getAndIncrement();
        //每次+100
        return serialNumber.addAndGet(100);
    }

}

```

输出结果（不唯一）：

```
Thread-3  202
Thread-4  102
Thread-2  303
Thread-0  505
Thread-1  404
```


注：

* 原子类在具有逻辑性的情况下输出结果也具有随机性

* 因为addAndGet()是原子的，但方法和方法之间的调用不是原子的，所以需要用synchronized

<br>
<br>
本人才疏学浅，若有错误，还请指出 
谢谢！