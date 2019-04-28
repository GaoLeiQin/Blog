
**1.定义**

ReadWriteLock 维护了一对相关的锁，一个用于只读操作，另一个用于写入操作。只要没有writer，读取锁可以由多个reader 线程同时保持。写入锁是独占的。

**2.适用性**

* ReadWriteLock 读取操作通常不会改变共享资源，但执行写入操作时，必须独占方式来获取锁。对于读取操作占多数的数据结构。ReadWriteLock 能提供比独占锁更高的并发性。而对于只读的数据结构，其中包含的不变性可以完全不需要考虑加锁操作。

* 特别适用于写少读多的情况

**3.功能**

读写锁表示也有两个锁，一个是读操作相关的锁，也称为共享锁；另一个是写操作相关的锁，也叫排他锁。

**4.特性**

* 公平性：非公平锁（默认）,为了防止写线程饿死，规则是：当等待队列头部结点是独占模式（即要获取写锁的线程）时，只有获取独占锁线程可以抢占，而试图获取共享锁的线程必须进入队列阻塞；当队列头部结点是共享模式（即要获取读锁的线程）时，试图获取独占和共享锁的线程都可以抢占。

* 公平锁，利用AQS的等待队列，线程按照FIFO的顺序获取锁，因此不存在写线程一直等待的问题。

* 重入性：读写锁均是可重入的，读/写锁重入次数保存在了32位int state的高/低16位中。而单个读线程的重入次数，则记录在ThreadLocalHoldCounter类型的readHolds里。

* 锁降级：写线程获取写入锁后可以获取读取锁，然后释放写入锁，这样就从写入锁变成了读取锁，从而实现锁降级。

* 锁获取中断：读取锁和写入锁都支持获取锁期间被中断。

* 条件变量：写锁提供了条件变量(Condition)的支持，这个和独占锁ReentrantLock一致，但是读锁却不允许，调用readLock().newCondition()会抛出UnsupportedOperationException异常。


**5.阻塞情况**

* 读，读之间不阻塞，

* 读阻塞写，

* 写阻塞读

* 写阻塞写

**6.约束** 

* 当任一线程持有写锁或读锁时，其他线程不能获得写锁； 

* 当任一线程持有写锁时，其他线程不能获取读锁； 

* 多个线程可以同时持有读锁。

**7.实例**

例1：一写多读

```
public class ReadWriteLockTest {
    public static void main(String[] args) {
        ReadWriteLockDemo rw = new ReadWriteLockDemo();

        new Thread(new Runnable() {

            @Override
            public void run() {
                rw.set((int)(Math.random() * 101));
            }
        }, "Write:").start();


        for (int i = 0; i < 10; i++) {
            new Thread(new Runnable() {

                @Override
                public void run() {
                    rw.get();
                }
            }).start();
        }
    }

}

class ReadWriteLockDemo{

    private int number = 0;

    private ReadWriteLock lock = new ReentrantReadWriteLock();

    //读
    public void get(){
        lock.readLock().lock(); //上锁

        try{
            System.out.println(Thread.currentThread().getName() + " : " + number);
        }finally{
            lock.readLock().unlock(); //释放锁
        }
    }

    //写
    public void set(int number){
        lock.writeLock().lock();

        try{
            System.out.println(Thread.currentThread().getName());
            this.number = number;
        }finally{
            lock.writeLock().unlock();
        }
    }
}

```

输出结果：

```
Write:
Thread-0 : 87
Thread-1 : 87
Thread-2 : 87
Thread-4 : 87
Thread-6 : 87
Thread-8 : 87
Thread-9 : 87
Thread-5 : 87
Thread-7 : 87
Thread-3 : 87
```

例2：多写多读

```
public class ReadWriteLock {
    public static void main(String[] args) {
        Queue q = new Queue();
        for (int i = 0; i < 5; i++) {
            new Thread(){
                @Override
                public void run() {
                    q.put(new Random().nextInt(100));
                    q.get();

                }
            }.start();
        }
    }
}

class Queue{
    private Object data = null;
    private ReentrantReadWriteLock rwl = new ReentrantReadWriteLock();
    public void get() {
        rwl.readLock().lock();
        try {
            System.out.println(Thread.currentThread().getName()+"ready read data!");
            Thread.sleep((long)Math.random()*1000);
            System.out.println(Thread.currentThread().getName()+"have ready read data!"+data);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }finally {
            rwl.readLock().unlock();
        }
    }

    public void put(Object datas) {
        rwl.writeLock().lock();
        try {
            System.out.println(Thread.currentThread().getName()+"ready write read data!");
            Thread.sleep((long)Math.random()*1000);
            this.data = datas;
            System.out.println(Thread.currentThread().getName()+"have write read data!"+data);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }finally {
            rwl.writeLock().unlock();
        }
    }
}

```

输出结果：

```
Thread-2ready write read data!
Thread-2have write read data!98
Thread-1ready write read data!
Thread-1have write read data!7
Thread-3ready write read data!
Thread-3have write read data!38
Thread-0ready write read data!
Thread-0have write read data!64
Thread-4ready write read data!
Thread-4have write read data!64
Thread-2ready read data!
Thread-2have ready read data!64
Thread-0ready read data!
Thread-3ready read data!
Thread-1ready read data!
Thread-4ready read data!
Thread-0have ready read data!64
Thread-4have ready read data!64
Thread-1have ready read data!64
Thread-3have ready read data!64
```

<br>
<br>
本人才疏学浅，若有错误，还请指出 
谢谢！

