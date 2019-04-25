###**CountDownLatch 闭锁**

**1.引入**

CountDownLatch也叫闭锁，在JDK1.5被引入 ，java.util.concurrent 包中提供了多种并发容器类来改进同步容器的性能。

**2.定义**

* CountDownLatch实现主要基于java同步器AQS

* CountDownLatch 一个同步辅助类，在完成一组正在其他线程中执行的操作之前，它允许一个或多个线程一直等待。

* CountDownLatch内部会维护一个初始值为线程数量的计数器，主线程执行await方法，如果计数器大于0，则阻塞等待。当一个线程完成任务后，计数器值减1。当计数器为0时，表示所有的线程已经完成任务，等待的主线程被唤醒继续执行。

3.闭锁可以延迟线程的进度直到其到达终止状态，闭锁可以用来确保某些活动直到其他活动都完成才继续执行：

（1）确保某个计算在其需要的所有资源都被初始化之后才继续执行;

（2）确保某个服务在其依赖的所有其他服务都已经启动之后才启动;

（3）等待直到某个操作所有参与者都准备就绪再继续执行。

4.构造器中的计数值（count）实际上就是闭锁需要等待的线程数量。这个值只能被设置一次，而且CountDownLatch没有提供任何机制去重新设置这个计数值。

5.与CountDownLatch的第一次交互是主线程等待其他线程。主线程必须在启动其他线程后立即调用CountDownLatch.await()方法。这样主线程的操作就会在这个方法上阻塞，直到其他线程完成各自的任务。

6.每调用一次countDown方法，在构造函数中初始化的count值就减1。所以当N个线程都调用了这个方法，count的值等于0，然后主线程就能通过await()方法，恢复执行自己的任务

7.使用场景

* 实现最大的并行性：有时我们想同时启动多个线程，实现最大程度的并行性。例如，我们想测试一个单例类。如果我们创建一个初始计数为1的CountDownLatch，并让所有线程都在这个锁上等待，那么我们可以很轻松地完成测试。我们只需调用 一次countDown()方法就可以让所有的等待线程同时恢复执行。

* 开始执行前等待n个线程完成各自任务：例如应用程序启动类要确保在处理用户请求前，所有N个外部系统已经启动和运行了。

* 死锁检测：一个非常方便的使用场景是，你可以使用n个线程访问共享资源，在每次测试阶段的线程数目是不同的，并尝试产生死锁。

8.实例

```
public class CountDownLatchTest {
    public static void main(String[] args) {
        final CountDownLatch latch = new CountDownLatch(3);
        LatchDemo ld = new LatchDemo(latch);

        long start = System.currentTimeMillis();

        for (int i = 0; i < 3; i++) {
            new Thread(ld).start();
        }

        try {
            latch.await();
        } catch (InterruptedException e) {
        }

        long end = System.currentTimeMillis();

        System.out.println("耗费时间为：" + (end - start));
    }

}

class LatchDemo implements Runnable {

    private CountDownLatch latch;

    public LatchDemo(CountDownLatch latch) {
        this.latch = latch;
    }

    @Override
    public void run() {
        try {
            for (int i = 0; i < 5; i++) {
                if (i % 2 == 0) {
                    System.out.println(i);
                }
            }
        } finally {
            latch.countDown();
        }

    }

}

```

输出结果：

```
0
2
4
0
2
4
0
2
4
耗费时间为：1
```

###**CyclicBarrier（回环栅栏）**


**1.定义：**

* 通过它可以实现让一组线程等待至某个状态之后再全部同时执行。叫做回环是因为当所有等待线程都被释放以后，CyclicBarrier可以被重用。我们暂且把这个状态就叫做barrier，当调用await()方法之后，线程就处于barrier了。

* CyclicBarrier实现主要基于ReentrantLock

**2.CyclicBarrier提供2个构造器：**

（1）	CyclicBarrier(int parties, Runnable barrierAction)

（2）	CyclicBarrier(int parties)

注：参数parties指让多少个线程或者任务等待至barrier状态；参数barrierAction为当这些线程都达到barrier状态时会执行的内容。

**3.与闭锁的区别**

