###**为什么使用线程池**

* 线程池为线程生命周期开销问题和资源不足问题提供了解决方案，因为线程是稀缺资源，如果被无限制的创建，不仅会消耗系统资源，还会降低系统的稳定性，合理的使用线程池对线程进行统一分配、调优和监控；

*  多线程技术可以解决处理器单元内多个线程执行的问题，它可以显著减少处理器单元的闲置时间，增加处理器单元的吞吐能力；

###**什么时候使用线程池**

* 需要大量的线程来完成任务，且完成任务的时间比较短，如WEB服务器完成网页请求这样的任务。因为单个任务小，而任务数量巨大，比如一个热门网站的点击次数。 但对于长时间的任务，比如一个ftp连接请求，线程池的优点就不明显了。因为ftp会话时间相对于线程的创建时间长多了。

* 对性能要求苛刻的应用，比如要求服务器迅速相应客户请求。

* 接受突发性的大量请求，但不至于使服务器因此产生大量线程的应用。突发性大量客户请求，在没有线程池情况下，将产生大量线程，虽然理论上大部分操作系统线程数目最大值不是问题，短时间内产生大量线程可能使内存到达极限。

###**使用线程池的好处**

* 降低资源消耗；

* 提高响应速度；

* 提高线程的可管理性。


###**线程池类之间的关系**

UML图解

