**1.概念**

（1）进程： 一个执行中的程序，每一个进程执行都有一个执行的顺序或叫执行单元。

（2）线程：是进程中的一个独立的控制单元，线程控制着进程的执行或说一个程序执行多个任务，每一个任务称为一个线程，一个进程中至少有一个线程。

**2.创建线程的方式**

（1）实现接口（常用）

* 实现Runnable接口（无返回值）

例：
```java
public class TicketWindow implements Runnable {
        @Override
        public void run() {
            
        }

        public static void main(String[] args) {
            TicketWindow t = new TicketWindow(); 
            new Thread(t).start();
        }
}
```

* 实现Callable接口（有返回值）

例：

```
public class CallableTest {
    public static void main(String[] args) {
        ThreadTest td = new ThreadTest();

        //1.执行 Callable 方式，需要 FutureTask 实现类的支持，用于接收运算结果。
        FutureTask<Integer> result = new FutureTask<Integer>(td);

        new Thread(result).start();

        //2.接收线程运算后的结果
        try {
            Integer sum = result.get();  //FutureTask 可用于 闭锁
            System.out.println(sum);
        } catch (InterruptedException | ExecutionException e) {
            e.printStackTrace();
        }
    }

}

class ThreadTest implements Callable<Integer> {

    @Override
    public Integer call() throws Exception {
        int sum = 0;

        for (int i = 0; i <= 100000; i++) {
            sum += i;
        }

        return sum;
    }

}
```


（2）继承Thread类

例：

```
public class Tick extends Thread {
        @Override
        public void run() {
            
        }

        public static void main(String[] args) {
            new Tick().start(); 
        }
}
```

（3）小提示：

创建线程时，不要调用Thread类或Runnable的run方法，直接调用run方法，只会执行同一个线程中的任务，而不会启动新线程，应该调用Thread.start()方法，这个方法将创建一个执行run方法的新线程。

**3.中断线程**

（1）Interrupted是一个静态方法，它检测当前的线程是否被中断，会产生副作用，它将当前线程的中断状态重置为false,清除该线程的中断状态。

（2）isInterrupt是一个实例方法，检验是否有线程被中断，但不会改变中断状态。

（3）Interrupt（）向线程发送中断请求，线程的中断状态将被设置为true，如果目前该线程被一个sleep调用阻塞，那么，InterruptedException异常被抛出。

**4.线程的状态**