（1）

* CountDownLatch : 一个线程(或者多个)， 等待另外N个线程完成某个事情之后才能执行。

* CyclicBarrier : N个线程相互等待，任何一个线程完成之前，所有的线程都必须等待。

分析：对于CountDownLatch来说，重点是那个“一个线程”, 是它在等待， 而另外那N的线程在把“某个事情”做完之后可以继续等待，可以终止。而对于CyclicBarrier来说，重点是那N个线程，他们之间任何一个没有完成，所有的线程都必须等待。

（2）
CountDownLatch 是计数器, 线程完成一个就记一个, 就像 报数一样, 只不过是递减的.而CyclicBarrier更像一个水闸, 线程执行就想水流, 在水闸处都会堵住, 等到水满(线程到齐)了, 才开始泄流.

**4.实例**

```
public class CyclicBarrierTest {
    public static void main(String[] args) {
        ExecutorService service = Executors.newCachedThreadPool();
        final CyclicBarrier cb = new CyclicBarrier(3);
        for (int i = 0; i < 10; i++) {
            Runnable runnable = new Runnable() {
                @Override
                public void run() {
                    try {
                        Thread.sleep(new Random().nextInt(100));
                        System.out.println("线程 " + Thread.currentThread().getName() + " 即将到达集合地点  当前已有 " + cb.getNumberWaiting());
                        cb.await();
                        Thread.sleep(new Random().nextInt(100));
                        System.out.println(Thread.currentThread().getName() + "即将到达集合地点 有--- " + cb.getNumberWaiting());
                        cb.await();
                        Thread.sleep(new Random().nextInt(100));
                        System.out.println(Thread.currentThread().getName() + "即将到达集合地点 当前有、、、 " + cb.getNumberWaiting());
                        cb.await();
                    } catch (Exception e) {
                        e.printStackTrace();
                    }
                }
            };
            service.execute(runnable);
        }
        service.shutdown();
    }
}
```

运行结果：

```
线程 pool-1-thread-4 即将到达集合地点  当前已有 0
线程 pool-1-thread-5 即将到达集合地点  当前已有 1
线程 pool-1-thread-7 即将到达集合地点  当前已有 2
线程 pool-1-thread-6 即将到达集合地点  当前已有 0
线程 pool-1-thread-8 即将到达集合地点  当前已有 1
线程 pool-1-thread-9 即将到达集合地点  当前已有 2
线程 pool-1-thread-1 即将到达集合地点  当前已有 0
线程 pool-1-thread-2 即将到达集合地点  当前已有 1
pool-1-thread-7即将到达集合地点 有--- 2
pool-1-thread-8即将到达集合地点 有--- 0
pool-1-thread-2即将到达集合地点 有--- 1
pool-1-thread-7即将到达集合地点 当前有、、、 2
线程 pool-1-thread-10 即将到达集合地点  当前已有 0
pool-1-thread-1即将到达集合地点 有--- 1
线程 pool-1-thread-3 即将到达集合地点  当前已有 2
pool-1-thread-5即将到达集合地点 有--- 0
pool-1-thread-4即将到达集合地点 有--- 1
pool-1-thread-10即将到达集合地点 有--- 2
pool-1-thread-2即将到达集合地点 当前有、、、 0
pool-1-thread-5即将到达集合地点 当前有、、、 1
pool-1-thread-10即将到达集合地点 当前有、、、 2
pool-1-thread-6即将到达集合地点 有--- 0
pool-1-thread-8即将到达集合地点 当前有、、、 1
pool-1-thread-4即将到达集合地点 当前有、、、 1
pool-1-thread-1即将到达集合地点 当前有、、、 0
pool-1-thread-3即将到达集合地点 有--- 1
pool-1-thread-9即将到达集合地点 有--- 2
pool-1-thread-6即将到达集合地点 当前有、、、 0
pool-1-thread-9即将到达集合地点 当前有、、、 1
pool-1-thread-3即将到达集合地点 当前有、、、 2

```

###**Semaphore**

**定义**

