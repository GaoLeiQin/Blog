先来介绍一下Thread类

###**Thread**

**1.常用API**
|方法|描述|
|:-----|:-------|
|int activeCount() |返回当前线程的线程组中活动线程的数目|
|Thread currentThread() | 返回对当前正在执行的线程对象的引用|
|int enumerate(Thread[] tarray) | 将当前线程的线程组及其子组中的每一个活动线程复制到指定的数组中|
|ThreadGroup getThreadGroup()  | 返回该线程所属的线程组|
|boolean interrupted()   |测试当前线程是否已经中断。清除中断状态|
|boolean isInterrupted() | 测试线程是否已经中断。中断状态不受影响|
|void join()   |等待该线程终止（该线程优先运行，终止后其他线程再运行）|
|void setPriority(int newPriority)  |  更改线程的优先级,优先级：1‐10，最低1，最高10，默认5|
|void yield() | 暂停当前正在执行的线程对象，并执行其他拥有相同优先级线程，即让当前线程主动让出时间片|
|void setDaemon(boolean on)| 将该线程标记为守护线程或用户线程，等待其他所有线程执行完，守护线程才会执行|

**2.区别**


（1）interrupted()和isInterrupted()的区别

* interrupted()   测试当前线程是否已经中断。清除中断状态，

* isInterrupted()  测试线程是否已经中断。中断状态不受影响

例1：

```
Thread.currentThread().isInterrupted();
System.out.println("isInterrupted的中断状态1 ： "+Thread.currentThread().isInterrupted());
System.out.println("isInterrupted的中断状态2 ： "+Thread.currentThread().isInterrupted());
```

例1输出结果：

```
isInterrupted的中断状态1 ：true
isInterrupted的中断状态2 ：true
```

例2：

```
Thread.currentThread().interrupt();
System.out.println("interrupted的中断状态1 ： "+Thread.interrupted());
System.out.println("interrupted的中断状态2 ： "+Thread.interrupted());
```

例2输出结果:

```
interrupted的中断状态1 ： true
interrupted的中断状态2 ： false
```

注：

* interrupt不能打断一个正在运行的线程，它只能在运行的线程里面判断中断标记位，然后让线程尽快执行完，。Interrupt会使wait和sleep的线程抛出InterruptedException异常；如果再sleep状态下停止某一线程，会进入catch语句，并且清除停止状态值，使之变成false；


* 将方法interrupt()和return结合使用也能实现停止线程的效果，如果用return，会可以结束当前方法，但是如果当时获取了锁，也会使锁被释放，所以建议抛异常的方法来实现线程的停止，因为再catch语句块中还可以将异常向上抛，实线程停止的事件得以传播，执行return会释放锁，则有可能使一些请理性的工作得不到完成，另一个原因是对锁定的对象进行了解锁，导致数据得不到同步处理，出现了数据不一致问题。

（2）currentThread()方法返回代码正在被哪个线程调用。而this返回的是当前对象。

* run方法中，currentThread()和this指同一对象

* 其他方法中，currentThread()指主线程，而this指Thread-0 等

（3）wait与sleep方法的区别：

wait会释放锁，而sleep不会释放锁

（4）守护线程：JVM中垃圾回收线程就是一个守护线程。

###**ThreadLocal**

**1.定义**

该类提供了线程局部 (thread-local) 变量。这些变量不同于它们的普通对应物，因为访问某个变量（通过其 get 或 set 方法）的每个线程都有自己的局部变量，它独立于变量的初始化副本。也就是ThreadLocalVariable（线程本地变量）

**2.特性**

（1）每个线程都有自己的局部变量

 每个线程都有一个独立于其他线程的上下文来保存这个变量，一个线程的本地变量对其他线程是不可见的

（2）独立于变量的初始化副本

ThreadLocal可以给一个初始值，而每个线程都会获得这个初始化值的一个副本，这样才能保证不同的线程都有一份拷贝。

（3）状态与某一个线程相关联

ThreadLocal 不是用于解决共享变量的问题的，不是为了协调线程同步而存在，而是为了方便每个线程处理自己的状态而引入的一个机制，

**3.常用方法**

（1）get方法

源码如下

```
public T get() {
        //获取当前执行线程
        Thread t = Thread.currentThread();
        //取得当前线程的ThreadLocalMap实例
        ThreadLocalMap map = getMap(t);
        //如果map不为空，说明该线程已经有了一个ThreadLocalMap实例
        if (map != null) {
            //map中保存线程的所有的线程本地变量，我们要去查找当前线程本地变量
            ThreadLocalMap.Entry e = map.getEntry(this);
            //如果当前线程本地变量存在这个map中，则返回其对应的值
            if (e != null)
                return (T)e.value;
        }
        //如果map不存在或者map中不存在当前线程本地变量，返回初始值
        return setInitialValue();
    }
```
getMap方法源码如下

```
//直接返回线程对象的threadLocals属性
ThreadLocalMap getMap(Thread t) {
        return t.threadLocals;
    }
```

（2）setInitialValue方法

源码如下：

```
private T setInitialValue() {
        //获取初始化值，initialValue 就是我们之前覆盖的方法
        T value = initialValue();
        Thread t = Thread.currentThread();
        ThreadLocalMap map = getMap(t);
        //如果map不为空，将初始化值放入到当前线程的ThreadLocalMap对象中
        if (map != null)
            map.set(this, value);
        else
            //当前线程第一次使用本地线程变量，需要对map进行初始化工作
            createMap(t, value);
        //返回初始化值
        return value;
    }
```
（3）set方法

源码如下：

```
public void set(T value) {
        Thread t = Thread.currentThread();
        ThreadLocalMap map = getMap(t);

        if (map != null)
            map.set(this, value);
        //说明线程第一次使用线程本地变量(注意这里的第一次含义)
        else
            createMap(t, value);
    }
```

（4）remove方法

```
public void remove() {
         //获取当前线程的ThreadLocalMap对象
         ThreadLocalMap m = getMap(Thread.currentThread());
         //如果map不为空，则删除该本地变量的值
         if (m != null)
             m.remove(this);
     }
```
（5）ThreadLocal源码中有一个非常重要的内部类ThreadLocalMap，它占了ThreadLocal源码的多一半（437/723）,我在这里就不介绍了，如果有兴趣，可以研究一下。

**4.适用场景**

* 解决多线程中数据数据因并发产生不一致问题。

* ThreadLocal为每个线程的中并发访问的数据提供一个副本，通过访问副本来运行业务，这样的结果是耗费了内存，单大大减少了线程同步所带来性能消耗，也减少了线程并发控制的复杂度。

**5.小结**

* ThreadLocal与线程同步无关，并不是为了解决多线程共享变量问题！ 

* 内存泄露：只要该线程对象被gc回收，就不会出现内存泄露，但在ThreadLocal设为null和线程结束这段时间不会被回收的，就发生了我们认为的内存泄露。

* 其实在ThreadLocal类中有一个静态内部类ThreadLocalMap(其类似于Map)，用键值对的形式存储每一个线程的变量副本，ThreadLocalMap中元素的key为当前ThreadLocal对象，而value对应线程的变量副本，每个线程可能存在多个ThreadLocal。

* 每个线程内部都会持有一个对ThreadLocalMap实例的引用，ThreadLocalMap实例相当于线程的局部变量空间，存储着线程各自的数据，ThreadLocalMap内部使用table数组存储Entry，默认大小INITIAL_CAPACITY(16)

<br>
<br>
本人才疏学浅，若有错误，还请指出 
谢谢！