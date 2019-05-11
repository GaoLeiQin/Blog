接着上篇文章，我接下来继续介绍线程池的工作原理，如果你还没有看上篇，我建议最好浏览一下：[线程池的工作原理详解（上）](http://blog.csdn.net/baiye_xing/article/details/73481090)

###**Executors 工具类 **

**1.定义**

Executors是java线程池的工厂类，通过它可以快速初始化一个符合业务需求的线程池。

**2.常用方法**

（1） **newFixedThreadPool方法**  ：创建固定大小的线程池

* 适用于为了满足资源管理的需求，而需要限制当前线程数量的应用场景，即负载比较重的服务器

FixedThreadPool方法源码如下：

```
    public static ExecutorService newFixedThreadPool(int nThreads) {
        return new ThreadPoolExecutor(nThreads, nThreads,
                                      0L, TimeUnit.MILLISECONDS,
                                      new LinkedBlockingQueue<Runnable>());
    }
```

分析：从源码可以看出，其实是初始化了一个指定线程数的线程池，其中corePoolSize == maximumPoolSize，使用LinkedBlockingQuene作为阻塞队列，如果线程池没有可执行任务时，也不会释放线程。

**运行FixedThreadPool图解：**

![这里写图片描述](http://img.blog.csdn.net/20170620114544615?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYmFpeWVfeGluZw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

分析：

1）如果当前运行的线程数少于corePoolSize，则创建新线程来执行任务。

2）在线程池完成预热之后（当前运行的线程数等于corePoolSize），将任务加入LinkedBlockingQueue。

3）线程执行完1）中的任务后，会在循环中反复从LinkedBlockingQueue获取任务来执行。

FixedThreadPool使用无界队列LinkedBlockingQueue作为线程池的工作队列（队列的容量为
Integer.MAX_VALUE）。使用无界队列作为工作队列会对线程池带来如下影响。

①当线程池中的线程数达到corePoolSize后，新任务将在无界队列中等待，因此线程池中
的线程数不会超过corePoolSize。

②由于①，使用无界队列时maximumPoolSize将是一个无效参数。

③	由于①和②，使用无界队列时keepAliveTime将是一个无效参数。

④	由于使用无界队列，运行中的FixedThreadPool（未执行方法shutdown()或
shutdownNow()）不会拒绝任务（不会调用RejectedExecutionHandler.rejectedExecution方法）。


（2）**newCachedThreadPool() 方法** ：缓存线程池，线程池的数量不固定，可以根据需求自动的更改数量。

大小无界的线程池，适用于执行很多的短期异步任务的小程序，或是负载较轻的服务器。

CachedThreadPool方法源码如下

```
    public static ExecutorService newCachedThreadPool() {
        return new ThreadPoolExecutor(0, Integer.MAX_VALUE,
                                      60L, TimeUnit.SECONDS,
                                      new SynchronousQueue<Runnable>());
    }
```

分析：

* 初始化一个可以缓存线程的线程池，默认缓存60s，线程池的线程数可达到Integer.MAX_VALUE，即2147483647，内部使用不存储元素的SynchronousQueue作为阻塞队列；

* newCachedThreadPool在没有任务执行时，当线程的空闲时间超过keepAliveTime，会自动释放线程资源，当提交新任务时，如果没有空闲线程，则创建新线程执行任务，会导致一定的系统开销；

**运行CachedThreadPool图解：**

![这里写图片描述](http://img.blog.csdn.net/20170620115717787?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYmFpeWVfeGluZw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

分析：

① ：首先执行SynchronousQueue.offer（Runnable task）。如果当前maximumPool中有空闲线程正在执行SynchronousQueue.poll（keepAliveTime，TimeUnit.NANOSECONDS），那么主线程执行offer操作与空闲线程执行的poll操作配对成功，主线程把任务交给空闲线程执行，execute()方法执行完成；否则执行下面的 2）

② ：当初始maximumPool为空，或者maximumPool中当前没有空闲线程时，将没有线程执行
SynchronousQueue.poll（keepAliveTime，TimeUnit.NANOSECONDS）。这种情况下，①将失败。此时CachedThreadPool会创建一个新线程执行任务，execute()方法执行完成。

③	 ：在 ②中新创建的线程将任务执行完后，会执行
SynchronousQueue.poll（keepAliveTime，TimeUnit.NANOSECONDS）。这个poll操作会让空闲线程最多在SynchronousQueue中等待60秒钟。如果60秒钟内主线程提交了一个新任务（主线程执行①），那么这个空闲线程将执行主线程提交的新任务；否则，这个空闲线程将终止。由于空闲60秒的空闲线程会被终止，因此长时间保持空闲的CachedThreadPool不会使用任何资源。

* CachedThreadPool使用SynchronousQueue，把主线程提交的任务传递给空闲线程执行。

* CachedThreadPool中任务传递示意图如下：

![这里写图片描述](http://img.blog.csdn.net/20170620120048855?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYmFpeWVfeGluZw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

**（3）newSingleThreadExecutor() 方法** ： 创建单个线程池。线程池中只有一个线程

适用于需要保证顺序地执行各个任务；并且在任意时间点，不会有多个线程是活动的应用场景。
  
SingleThreadExecutor方法源码如下

```
    public static ExecutorService newSingleThreadExecutor() {
        return new FinalizableDelegatedExecutorService
            (new ThreadPoolExecutor(1, 1,
                                    0L, TimeUnit.MILLISECONDS,
                                    new LinkedBlockingQueue<Runnable>()));
    }
```

分析：

初始化的线程池中只有一个线程，如果该线程异常结束，会重新创建一个新的线程继续执行任务，唯一的线程可以保证所提交任务的顺序执行，内部使用LinkedBlockingQueue作为阻塞队列。

**运行SingleThreadExecutor图解：**

![这里写图片描述](http://img.blog.csdn.net/20170620121056978?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYmFpeWVfeGluZw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

分析：

1）如果当前运行的线程数少于corePoolSize（即线程池中无运行的线程），则创建一个新线程来执行任务。

2）在线程池完成预热之后（当前线程池中有一个运行的线程），将任务加入Linked-BlockingQueue。

3）线程执行完1中的任务后，会在一个无限循环中反复从LinkedBlockingQueue获取任务来执行。

**（4）newScheduledThreadPool() 方法**: 创建固定大小的线程，可以延迟或定时的执行任务。

适用于需要多个后台线程执行周期任务，同时为了满足资源 管理的需求而需要限制后台线程的数量的应用场景

newScheduledThreadPool方法源码如下：

```
    public static ScheduledExecutorService newScheduledThreadPool(int corePoolSize) {
        return new ScheduledThreadPoolExecutor(corePoolSize);
    }
```
* ScheduledThreadPoolExecutor源码
```
    public ScheduledThreadPoolExecutor(int corePoolSize) {
        super(corePoolSize, Integer.MAX_VALUE, 0, NANOSECONDS,
              new DelayedWorkQueue());
    }
```
* 继承父类的构造方法
```
    public ThreadPoolExecutor(int corePoolSize,
                              int maximumPoolSize,
                              long keepAliveTime,
                              TimeUnit unit,
                              BlockingQueue<Runnable> workQueue) {
        this(corePoolSize, maximumPoolSize, keepAliveTime, unit, workQueue,
             Executors.defaultThreadFactory(), defaultHandler);
    }
```

分析：初始化的线程池可以在指定的时间内周期性的执行所提交的任务，在实际的业务场景中可以使用该线程池定期的同步数据。其实最底层还是通过ThreadPoolExecutor类实现的

**运行ScheduledThreadPoolExecutor图解**

 ![这里写图片描述](http://img.blog.csdn.net/20170620121836130?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYmFpeWVfeGluZw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

分析：

* 当调用ScheduledThreadPoolExecutor的scheduleAtFixedRate()方法或者scheduleWith-
FixedDelay()方法时，会向ScheduledThreadPoolExecutor的DelayQueue添加一个实现了
RunnableScheduledFutur接口的ScheduledFutureTask。

* 线程池中的线程从DelayQueue中获取ScheduledFutureTask，然后执行任务。


###**如何合理配置线程池**

**1.任务特性**

* 任务的性质：CPU密集型任务、IO密集型任务和混合型任务。

* 任务的优先级：高、中和低。

* 任务的执行时间：长、中和短。

* 任务的依赖性：是否依赖其他系统资源，如数据库连接。

**2.处理策略**

性质不同的任务可以用不同规模的线程池分开处理。

* CPU密集型任务应配置尽可能小的 线程，如配置Ncpu+1个线程的线程池。由于IO密集型任务线程并不是一直在执行任务，则应配置尽可能多的线程，如2*Ncpu。

* 混合型的任务，如果可以拆分，将其拆分成一个CPU密集型任务 和一个IO密集型任务，只要这两个任务执行的时间相差不是太大，那么分解后执行的吞吐量将高于串行执行的吞吐量。

* 如果这两个任务执行时间相差太大，则没必要进行分解。可以通过Runtime.getRuntime().availableProcessors()方法获得当前设备的CPU个数。

* 优先级不同的任务可以使用优先级队列PriorityBlockingQueue来处理。它可以让优先级高的任务先执行。

**3.注意** 

* 如果一直有优先级高的任务提交到队列里，那么优先级低的任务可能永远不能执行。执行时间不同的任务可以交给不同规模的线程池来处理，或者可以使用优先级队列，让执行时间短的任务先执行。

* 依赖数据库连接池的任务，因为线程提交SQL后需要等待数据库返回结果，等待的时间越 长，则CPU空闲时间就越长，那么线程数应该设置得尽量大，这样才能更好地利用CPU。

* 建议使用有界队列。有界队列能增加系统的稳定性和预警能力，可以根据需要设大一点儿，如果当时我们设置成无界队列，那么线程池的队列就会越来越多，有可能会撑满内存，导致整个系统不可用，而不只是后台任务出现问题。

###**线程池的监控**

**1.定义**

如果在系统中大量使用线程池，则有必要对线程池进行监控，方便在出现问题时，可以根据线程池的使用状况快速定位问题。可以通过线程池提供的参数进行监控，

**2.监控线程池使用的属性**

* taskCount：线程池需要执行的任务数量。

* completedTaskCount：线程池在运行过程中已完成的任务数量，小于或等于taskCount。

* largestPoolSize：线程池里曾经创建过的最大线程数量。通过这个数据可以知道线程池是否曾经满过。如该数值等于线程池的最大大小，则表示线程池曾经满过。

* getPoolSize：线程池的线程数量。如果线程池不销毁的话，线程池里的线程不会自动销毁，所以这个大小只增不减。

* getActiveCount：获取活动的线程数。通过扩展线程池进行监控。可以通过继承线程池来自定义线程池，重写线程池的 

注：
beforeExecute、afterExecute和terminated方法，也可以在任务执行前、执行后和线程池关闭前执行一些代码来进行监控。例如，监控任务的平均执行时间、最大执行时间和最小执行时间等。

<br>
<br>
本人才疏学浅，若有错，请指出，谢谢！ 
如果你有更好的建议，可以留言我们一起讨论，共同进步！ 
衷心的感谢您能耐心的读完本篇博文。