Semaphore是一种基于计数的信号量。它可以设定一个阈值，基于此，多个线程竞争获取许可信号，做完自己的申请后归还，超过阈值后，线程申请许可信号将会被阻塞。Semaphore可以用来构建一些对象池，资源池之类的，比如数据库连接池，我们也可以创建计数为1的Semaphore，将其作为一种类似互斥锁的机制，这也叫二元信号量，

**应用场景**

Semaphore可以用于做流量控制，特别公用资源有限的应用场景，比如数据库连接。假如有一个需求，要读取几万个文件的数据，因为都是IO密集型任务，我们可以启动几十个线程并发的读取，但是如果读到内存后，还需要存储到数据库中，而数据库的连接数只有10个，这时我们必须控制只有十个线程同时获取数据库连接保存数据，否则会报错无法获取数据库连接。这个时候，我们就可以使用Semaphore来做流控，

例：

```
public class SemaphoreTest {
    public static void main(String[] args) {
        ExecutorService service = Executors.newCachedThreadPool();
        final Semaphore sp = new Semaphore(2);
        for (int i = 0; i < 4; i++) {
            service.execute(new Runnable() {
                @Override
                public void run() {
                    try {
                        sp.acquire();
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                    System.out.println("线程 "+Thread.currentThread().getName()+" 进入  当前已有 "+(3-sp.availablePermits()));
                    try {
                        Thread.sleep(2000);
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                    System.out.println(Thread.currentThread().getName() + "号线程即将离开-- ");
                    sp.release();
                    System.out.println(Thread.currentThread().getName() + "已离开--当前有 "+(3-sp.availablePermits()));
                }
            });
        }

    }
}

```

运行结果

```
线程 pool-1-thread-1 进入  当前已有 2
线程 pool-1-thread-2 进入  当前已有 3
pool-1-thread-1号线程即将离开-- 
pool-1-thread-2号线程即将离开-- 
pool-1-thread-1已离开--当前有 3
线程 pool-1-thread-3 进入  当前已有 3
线程 pool-1-thread-4 进入  当前已有 3
pool-1-thread-2已离开--当前有 3
pool-1-thread-4号线程即将离开-- 
pool-1-thread-3号线程即将离开-- 
pool-1-thread-3已离开--当前有 1
pool-1-thread-4已离开--当前有 1
```


###**Exchanger**

Exchanger可以在两个线程之间交换数据，只能是2个线程，他不支持更多的线程之间互换数据。

当线程A调用Exchange对象的exchange()方法后，他会陷入阻塞状态，直到线程B也调用了exchange()方法，然后以线程安全的方式交换数据，之后线程A和B继续运行

例：

```
public class ExchangeTest {
    public static void main(String[] args) {
        ExecutorService service = Executors.newCachedThreadPool();
        final Exchanger exchanger = new Exchanger();
            service.execute(new Runnable() {
                @Override
                public void run() {
                    try {
                        String s1 = "dream";
                        System.out.println("线程 " + Thread.currentThread().getName() + " 正在把数据 "+s1+"换出去");
                        Thread.sleep(new Random().nextInt(10));
                        String s2 = (String)exchanger.exchange(s1);
                        System.out.println(Thread.currentThread().getName() + "换回的数据为 "+s2);
                    } catch (Exception e) {
                        e.printStackTrace();
                    }
                }
            });

        service.execute(new Runnable() {
            @Override
            public void run() {
                try {
                    String s1 = "bad blood";
                    System.out.println("线程 " + Thread.currentThread().getName() + " 正在把数据 "+s1+"换出去");
                    Thread.sleep(new Random().nextInt(10));
                    String s2 = (String)exchanger.exchange(s1);
                    System.out.println(Thread.currentThread().getName() + "换回的数据为 "+s2);
                } catch (Exception e) {
                    e.printStackTrace();
                }
            }
        });
            service.shutdown();
        }
}

```

运行结果

```
线程 pool-1-thread-1 正在把数据 dream换出去
线程 pool-1-thread-2 正在把数据 bad blood换出去
pool-1-thread-2换回的数据为 dream
pool-1-thread-1换回的数据为 bad blood
```


<br>
<br>
本人才疏学浅，若有错误，还请指出 
谢谢！