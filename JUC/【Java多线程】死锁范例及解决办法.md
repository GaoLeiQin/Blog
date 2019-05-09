**1.定义**

两个或两个以上的进程在执行过程中，因争夺资源而造成的一种互相等待的现象，若无外力作用，它们都将无法推进下去。此时称系统处于死锁状态或系统产生了死锁，这些永远在互相等待的进程称为死锁进程。由于资源占用是互斥的，当某个进程提出申请资源后，使得有关进程在无外力协助下，永远分配不到必需的资源而无法继续运行

**2.产生的原因**

* 系统资源的竞争

系统资源的竞争导致系统资源不足，以及资源分配不当，导致死锁。

* 进程运行推进顺序不合适

进程在运行过程中，请求和释放资源的顺序不当，会导致死锁。


**3.产生死锁的四个必要条件**

* 互斥条件：一个资源每次只能被一个进程使用，即在一段时间内某 资源仅为一个进程所占有。此时若有其他进程请求该资源，则请求进程只能等待。

* 请求与保持条件：进程已经保持了至少一个资源，但又提出了新的资源请求，而该资源 已被其他进程占有，此时请求进程被阻塞，但对自己已获得的资源保持不放。

* 不可剥夺条件：进程所获得的资源在未使用完毕之前，不能被其他进程强行夺走，即只能 由获得该资源的进程自己来释放（只能是主动释放﴿。

* 循环等待条件: 若干进程间形成首尾相接循环等待资源的关系


**4.死锁实例：**

例

```
public class DeathTest {
    public static void main(String[] args) {
        Object o1 = new Object();
        Object o2 = new Object();

        Thread t1 = new Thread(new T1(o1,o2));
        Thread t2 = new Thread(new T2(o1,o2));

        t1.start();
        //若主线程睡眠>=1s，则不会发生死锁,
        // 因为t1睡眠<1s,当t1醒来准备执行o2锁的方法时，若主线程睡眠<1s，主线程已执行过了，t2就会执行，然后就会发生死锁。
        //若主线程睡眠>=1s时，主线程还没执行，这时就会执行主线程，主线程执行完后才会去执行t2，而这时o2锁方法已执行，t2不会锁住o2对象，最后执行t2。
        try {
         Thread.sleep(999);
         } catch (InterruptedException e) {
         e.printStackTrace();
         }
        System.out.println(Thread.currentThread().getName());

        t2.start();
        }
}

    class T1 implements Runnable {
        Object o1;
        Object o2;

        T1(Object o1, Object o2) {
            this.o1 = o1;
            this.o2 = o2;
        }

        @Override
        public void run() {
            synchronized (o1) {
                try {
                    //睡眠1秒，为了让T2锁住o2对象
                    Thread.sleep(1000);
                    System.out.println("t1 -- o1");
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                synchronized (o2) {
                    System.out.println("t1 -- o2");
                }
            }
        }
    }

    class T2 implements Runnable{
        Object o1;
        Object o2;

        T2(Object o1,Object o2){
            this.o1 = o1;
            this.o2 = o2;
        }

        @Override
        public void run() {
            synchronized(o2){
                try {
                    Thread.sleep(3000);
                    System.out.println("t2 -- o2");
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                synchronized(o1){
                    System.out.println("t2 -- o1");
                }
            }
        }
    }

```

输出（一直在运行）：

```
main
t1 -- o1
t2 -- o2
```


用jstack命令打印线程堆栈信息如图

![这里写图片描述](http://img.blog.csdn.net/20170619144239308?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYmFpeWVfeGluZw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

分析：可以看出Thread-0等待Thread-1，而Thread-1也在等待Thread-0，从而导致死锁


**6.解决死锁的四种方法**

* 预防死锁。

　　这是一种较简单和直观的事先预防的方法。方法是通过设置某些限制条件，去破坏产生死锁的四个必要条件中的一个或者几个，来预防发生死锁。预防死锁是一种较易实现的方法，已被广泛使用。但是由于所施加的限制条件往往太严格，可能会导致系统资源利用率和系统吞吐量降低。

* 避免死锁。

　　该方法同样是属于事先预防的策略，但它并不须事先采取各种限制措施去破坏产生死锁的的四个必要条件，而是在资源的动态分配过程中，用某种方法去防止系统进入不安全状态，从而避免发生死锁。

* 检测死锁。

　　这种方法并不须事先采取任何限制性措施，也不必检查系统是否已经进入不安全区，此方法允许系统在运行过程中发生死锁。但可通过系统所设置的检测机构，及时地检测出死锁的发生，并精确地确定与死锁有关的进程和资源，然后采取适当措施，从系统中将已发生的死锁清除掉。

* 解除死锁。

　　这是与检测死锁相配套的一种措施。当检测到系统中已发生死锁时，须将进程从死锁状态中解脱出来。常用的实施方法是撤销或挂起一些进程，以便回收一些资源，再将这些资源分配给已处于阻塞状态的进程，使之转为就绪状态，以继续运行。死锁的检测和解除措施，有可能使系统获得较好的资源利用率和吞吐量，但在实现上难度也最大。

注：还有一种避免死锁的方法：银行家算法

7.脏读问题

若set方法加锁，而get方法没有加锁，获取的或许就是set之前的数据，导致脏读问题。
<br>
<br>
本人才疏学浅，若有错误，还请指出 
谢谢！