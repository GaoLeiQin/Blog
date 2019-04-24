**1.引入**

（1）在JDK1.5之前。Java主要靠synchronized这个关键字保证同步，已解决多线程下的线程不安全问题，但是这会导致锁的发生，会引发一些个性能问题。

（2）锁主要存在一下问题

* 在多线程竞争下，加锁、释放锁会导致比较多的上下文切换和调度延时，引起性能问题。

* 一个线程持有锁会导致其它所有需要此锁的线程挂起。

* 如果一个优先级高的线程等待一个优先级低的线程释放锁会导致优先级倒置，引起性能风险。

这时候你可能想到了volatile，但是volatile不能保证原子性，因此同步还是需要用到锁。

（3）独占锁是一种悲观锁，synchronized就是一种独占锁，会导致其它所有需要锁的线程挂起，等待持有锁的线程释放锁。而另一个更加有效的锁就是乐观锁。所谓乐观锁就是，每次不加锁而是假设没有冲突而去完成某项操作，如果因为冲突失败就重试，直到成功为止。乐观锁用到的机制就是CAS，Compare and Swap。

**2.定义**

* CAS (Compare-And-Swap) 是一种硬件对并发的支持，针对多处理器操作而设计的处理器中的一种特殊指令，用于管理对共享数据的并发访问。

* CAS，即比较并交换，用于在硬件层面上提供原子性操作。在 Intel 处理器中，比较并交换通过指令cmpxchg实现。比较是否和给定的数值一致，如果一致则修改，不一致则不修改。

**3.特点**

（1）CAS 是一种无锁的非阻塞算法的实现，一个线程的失败或者挂起不影响其他线程的失败或者挂起的算法。乐观锁用到的主要机制是CAS。

（2）CAS包含三个操作数：

需要读写的内存值V、进行比较的预估值A、拟写入的更新值B、

注：当且仅当V 的值等于A 时，CAS 通过原子方式用新值B 来更新V 的值，否则不会执行任何操作


**4.CAS的缺点**

（1）ABA问题

* CAS操作容易导致ABA问题,也就是在做a++之间，a可能被多个线程修改过了，只不过回到了最初的值，这时CAS会认为a的值没有变。解决ABA问题的方法有很多，可以增加一个修改计数，只有修改计数不变的且a值不变的情况下才做a++，也可以引入版本号，当版本号相同时才做a++操作等；

* JDK5引入类AtomicStampedReference来解决ABA问题。这个类的compareAndSet方法作用是首先检查当前引用是否等于预期引用，并且当前标志是否等于预期标志，如果全部相等，则以原子方式将该引用和该标志的值设置为给定的更新值。

（2）循环时间长开销大。

自旋CAS如果长时间不成功，会给CPU带来非常大的执行开销。如果JVM能支持处理器提供的pause指令那么效率会有一定的提升，pause指令有两个作用，第一它可以延迟流水线执行指令（de-pipeline）,使CPU不会消耗过多的执行资源，延迟的时间取决于具体实现的版本，在一些处理器上延迟时间是零。第二它可以避免在退出循环的时候因内存顺序冲突（memory order violation）而引起CPU流水线被清空（CPU pipeline flush），从而提高CPU的执行效率。

（3）会增加程序测试的复杂度，稍不注意就会出现问题。 

（4）只能保证一个共享变量的原子操作。

当对一个共享变量执行操作时，我们可以使用循环CAS的方式来保证原子操作，但是对多个共享变量操作时，循环CAS就无法保证操作的原子性，这个时候就可以用锁，或者有一个取巧的办法，就是把多个共享变量合并成一个共享变量来操作。比如有两个共享变量i＝2,j=a，合并一下ij=2a，然后用CAS来操作ij。从Java1.5开始JDK提供了AtomicReference类来保证引用对象之间的原子性，你可以把多个变量放在一个对象里来进行CAS操作。


**5.应用**

concurrent包中的基础类都是使用这种模式来实现的，而concurrent包中的高层类又是依赖于这些基础类来实现的。

图解：

![这里写图片描述](http://img.blog.csdn.net/20170619172022881?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYmFpeWVfeGluZw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

**6.自定义CAS**

例：

```
public class CompareAndSwapTest {
    public static void main(String[] args) {
        final CompareAndSwap cas = new CompareAndSwap();

        for (int i = 0; i < 10; i++) {
            new Thread(new Runnable() {

                @Override
                public void run() {
                    int expectedValue = cas.get();
                    boolean b = cas.compareAndSet(expectedValue, (int)(Math.random() * 101));
                    System.out.println(b);
                }
            }).start();
        }
    }

}

class CompareAndSwap{
    private int value;

    //获取内存值
    public synchronized int get(){
        return value;
    }

    //比较
    public synchronized int compareAndSwap(int expectedValue, int newValue){
        int oldValue = value;

        if(oldValue == expectedValue){
            this.value = newValue;
        }

        return oldValue;
    }

    //设置
    public synchronized boolean compareAndSet(int expectedValue, int newValue){
        return expectedValue == compareAndSwap(expectedValue, newValue);
    }
}

```

输出结果（不唯一）：

```
true
false
false
false
true
true
true
true
true
true
```

<br>
<br>
本人才疏学浅，若有错误，还请指出 
谢谢！

参考资料：[CAS原理深度分析](http://zl198751.iteye.com/blog/1848575)