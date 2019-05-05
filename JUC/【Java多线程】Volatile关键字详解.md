###**预备知识**

**1..内存模型**

通过缓存一致性协议来保证一致性。最出名的就是Intel 的MESI协议，MESI协议保证了每个缓存中使用的共享变量的副本是一致的。它核心的思想是：当CPU写数据时，如果发现操作的变量是共享变量，即在其他CPU中也存在该变量的副本，会发出信号通知其他CPU将该变量的缓存行置为无效状态，因此当其他CPU需要读取这个变量时，发现自己缓存中缓存该变量的缓存行是无效的，那么它就会从内存重新读取。

图解：
![这里写图片描述](http://img.blog.csdn.net/20170619160123383?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYmFpeWVfeGluZw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

**2.原子性、可见性与有序性**

* 原子性：在Java中，对基本数据类型的变量的读取和赋值操作是原子性操作，即这些操作是不可被中断的，要么执行，要么不执行。

* 可见性：当多个线程访问同一个变量时，一个线程修改了这个变量的值，其他线程能够立即看得到修改的值

* 有序性：在Java内存模型中，允许编译器和处理器对指令进行重排序，但是重排序过程不会影响到单线程程序的执行，却会影响到多线程并发执行的正确性。

**3.happens-before原则（先行发生原则）**

* 程序顺序规则：一个线程中的每个操作，先行发生于该线程中的任意后序操作。

* 监视器锁规则：对一个锁的解锁，先行发生于随后对这个锁的加锁

* volatile变量规则：对一个volatile域的写，先行发生于任意后续对这个volatile域的读。

* 传递性：如果A先行发生于B，而B又先行发生于C，那么A先行发生于C

* start（）规则：Thread对象的start()方法先行发生于此线程的每个一个动作

* join（）规则：如果线程A执行操作ThreadB.join()方法并成功返回，那么线程B中的任意操作先行发生于线程A从ThreadB.join（）操作成功返回


###**引入**

例1：没有Volatile修饰

```
public class VolatileTest {
    public static void main(String[] args) {
        ThreadDemo td = new ThreadDemo();
        new Thread(td).start();

        while(true){
            if(td.isFlag()){
                System.out.println("已跳出循环！");
                break;
            }
        }
    }
}

class ThreadDemo implements Runnable {

    private boolean flag = false;

    @Override
    public void run() {

        try {
            Thread.sleep(200);
        } catch (InterruptedException e) {
        }

        flag = true;

        System.out.println("flag=" + isFlag());

    }

    public boolean isFlag() {
        return flag;
    }

}
```

输出结果（程序一直在运行）：flag=true

分析：你会产生疑问，flag=true啊，为什么没有进入到if 方法里面呢！别急，我来小小的改造一下，你看看结果！

例2：加入Volatile关键字修饰

```
public class VolatileTest {
    public static void main(String[] args) {
        ThreadDemo td = new ThreadDemo();
        new Thread(td).start();

        while(true){
            if(td.isFlag()){
                System.out.println("已跳出循环！");
                break;
            }
        }
    }
}

class ThreadDemo implements Runnable {

    //用volatile关键字修饰
    private volatile boolean flag = false;

    @Override
    public void run() {

        try {
            Thread.sleep(200);
        } catch (InterruptedException e) {
        }

        flag = true;

        System.out.println("flag=" + isFlag());

    }

    public boolean isFlag() {
        return flag;
    }

}
```

运行结果（程序已结束）：

```
flag=true
已跳出循环！
```

分析：你会发现结果正常了，仅仅是因为加了volatile关键字修饰吗，对，就是这样，我接下来就讲讲volatile关键字。

###**解析**

**1.特点**

（1）禁止指令重排序，但不具备“互斥性”；

（2）当多个线程进行操作共享数据时，可以保证内存中的数据可见，但不能保证变量的“原子性”。

**2.指令重排序的问题**

（1）实例化一个对象其实可以分为三个步骤：

* 分配内存空间
* 初始化对象
* 将内存空间的地址赋值给对应的引用

但是由于操作系统可以对指令进行重排序，所以上面的过程也可能会变成如下过程：

* 分配内存空间。
* 将内存空间的地址赋值给对应的引用。
* 初始化对象

如果是这个流程，多线程环境下就可能将一个未初始化的对象引用暴露出来，从而导致不可预料的结果。因此，为了防止这个过程的重排序，我们需要将变量设置为volatile类型的变量。


**3..内存可见性问题**

（1）Lock前缀指令会引起处理器缓存写到内存，线程的本地内存失效，别的线程只能从主存中读取数据。而本地内存的值会立马刷新到主存中去。

（2）一个处理器的缓存回写到内存会导致其他处理器的缓存无效。

**4.synchronized和volatile比较**

（1）关键字volatile是线程同步的轻量级实现，所以volatile性能比synchronized要好并且volatile只用来修饰变量，而synchronized可以修饰方法，以及代码块

（2）多线程访问volatile不会发生阻塞，而synchronized会出现阻塞

（3）volatile可以保证数据的可见性但不能保证原子性，synchronized可以保证原子性，也可以保证可见性。

**5.注意**

（1） 使用volatile强制的从公共内存中读取变量的值，

（2）volatile的非原子性，在i++或i=i+1;会出现不安全

（3）在JSR-133之前的旧内存模型中，一个64位long/double型变量的读写操作可以被拆分为两个32位的读写操作执行，但从JSR-133（JDK5）开始，只允许将一个64位long/double型变量的写操作可以被拆分为两个32位的写操作执行，任意读操作必须具有原子性（即必须要在单个读事务中执行）

<br>
<br>
本人才疏学浅，若有错误，还请指出 
谢谢！
