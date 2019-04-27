###**ReentrantLock 与synchronized异同**


**1.引入**

经过之前的学习，你肯定知道锁就用synchronized，但今天我再介绍一种锁-Lock

**2.底层实现**

synchronized是基于JVM层面实现的，而Lock是基于JDK层面实现的

**3.使用**

lock不仅要上锁还要释放锁，如果忘记释放锁就会产生死锁的问题，所以，通常需要在finally中进行锁的释放。但是synchronized的使用十分简单

**4.公平锁与非公平锁**

* 锁Lock分为公平锁和非公平锁，默认是非公平的，公平锁表示线程获取锁的顺序按照线程加锁的顺序来分配，即先来先得的FIFO先进先出顺序。而非公平锁就是一种获取锁的抢占机制，是随机获得锁的，和公平锁不一样就是先来不一定先得到锁。

* 对于非公平锁，只要CAS设置同步状态成功，则表示当前线程获取了锁，而公平锁则不同，他的判断条件多了hasQueuePredecessors()方法，即加入了同步队列中当前节点是否有前驱节点的判断，如果该方法返回true，则表示有线程比更早的请求获取锁，因此需要等待前驱线程获取锁并释放之后才能继续获取锁。

###**ReentrantLock**


**1.引入**

在Java 5.0 之前，协调共享对象的访问时可以使用的机制只有synchronized 和volatile 。Java 5.0 后增加了一些新的机制，但并不是一种替代内置锁的方法，而是当内置锁不适用时，作为一种可选择的高级功能。ReentrantLock是synchronized关键字的替代品，

**2.特点**

synchronized使用简单，功能上比较薄弱，ReentrantLock具备sychronized关键字所不具备的特性：

（1）	尝试非阻塞的获取锁：当前线程尝试获取锁，如果这一时刻没有被其他线程获取到，则成功获取并持有锁。

（2）	能被中断地获取锁：与sychronized不同，获取到锁的线程能够响应中断，当获取到锁的线程被中断时，中断的异常将会抛出，同时锁会被释放。

（3）	超时获取锁：在指定的截止时间之前获取锁，如果截止时间到了仍旧无法获取锁，则返回。

（4）	是公平锁

（5）ReentrantLock 实现了Lock 接口，并提供了与synchronized 相同的互斥性和内存可见性。但相较于synchronized 提供了更高的处理锁的灵活性。

**3.实现原理**

* 队列同步器AbstractQueuedSynchronizer（以下简称同步器），是用来构建锁或者其他同步组 件的基础框架，它使用了一个int成员变量表示同步状态，通过内置的FIFO队列来完成资源获 取线程的排队工作。

* 同步器的主要使用方式是继承，子类通过继承同步器并实现它的抽象方法来管理同步状 态，在抽象方法的实现过程中免不了要对同步状态进行更改，这时就需要使用同步器提供的3 个方法（getState()、setState(int newState)和compareAndSetState(int expect,int update)）来进行操作，因为它们能够保证状态的改变是安全的。

* 子类推荐被定义为自定义同步组件的静态内部类，同步器自身没有实现任何同步接口，它仅仅是定义了若干同步状态获取和释放的方法来供自定义同步组件使用，同步器既可以支持独占式地获取同步状态，也可以支持共享式地获 取同步状态，这样就可以方便实现不同类型的同步组件（ReentrantLock、 ReentrantReadWriteLock和CountDownLatch等）。

**4.常用方法**

|方法|功能|
|:----|:-----|
|int getHoldCount()| 查询当前线程保持此锁的次数|
|protected  Thread getOwner() |返回目前拥有此锁的线程，如果此锁不被任何线程拥有，则返回 null|
| boolean hasQueuedThreads() | 查询是否有些线程正在等待获取此锁| 
| boolean hasWaiters(Condition condition) | 查询是否有些线程正在等待与此锁有关的给定条件|
| boolean isFair() | 如果此锁的公平设置为 true，则返回 true| 
| boolean isHeldByCurrentThread() | 查询当前线程是否保持此锁| 
| boolean isLocked() | 查询此锁是否由任意线程保持|
| void lock() |          获取锁| 
 |void lockInterruptibly() | 如果当前线程未被中断，则获取锁| 
| Condition newCondition() | 返回用来与此 Lock 实例一起使用的 Condition 实例| 
| boolean tryLock() | 仅在调用时锁未被另一个线程保持的情况下，才获取该锁| 
| boolean tryLock(long timeout, TimeUnit unit) |如果锁在给定等待时间内没有被另一个线程保持，且当前线程未被中断，则获取该锁|
| void unlock() |试图释放此锁|


**5.实例**

例：

```
public class LockSynchronized {
    public static void main(String[] args) {
        OutPut o = new OutPut();
        new Thread(new Runnable() {
            @Override
            public void run() {
                while (true) {
                    try {
                        Thread.sleep(500);
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                    o.output("Dream it possible!");
                }
            }
        }).start();

        new Thread(new Runnable() {
            @Override
            public void run() {
                while (true) {
                    try {
                        Thread.sleep(500);
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                    o.output("We don't talk anymore!");
                }
            }
        }).start();
    }

    static class OutPut{
        Lock lk = new ReentrantLock();
        public void output(String name) {
            int len = name.length();
            lk.lock();
            try {
                for (int i = 0; i < len; i++) {
                    System.out.print(name.charAt(i));
                }
                System.out.println();
            }finally {
                lk.unlock();
            }

        }
    }
}

```

输出结果：