<font color="blue">线程状态转换图</font>
![这里写图片描述](http://img.blog.csdn.net/20170413221120881?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYmFpeWVfeGluZw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

<br/>
（1）新建状态（New）：当线程对象对创建后，即进入了新建状态，如：Thread t = new MyThread();

（2）就绪状态（Runnable）：当调用线程对象的start()方法（t.start();），线程即进入就绪状态。处于就绪状态的线程，只是说明此线程已经做好了准备，随时等待CPU调度执行，并不是说执行了t.start()此线程立即就会执行；

（3）运行状态（Running）：当CPU开始调度处于就绪状态的线程时，此时线程才得以真正执行，即进入到运行状态。注：就     绪状态是进入到运行状态的唯一入口，也就是说，线程要想进入运行状态执行，首先必须处于就绪状态中；

（4）阻塞状态（Blocked）：当一个线程试图获取一个内部的对象锁，而该锁被其他线程持有，则该线程进入到阻塞状态。根据阻塞产生的原因不同，阻塞状态又可以分为三种：

*等待阻塞：运行状态中的线程执行wait()方法，使本线程进入到等待阻塞状态；

*同步阻塞 -- 线程在获取synchronized同步锁失败(因为锁被其它线程所占用)，它会进入同步阻塞状态；

*其他阻塞 -- 通过调用线程的sleep()或join()或发出了I/O请求时，线程会进入到阻塞状态。当sleep()状态超时、join()等待线程终止或者超时、或者I/O处理完毕时，线程重新转入就绪状态。

（5）死亡状态（Dead）：线程执行完了或者因异常退出了run()方法，该线程结束生命周期。

<br/>
**5.线程属性**

（1）线程优先级

*方法：
setPriority（）    设置线程优先级；
yield（）        暂停当前执行的线程对象，并执行其他线程；
toString（）  返回该线程的字符串形式，包括线程名称、优先级和线程组。

*等级  
默认是5 ， 一共10级
*MAX_PRIORITY  · 最高优先级 · 优先等级为10
*MIN_PRIORITY  · 最低优先级 · 优先等级为1
*NORM_PRIORITY  · 默认优先级 · 优先等级为5

（2）守护线程（后台线程）：为其他线程提供服务

方法：
setDamen(true) 将该线程标记为守护线程或用户线程，该方法必须在启动线程前调用，
isDamen( ) 测试线程是否为守护线程

注：
只要当前JVM中尚存在任何一个非守护线程没有结束，守护线程就全部工作；只有当最后一个非守护线程结束时，守护线程随着JVM一同结束工作。
Daemon的作用是为其他线程的运行提供便利服务，守护线程最典型的应用就是 GC (垃圾回收器)，它就是一个很称职的守护者。

（3）主线程：JVM调用程序main()所产生的线程。

（4）当前线程：指通过Thread.currentThread()来获取的进程。

（5）join（n）等待该线程终止的时间最长为n毫秒，加入线程。

例：当A线程执行到了B线程的join（）方法时，A就会等待，等B线程执行完，A才会执行；join（）可以用来临时加入线程执行。

**6.同步与锁**

（1）同步：
如果有多个线程因等待同一个对象的标志位而处于阻塞状态时，当该对象的标志位恢复到1（解锁）状态时，只有一个线程能够进入同步代码执行其它的线程仍处于阻塞状态。

（2）锁
*synchronized关键字
例：
synchronized (Object) { // 代码块 }

*方法

```
public synchronized void output() {
    public void run () {
	
	}
}
```

*Lock(需要解锁)  是一个接口
例：

```
 public void mainMethod(int i) {
        lock.lock();
        try {
            while(bShouldSub) {
                try {
                    condition.await();
                } catch (Exception e) {
                    e.printStackTrace();
                }
            }
            for (int j = 0; j < 100; j++) {
                System.out.println("main shape of you!"+i);
            }
            bShouldSub = true;
            condition.signal();
        }finally {
            lock.unlock();
        }

    }
```

（3）等待方法与唤醒

*wait() 导致线程进入到等待状态直到它被通知，只能在一个同步方法中调用
*notify() 随机选择一个在该对象上调用wait()方法的线程，解除其阻塞状态
notifyAll() 解除全部调用wait方法对象的阻塞状态。

（4）volatile关键字
*保证可见性
*禁止指令重排序

**7.死锁**

（1）概念：
某个任务在等待另一个任务，而后者又等待别的任务，这样一直下去就会出现死锁。

（2）以下四个条件同时满足就会发生死锁

*互斥条件：任务使用的资源中至少有一个是不能共享的，；
*至少有一个任务它必须持有一个资源且正在等待获取一个当前被别的任务持有的资源
*资源 不能被任务抢占，任务必须把资源释放当作普通事件；
*必须有循环等待。

**8.执行器与线程池**

（1）若程序中创建了大量的生命期很短的线程，应该用线程池，一个线程池中包含了许多准备运行的空闲线程，创建线程池会减少并发线程的数目；

（2）执行器（Executor）类有许多静态工厂方法用来构建线程池

*newCachedThreadPool  必要时创建新线程，空闲线程被保留60秒，之后终止线程。
*newFixedThreadPool  该线程包含固定数量的线程，空闲线程会一直保留
*newSingleThreadExector 只有一个线程的池，该线程顺序执行每一个提交的任务。

（3）使用线程池时应注意

*调用Exectors类中静态的方法newCachedThreadPool或newFixedThreadPool
*调用submit提交Runable或Callable对象
*如果想要取消一个任务，或提交Callable对象，保存好返回的Future对象
*当不再提交任何任务时，调用shutdown（）；

（4）创建线程池

例：

```
public class ThreadPoolTest {
    public static void main(String[] args) {
        ExecutorService threadPool = Executors.newCachedThreadPool();
            threadPool.execute(new Runnable() {
                @Override
                public void run() {
                    for (int j = 0; j < 10; j++) {
                        System.out.println( Thread.currentThread().getName()+" -- "+j);
                    }
                }
            });
    }
}
```
<br/>
<br/>

本人才疏学浅，如有错误，请指出~
谢谢！