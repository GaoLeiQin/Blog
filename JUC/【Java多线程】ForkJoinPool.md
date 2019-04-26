**1.定义**

Fork/Join框架是Java7提供了的一个用于并行执行任务的框架， 是一个把大任务分割成若干个小任务，最终汇总每个小任务结果后得到大任务结果的框架

**2.与线程池的区别**

（1）采用工作窃取模式（work-stealing）：某个线程从其他队列里窃取任务来执行。

   为了减少窃取任务线程和被 窃取任务线程之间的竞争，通常会使用双端队列，被窃取任务线程永远从自己双端队列的头部拿 任务执行，而窃取任务的线程永远从别人双端队列的尾部拿任务执行。

（2）需要实现compute方法(RecursiveTask类的抽象方法)，而RecursiveTask类继承于ForkJoinTask类，在这个方法里，首先需要判断任务是否足够小，如果足够小就执行任务。如果不足够小，就必须分割 成两个子任务，每个子任务在调用fork方法时，又会进入到compute方法。使用join方法会等待子任务执行完并得到其结果。

  当执行新的任务时它可以将其拆分分成更小的任务执行，并将小任务加到线程队列中，然后再从一个随机线程的队列中偷一个并把它放在自己的队列中。

（3）fork/join框架的优势体现在对其中包含的任务的处理方式上.在一般的线程池中，如果一个线程正在执行的任务由于某些原因无法继续运行，那么该线程会处于等待状态。而在fork/join框架实现中，如果某个子问题由于等待另外一个子问题的完成而无法继续运行。那么处理该子问题的线程会主动寻找其他尚未运行的子问题来执行.这种方式减少了线程的等待时间，提高了性能。

（4）它同ThreadPoolExecutor一样，也实现了Executor和ExecutorService接口。它使用了一个无限队列来保存需要执行的任务，而线程的数量则是通过构造函数传入，如果没有向构造函数中传入希望的线程数量，那么当前计算机可用的CPU数量会被设置为线程数量作为默认值。

图解：

![这里写图片描述](http://img.blog.csdn.net/20170620174516895?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYmFpeWVfeGluZw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

**3.使用Fork/join框架的步骤**

(1) ForkJoinTask ： 创建一个ForkJoin任务，可以继承子类 ： RecursiveAction：用于没有返回结果的任务。RecursiveTask：用于有返回结果的任务。

（2）ForkJoinPool：ForkJoinTask需要通过ForkJoinPool来执行。 任务分割出的子任务会添加到当前工作线程所维护的双端队列中，进入队列的头部。当 一个工作线程的队列里暂时没有任务时，它会随机从其他工作线程的队列的尾部获取一个任务。

执行方法：

```
//直接获取结果
Long sum = pool.invoke(task);
//通过Future接口获取结果
Future<Long> future = pool.submit(task);
Long sum = future.get();
```

**4.实例**

```
public class ForkJoinPoolTest {
    public static void main(String[] args) {
        Instant start = Instant.now();

        ForkJoinPool pool = new ForkJoinPool();

        ForkJoinTask<Long> task = new ForkJoinSumCalculate(0L, 50000000L);

        //获取结果法一
        Long sum = pool.invoke(task);
        //法二
        Future<Long> future = pool.submit(task);

        try {
            System.out.println("Future  : " + future.get());
        } catch (InterruptedException e) {
            e.printStackTrace();
        } catch (ExecutionException e) {
            e.printStackTrace();
        }


        System.out.println("计算结果：" + sum);

        Instant end = Instant.now();

        System.out.println("耗费时间为：" + Duration.between(start, end).toMillis());//166-1996-10590
    }
}


class ForkJoinSumCalculate extends RecursiveTask<Long>{

    private static final long serialVersionUID = -259195479995561737L;

    private long start;
    private long end;

    private static final long THURSHOLD = 10000L;  //临界值

    public ForkJoinSumCalculate(long start, long end) {
        this.start = start;
        this.end = end;
    }

    @Override
    protected Long compute() {
        long length = end - start;

        if(length <= THURSHOLD){
            long sum = 0L;

            for (long i = start; i <= end; i++) {
                sum += i;
            }

            return sum;
        }else{
            long middle = (start + end) / 2;

            ForkJoinSumCalculate left = new ForkJoinSumCalculate(start, middle);
            left.fork(); //进行拆分，同时压入线程队列

            ForkJoinSumCalculate right = new ForkJoinSumCalculate(middle+1, end);
            right.fork(); //

            return left.join() + right.join();
        }
    }

}
```

输出结果：

```
Future  : 1250000025000000
计算结果：1250000025000000
耗费时间为：99

```

<br>
<br>
本人才疏学浅，若有错误，请指出
谢谢！