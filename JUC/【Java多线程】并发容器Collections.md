###**1.迭代器与ConcurrentModificationException**

使用Iterator迭代容器或使用使用for-each遍历容器，在迭代过程中修改容器会抛出ConcurrentModificationException异常。想要避免出现ConcurrentModificationException，就必须在迭代过程持有容器的锁。但是若容器较大，则迭代的时间也会较长。那么需要访问该容器的其他线程将会长时间等待。从而会极大降低性能。

若不希望在迭代期间对容器加锁，可以使用"克隆"容器的方式。使用线程封闭，由于其他线程不会对容器进行修改，可以避免ConcurrentModificationException。但是在创建副本的时候，存在较大性能开销。

###**2.并发容器的特性：**

* 根据具体场景进行设计，尽量避免使用锁，提高容器的并发访问性。

* 并发容器定义了一些线程安全的复合操作。

* 并发容器在迭代时，可以不封闭在synchronized中。但是未必每次看到的都是"最新的、当前的"数据。如果说将迭代操作包装在synchronized中，可以达到"串行"的并发安全性，那么并发容器的迭代达到了"脏读"。

###**3.并发容器的分类**

* CopyOnWriteArrayList和CopyOnWriteArraySet分别代替List和Set，主要是在遍历操作为主的情况下来代替同步的List和同步的Set，这也就是上面所述的思路：迭代过程要保证不出错，除了加锁，另外一种方法就是"克隆"容器对象。

* ConcurrentHashMap是HashMap的线程安全版本

* ConcurrentSkipListMap是TreeMap的线程安全版本

* ConcurrentSkipListSet是TreeSet的线程安全版本

* ConcurrentLinkedQueue是使用非阻塞的方式实现的基于链接节点的无界的线程安全队列，它采用先进先出的规则对节点进行排序，添加元素到队列的尾部；获取元素时返回队列头部的元素。它采用CAS算法实现。



###**4.阻塞队列**

**1.一共7个阻塞队列：**
（1）ArrayBlockingQueue ：一个由数组结构组成的有界阻塞队列。

（2）LinkedBlockingQueue ：一个由链表结构组成的无界阻塞队列。

* 构造方法可以设定容量范围。如果未指定容量，则它等于Integer.MAX_VALUE。除非插入节点会使队列超出容量，否则每次插入后会动态地创建链接节点。

（3）PriorityBlockingQueue ：一个支持优先级排序的无界阻塞队列。

* 调用take方法时才去排序

（4）DelayQueue：一个使用优先级队列实现的无界阻塞队列。

* 队列中的元素必须实现Delay接口，在创建元素时，可以指定从队列获取元素的限制时长。只有在延迟期满，元素才能被提出队列。

* 队列中的元素都要延迟时间（超时时间），只有一个元素达到了延时时间才能出队列，也就是说每次从队列中获取的元素总是最先到达延时的元素。

* 应用场景 
 *  缓存系统的设计：可以用DelayQueue保存缓存元素。使用一个线程循环检查队列中的元素，一旦可以从DelayQueue中获取元素时，表示缓存有效期到了。
 * 定时任务调度：使用DelayQueue保存当天将会执行的任务和执行时间，一旦从DelayQueue中获取到任务就开始执行，比如TimeQueue就是使用DelayQueue实现的。

（5）SynchronousQueue：一个不存储元素的阻塞队列。

* 每个put操作必须等待一个take操作，否则不能继续添加元素

* SynchronousQueue内部其实没有任何一个元素，容量是0，严格说并不是一种容器。由于队列没有容量，因此不能调用peek操作。因为仅在试图要移除元素时，该元素才存在。除非另一个线程试图移除某个元素，否则也不能（使用任何方法）插入元素；也不能迭代队列，因为其中没有元素可用于迭代。

* SynchronousQueue更像是一种信道（管道），资源从一个方向快速传递到另一方向

（6）LinkedTransferQueue：一个由链表结构组成的无界阻塞队列。

（7）LinkedBlockingDeque：一个由链表结构组成的双向阻塞队列。

* 双向队列允许在队列头和尾部进行入队出队操作。Deque不仅具有FIFO的Queue实现，也有FILO的实现，也就是不仅可以实现队列，也可以实现一个堆栈


**2.阻塞队列实现的原理**

使用通知模式实现。所谓通知模式，就是当生产者往满的队列里添加元素时会阻塞住生产者，当消费者消费了一个队列中的元素后，会通知生产者当前队列不可用。通过查看jdk源码发现ArrayBlockingQueue使用了Condition来实现。

**3.park方法**

park(boolean isAbsolute, long time)方法会阻塞当前线程，只有以下四种情况中的一种发生时，该方法才会返回