```
Dream it possible!
We don't talk anymore!
Dream it possible!
We don't talk anymore!
Dream it possible!
We don't talk anymore!
We don't talk anymore!
Dream it possible!
//一直在输出.....
```

分析：从结果可以看出，每次输出一整句，而不是半句或者完整的一句，说明加锁成功


###**Condition 控制线程通信**

1.Condition 接口描述了可能会与锁有关联的条件变量。这些变量在用法上与使用Object.wait 访问的隐式监视器类似，但提供了更强大的功能。需要特别指出的是，单个Lock 可能与多个Condition 对象关联。为了避免兼容性问题，Condition 方法的名称与对应的Object 版本中的不同。Condition是依赖Lock对象的

2.在Condition 对象中，与wait、notify 和notifyAll 方法对应的分别是await、signal 和signalAll。

3.Condition 实例实质上被绑定到一个锁上。要为特定Lock 实例获得Condition 实例，请使用其newCondition() 方法。

4.Condition内部有一个等待队列，等待队列是一个FIFO的队列，在队列中的每个节点都包含了一个线程引用，该线程就是 在Condition对象上等待的线程，如果一个线程调用了Condition.await()方法，那么该线程将会 释放锁、构造成节点加入等待队列并进入等待状态。事实上，节点的定义复用了同步器中节点 的定义，也就是说，同步队列和等待队列中节点类型都是同步器的静态内部类。

5.Condition能够更加精细的控制多线程的休眠与唤醒。对于同一个锁，可以创建多个Condition，在不同的情况下使用不同的Condition。

6.常用方法：

|方法|功能|
|:----|:-----|
 |void await() |造成当前线程在接到信号或被中断之前一直处于等待状态|
|boolean await(long time, TimeUnit unit) |造成当前线程在接到信号、被中断或到达指定等待时间之前一直处于等待状态|
| long awaitNanos(long nanosTimeout) |造成当前线程在接到信号、被中断或到达指定等待时间之前一直处于等待状态| 
| void awaitUninterruptibly() | 造成当前线程在接到信号之前一直处于等待状态|
| boolean awaitUntil(Date deadline) |造成当前线程在接到信号、被中断或到达指定最后期限之前一直处于等待状态| 
| void signal() |唤醒一个等待线程|
| void signalAll() |唤醒所有等待线程|


###**生产者消费者模型**

之前介绍了利用wait和notify 方法实现的生产者消费者模型，既然学习了ReentrantLock与Condition，那我就利用ReentrantLock与Condition实现的一个生产者消费者模式

例：

```
public class AlternateTest {
    public static void main(String[] args) {
        AlternateDemo ad = new AlternateDemo();

        new Thread(new Runnable() {
            @Override
            public void run() {

                for (int i = 1; i <= 5; i++) {
                    ad.loopA(i);
                }

            }
        }, "A").start();

        new Thread(new Runnable() {
            @Override
            public void run() {

                for (int i = 1; i <= 5; i++) {
                    ad.loopB(i);
                }

            }
        }, "B").start();

        new Thread(new Runnable() {
            @Override
            public void run() {

                for (int i = 1; i <= 5; i++) {
                    ad.loopC(i);

                    System.out.println("-----------------------------------");
                }

            }
        }, "C").start();
    }

}

class AlternateDemo{

    private int number = 1; //当前正在执行线程的标记

    private Lock lock = new ReentrantLock();
    private Condition condition1 = lock.newCondition();
    private Condition condition2 = lock.newCondition();
    private Condition condition3 = lock.newCondition();

    //循环第几轮
    public void loopA(int totalLoop){
        lock.lock();

        try {
            //1. 判断
            if(number != 1){
                condition1.await();
            }

            //2. 打印
            for (int i = 1; i <= 1; i++) {
                System.out.println(Thread.currentThread().getName() + "\t" + i + "\t" + totalLoop);
            }

            //3. 唤醒
            number = 2;
            condition2.signal();
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            lock.unlock();
        }
    }

    public void loopB(int totalLoop){
        lock.lock();

        try {
            //1. 判断
            if(number != 2){
                condition2.await();
            }

            //2. 打印
            for (int i = 1; i <= 2; i++) {
                System.out.println(Thread.currentThread().getName() + "\t" + i + "\t" + totalLoop);
            }

            //3. 唤醒
            number = 3;
            condition3.signal();
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            lock.unlock();
        }
    }

    public void loopC(int totalLoop){
        lock.lock();

        try {
            //1. 判断
            if(number != 3){
                condition3.await();
            }

            //2. 打印
            for (int i = 1; i <= 3; i++) {
                System.out.println(Thread.currentThread().getName() + "\t" + i + "\t" + totalLoop);
            }

            //3. 唤醒
            number = 1;
            condition1.signal();
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            lock.unlock();
        }
    }

}

```

输出结果

```
A	1	1
B	1	1
B	2	1
C	1	1
C	2	1
C	3	1
-----------------------------------
A	1	2
B	1	2
B	2	2
C	1	2
C	2	2
C	3	2
-----------------------------------
A	1	3
B	1	3
B	2	3
C	1	3
C	2	3
C	3	3
-----------------------------------
A	1	4
B	1	4
B	2	4
C	1	4
C	2	4
C	3	4
-----------------------------------
A	1	5
B	1	5
B	2	5
C	1	5
C	2	5
C	3	5
-----------------------------------
```

小结：从代码中可以看出，相比于Object的wait和notify方法，Condition的await和signal方法更具有灵活性。

<br>
<br>
本人才疏学浅，若有错误，还请指出 
谢谢！