![这里写图片描述](http://img.blog.csdn.net/20170620104153582?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYmFpeWVfeGluZw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

分析：我介绍一下常用的几个类或接口

* ExecutorService ：线程池接口，继承于Executor接口，多了Submit、shutdown等方法

* ScheduledExecutorService：功能和Timer类似，解决那些需要任务重复执行的问题，负责线程的调度

* ThreadPoolExecutor ：线程池的实现类

* ScheduledThreadPoolExecutor：继承ThreadPoolExecutor的ScheduledExecutorService接口实现，周期性任务调度的类实现。

* Executors ：一个工具类，提供几个创建线程池的方法

* Future 接口：它和FutureTask类（实现了Future接口）都用来表示异步计算的结果。

###**ThreadPoolExecutor **

**1.定义**

一个 ExecutorService，它使用可能的几个池线程之一执行每个提交的任务，通常使用 Executors 工厂方法配置。

**2.线程池的处理流程**

![这里写图片描述](http://img.blog.csdn.net/20170620110741502?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYmFpeWVfeGluZw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

**3.ThreadPoolExecutor的构造方法**

```
    public ThreadPoolExecutor(int corePoolSize,
                              int maximumPoolSize,
                              long keepAliveTime,
                              TimeUnit unit,
                              BlockingQueue<Runnable> workQueue,
                              ThreadFactory threadFactory,
                              RejectedExecutionHandler handler) {
        if (corePoolSize < 0 ||
            maximumPoolSize <= 0 ||
            maximumPoolSize < corePoolSize ||
            keepAliveTime < 0)
            throw new IllegalArgumentException();
        if (workQueue == null || threadFactory == null || handler == null)
            throw new NullPointerException();
        this.corePoolSize = corePoolSize;
        this.maximumPoolSize = maximumPoolSize;
        this.workQueue = workQueue;
        this.keepAliveTime = unit.toNanos(keepAliveTime);
        this.threadFactory = threadFactory;
        this.handler = handler;
    }
```

**4.构造方法参数详解**

（1）corePoolSize ：线程池中的核心线程数

当提交一个任务时，线程池创建一个新线程执行任务，直到当前线程数等于corePoolSize；如果当前线程数为corePoolSize，继续提交的任务被保存到阻塞队列中，等待被执行；如果执行了线程池的prestartAllCoreThreads()方法，线程池会提前创建并启动所有核心线程。

（2）maximumPoolSize ：线程池中允许的最大线程数

如果当前阻塞队列满了，判断是否达到最大线程数，若没有，则创建新的线程执行任务，否则进行拒绝策略。但如果使用了无界的任务队列这个参数就没用了。

（3）keepAliveTime ：线程空闲时的存活时间

当线程没有任务执行时，继续存活的时间；默认情况下，该参数只在线程数大于corePoolSize时才有用。

（4）unit ：keepAliveTime的单位

（5）workQueue ：用来保存等待被执行的任务的阻塞队列

任务必须实现Runable接口，一共4有个阻塞队列：

* ArrayBlockingQueue：基于数组结构的有界阻塞队列，此队列按FIFO（先进先出）原则对元素进行排序。

* LinkedBlockingQuene：基于链表结构的无界阻塞队列，按FIFO排序任务，吞吐量通常要高于ArrayBlockingQuene；

* SynchronousQuene：一个不存储元素的阻塞队列，每个插入操作必须等到另一个线程调用移除操作，否则插入操作一直处于阻塞状态，吞吐量通常要高于LinkedBlockingQuene；

* priorityBlockingQuene：具有优先级的无界阻塞队列；

（6）threadFactory ：创建线程的工厂

可以选择DefaultThreadFactory，将创建一个同线程组且默认优先级的线程，也可以选择PrivilegedThreadFactory，使用访问权限创建一个权限控制的线程。但默认采用DefaultThreadFactory，当然也可以创建自定义线程工厂给每个新建的线程设置一个具有识别度的线程名。

（7）handler

线程池的饱和策略，当阻塞队列满了，且没有空闲的工作线程，如果继续提交任务，必须采取一种策略处理该任务，线程池提供了4种策略：

* AbortPolicy：直接抛出RejectedExecutionException异常，默认策略；

* CallerRunsPolicy：如果发现线程池还在运行，就直接运行这个线程，即用调用者所在的线程来执行任务；

* DiscardOldestPolicy：丢弃阻塞队列中靠最前的任务，并执行当前任务；

* DiscardPolicy：直接丢弃任务；

当然也可以根据应用场景实现RejectedExecutionHandler接口，自定义饱和策略，如记录日志或持久化存储不能处理的任务。

**5.提交任务的方法**

（1）submit()方法 ：用于提交需要返回值的任务

线程池会返回一个future类型的对象，通过这个 future对象可以判断任务是否执行成功，并且可以通过future的get()方法来获取返回值，但get()方 法会阻塞当前线程直到任务完成，而使用get（long timeout，TimeUnit unit）方法则会阻塞当前线 程一段时间后立即返回，这时候有可能任务没有执行完。

（2）execute()方法 ：用于提交不需要返回值的任务，

无法判断任务是否被线程池执行成功，线程池在执行excute方法时，会出现以下几种情况:

① 如果当前运行的线程少于corePoolSize，则创建新线程来执行任务（需要获得全局锁）；

②如果运行的线程等于或多于corePoolSize ,则将任务加入BlockingQueue；

③如果无法将任务加入BlockingQueue(队列已满并且正在运行的线程数量小于 maximumPoolSize)，则创建新的线程来处理任务（需要获得全局锁）

④如果创建新线程将使当前运行的线程超出maxiumPoolSize（队列已满并且正在运行的线程数量大于或等于 maximumPoolSize），任务将被拒绝，并调用RejectedExecutionHandler.rejectedExecution()方法(线程池会抛出异常，告诉调用者"我不能再接受任务了");

**以上四种情况图解：**
![这里写图片描述](http://img.blog.csdn.net/20170620113716938?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYmFpeWVfeGluZw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)


线程池采取上述的流程进行设计是为了减少获取全局锁的次数。在线程池完成预热（当前运行的线程数大于或等于corePoolSize）之后，几乎所有的excute方法调用都执行步骤②，还有两种情况如下：

⑤当一个线程完成任务时，它会从队列中取下一个任务来执行；

⑥当一个线程无事可做，超过一定的时间（keepAliveTime）时，线程池会判断，如果当前运行的线程数大于 corePoolSize，那么这个线程就被停掉。所以线程池的所有任务完成后，它最终会收缩到 corePoolSize 的大小

**6.关闭线程池的方法**

（1）关闭线程池方法的原理

遍历线 程池中的工作线程，然后逐个调用线程的interrupt方法来中断线程，所以无法响应中断的任务 可能永远无法终止。

（2）shutdown方法（常用） ：将线程池的状态设置成SHUTDOWN状态，然后中断所有没有正在执行任务的线程。

（3）shutdownNow方法：首先将线程池的状态设置成 STOP，然后尝试停止所有的正在执行或暂停任务的线程，并返回等待执行任务的列表

（4）可以调用isShutdown方法来判断线程池是否关闭



###**ScheduledThreadPoolExecutor **


**1.引入**

自jdk1.5开始，Java开始提供ScheduledThreadPoolExecutor类来支持周期性任务的调度，在这之前，这些工作需要依靠Timer/TimerTask或者其它第三方工具来完成。但Timer有着不少缺陷，如Timer是单线程模式，调度多个周期性任务时，如果某个任务耗时较久就会影响其它任务的调度；如果某个任务出现异常而没有被catch则可能导致唯一的线程死掉而所有任务都不会再被调度。ScheduledThreadPoolExecutor解决了很多Timer存在的缺陷。

**2.类的继承关系**

![这里写图片描述](http://img.blog.csdn.net/20170620163738853?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYmFpeWVfeGluZw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

注：绿色虚线表示实现的接口，黄色实线表示继承的类，绿色实线表示接口间的继承。

**3.执行周期任务的步骤**

（1）图解：

![这里写图片描述](http://img.blog.csdn.net/20170620164310195?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYmFpeWVfeGluZw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

分析：

1）线程1从DelayQueue中获取已到期的ScheduledFutureTask（DelayQueue.take()）。到期任务
是指ScheduledFutureTask的time大于等于当前时间。

2）线程1执行这个ScheduledFutureTask。

3）线程1修改ScheduledFutureTask的time变量为下次将要被执行的时间。

4）线程1把这个修改time之后的ScheduledFutureTask放回DelayQueue中（Delay-Queue.add()）。


**（2）DelayQueue.take()方法剖析**

DelayQueue.take()源码如下

```
    public E take() throws InterruptedException {
        final ReentrantLock lock = this.lock;
        lock.lockInterruptibly();　　　　　　　　　　　　 // 1
        try {
            for (;;) {
                E first = q.peek();
                if (first == null) {
                    available.await();　　　　　　　　　　 // 2.1
                } else {
                    long delay = first.getDelay(TimeUnit.NANOSECONDS);
                    if (delay > 0) {
                        long tl = available.awaitNanos(delay);　　 // 2.2
                    } else {
                        E x = q.poll();　　　　　　　　　　 // 2.3.1
                        assert x != null;
                        if (q.size() != 0)
                            available.signalAll();　　　　　　　　 // 2.3.2
                        return x;
                    }
                }
            }
        } finally {
            lock.unlock();　　　　　　　　　　　　　　 // 3
        }
    }
```

执行take()方法图解

![这里写图片描述](http://img.blog.csdn.net/20170620164635854?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYmFpeWVfeGluZw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

获取任务分为3个步骤。

①获取Lock。

②获取周期任务。

* 如果PriorityQueue为空，当前线程到Condition中等待；否则执行下面的2.2。

* 如果PriorityQueue的头元素的time时间比当前时间大，到Condition中等待到time时间；否则执行下面的2.3。

* 获取PriorityQueue的头元素（2.3.1）；如果PriorityQueue不为空，则唤醒在Condition中等待的所有线程（2.3.2）。

③释放Lock。

**（3）DelayQueue.add()方法剖析**

DelayQueue.add()源码如下

```
    public boolean offer(E e) {
        final ReentrantLock lock = this.lock;
        lock.lock();　　　　　　　　　　 // 1
        try {
            E first = q.peek();
            q.offer(e);　　　　　　　　 // 2.1
            if (first == null || e.compareTo(first) < 0)
                available.signalAll();　　　 // 2.2
            return true;
        } finally {
            lock.unlock();　　　　　　　　 // 3
        }
    }
```
执行add()方法图解

![这里写图片描述](http://img.blog.csdn.net/20170620165457809?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYmFpeWVfeGluZw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

添加任务分为3个步骤。

①获取Lock。

②添加任务。

* 向PriorityQueue添加任务。

* 如果在上面2.1中添加的任务是PriorityQueue的头元素，唤醒在Condition中等待
的所有线程。

③释放Lock。

**4.注意**

每一个被调度的任务都会由线程池中一个线程去执行，因此任务是并发执行的，相互之间不会受到干扰。需 要注意的是，只有当任务的执行时间到来时，ScheduedExecutor 才会真正启动一个线程，其余时间 ScheduledExecutor 都是在轮询任务的状态。

**5.实例**

```
public class ScheduledThreadPoolTest {
    public static void main(String[] args) throws Exception {
        ScheduledExecutorService pool = Executors.newScheduledThreadPool(5);

        for (int i = 0; i < 5; i++) {
            Future<Integer> result = pool.schedule(new Callable<Integer>(){

                @Override
                public Integer call() throws Exception {
                    int num = new Random().nextInt(100);//生成随机数
                    System.out.println(Thread.currentThread().getName() + " : " + num);
                    return num;
                }

            }, 1, TimeUnit.SECONDS);

            System.out.println(result.get());
        }

        //关闭周期任务
        pool.shutdown();
    }

}

```
输出结果（不唯一）

```
pool-1-thread-1 : 73
73
pool-1-thread-1 : 60
60
pool-1-thread-2 : 79
79
pool-1-thread-1 : 20
20
pool-1-thread-3 : 74
74
```

###**Futuretask **

**1.定义**

FutureTask除了实现Future接口外，还实现了Runnable接口。因此，FutureTask可以交给Executor执行，也可以由调用线程直接执行（FutureTask.run()）。

**2.FutureTask的3种状态**

1）未启动。FutureTask.run()方法还没有被执行之前，FutureTask处于未启动状态。当创建一个FutureTask，且没有执行FutureTask.run()方法之前，这个FutureTask处于未启动状态。

2）已启动。FutureTask.run()方法被执行的过程中，FutureTask处于已启动状态。

3）已完成。FutureTask.run()方法执行完后正常结束，或被取消（FutureTask.cancel（…）），或执行FutureTask.run()方法时抛出异常而异常结束，FutureTask处于已完成状态。

状态迁移图

![这里写图片描述](http://img.blog.csdn.net/20170620171256687?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYmFpeWVfeGluZw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

**3.适用场景**

（1）FutureTask可用于异步获取执行结果或取消执行任务的场景。通过传入Runnable或者Callable的任务给FutureTask，直接调用其run方法或者放入线程池执行，之后可以在外部通过FutureTask的get方法异步获取执行结果，

（2）FutureTask非常适合用于耗时的计算，主线程可以在完成自己的任务后，再去获取结果。另外，FutureTask还可以确保即使调用了多次run方法，它都只会执行一次Runnable或者Callable任务，或者通过cancel取消FutureTask的执行等。

（3）FutureTask执行多任务计算的使用场景，利用FutureTask和ExecutorService，可以用多线程的方式提交计算任务，主线程继续执行其他任务，当主线程需要子线程的计算结果时，在异步获取子线程的执行结果。

（4）FutureTask在高并发环境下确保任务只执行一次，在很多高并发的环境下，往往我们只需要某些任务只执行一次。这种使用情景FutureTask的特性恰能胜任。举一个例子，假设有一个带key的连接池，当key存在时，即直接返回key对应的对象；当key不存在时，则创建连接

（5）用ConcurrentHashMap的情况下，几乎可以避免加锁的操作，性能大大提高，但是在高并发的情况下有可能出现Connection被创建多次的现象。这时最需要解决的问题就是当key不存在时，创建Connection的动作能放在connectionPool之后执行，这正是FutureTask发挥作用的时机，


由于篇幅有限，Executors类的剖析就留到下篇介绍：[线程池的工作原理详解（下）](http://blog.csdn.net/baiye_xing/article/details/73504129)
<br>
<br>
本人才疏学浅，若有错，请指出，谢谢！ 
如果你有更好的建议，可以留言我们一起讨论，共同进步！ 
衷心的感谢您能耐心的读完本篇博文。

参考资料：[深入分析java线程池的实现原理](http://www.jianshu.com/p/87bff5cc8d8c)