* 与park对应的unpark执行或已经执行时。注意：已经执行是指unpark先执行，然后再执行的park。

* 线程被中断时。

* 如果参数中的time不是零，等待了指定的毫秒数时。

* 发生异常现象时。这些异常事先无法确定。

**4.LockSupport**

（1）定义：
LockSupport是用来创建锁和其他同步类的基本线程阻塞原语。 LockSupport中的park() 和 unpark() 的作用分别是阻塞线程和解除阻塞线程，而且park()和unpark()不会遇到“Thread.suspend 和Thread.resume所可能引发的死锁”问题。

（2）park会响应中断请求，但是不会有中断异常抛出，不能直接分辨出被park的线程有没有被中断，可以用检查中断标志位来判断，用Thread.interrupted()得到中断标志。


###**5.ConcurrentHashMap 锁分段机制**

1.引入

Java 5.0 在java.util.concurrent 包中提供了多种并发容器类来改进同步容器的性能

2.ConcurrentHashMap 同步容器类是Java 5 增加的一个线程安全的哈希表。对与多线程的操作，介于HashMap 与Hashtable 之间。内部采用“锁分段”机制替代Hashtable 的独占锁。进而提高性能。

3.此包还提供了设计用于多线程上下文中的Collection 实现：

ConcurrentHashMap、ConcurrentSkipListMap、ConcurrentSkipListSet、CopyOnWriteArrayList 和CopyOnWriteArraySet。当期望许多线程访问一个给定collection 时，ConcurrentHashMap 通常优于同步的HashMap，ConcurrentSkipListMap 通常优于同步的TreeMap。当期望的读数和遍历远远大于列表的更新数时，CopyOnWriteArrayList （写入复制，效率低，适合迭代）优于同步的ArrayList。

4 ConcurrentHashMap实现技术是保证HashEntry几乎是不可变的。HashEntry代表每个hash链中的一个节点


###**6.实例**

**（1）CopyOnWriteArrayList例子**

```
public class CopyOnWriteArrayListTest {
    public static void main(String[] args) {
        HelloThread ht = new HelloThread();

        for (int i = 0; i < 5; i++) {
            new Thread(ht).start();
        }
    }


}

class HelloThread implements Runnable{

//	private static List<String> list = Collections.synchronizedList(new ArrayList<String>());

    private static CopyOnWriteArrayList<String> list = new CopyOnWriteArrayList<>();

    static{
        list.add("AA");
        list.add("BB");
        list.add("CC");
    }

    @Override
    public void run() {

        Iterator<String> it = list.iterator();

        while(it.hasNext()){
            System.out.println(it.next());

            list.add("---");
        }

    }

}


```

输出结果（不唯一）：

```
AA
AA
AA
AA
BB
BB
AA
BB
BB
CC
CC
BB
CC
CC
CC
---
```


**（2）ArrayBlockingQueue例子**

```
public class BlockQueueTest {
    public static void main(String[] args) {
        BusinessBlock b = new BusinessBlock();
        new Thread(new Runnable() {
            @Override
            public void run() {
                for (int i = 0; i < 3; i++) {
                    b.sub(i);
                }
            }
        }).start();

        for(int i=0; i < 3; i++) {
            b.mainMethod(i);
        }
    }

    static class BusinessBlock {
        BlockingQueue<Integer> queue1 = new ArrayBlockingQueue<Integer>(1);
        BlockingQueue<Integer> queue2 = new ArrayBlockingQueue<Integer>(1);

        {
            try {
                queue2.put(1);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
        public void mainMethod(int i) {
            try {
                queue1.put(i);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            for (int j = 0; j < 4; j++) {
                System.out.println(j+"  main shape of you!  "+i);
            }
            try {
                queue2.take();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
        public void sub(int i) {
            try {
                queue2.put(i);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            for (int j = 0; j < 2; j++) {
                System.out.println(j+"  sub wildest dreams!  "+i);
            }
            try {
                queue1.take();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }
}


```
输出结果：

```
0  main shape of you!  0
1  main shape of you!  0
2  main shape of you!  0
3  main shape of you!  0
0  sub wildest dreams!  0
1  sub wildest dreams!  0
0  main shape of you!  1
1  main shape of you!  1
2  main shape of you!  1
3  main shape of you!  1
0  sub wildest dreams!  1
1  sub wildest dreams!  1
0  main shape of you!  2
1  main shape of you!  2
2  main shape of you!  2
3  main shape of you!  2
0  sub wildest dreams!  2
1  sub wildest dreams!  2
```

<br>
<br>
本人才疏学浅，若有错误，还请指出 
谢谢！