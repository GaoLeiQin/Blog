###**定义**

生产者消费者模式是通过一个容器来解决生产者和消费者的强耦合问题。生产者和消费者彼此之间不直接通讯，而通过阻塞队列来进行通讯，所以生产者生产完数据之后不用等待消费者处理，直接扔给阻塞队列，消费者不找生产者要数据，而是直接从阻塞队列里取，阻塞队列就相当于一个缓冲区，平衡了生产者和消费者的处理能力。


图解：

![这里写图片描述](http://img.blog.csdn.net/20170619150543105?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYmFpeWVfeGluZw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)


###**实例**

例1：wait、notify方法
```
public class ProductorAndConsumerTest {
    public static void main(String[] args) {
        Clerk2 clerk = new Clerk2();

        Productor2 pro = new Productor2(clerk);
        Consumer2 cus = new Consumer2(clerk);

        new Thread(pro, "2生产者 A").start();
        new Thread(cus, "2消费者 B").start();
        new Thread(pro, "2生产者 C").start();
        new Thread(cus, "2消费者 D").start();
    }
}

//店员
class Clerk2{
	private int product = 0;

	//进货
	public synchronized void get(){//循环次数：0
		while(product >= 1){//为了避免虚假唤醒问题，应该总是使用在循环中
			System.out.println("产品已满！");
			try {
				this.wait();
			} catch (InterruptedException e) {
			}
		}

		System.out.println(Thread.currentThread().getName() + " : " + ++product);
		this.notifyAll();
	}

	//卖货
	public synchronized void sale(){//product = 0; 循环次数：0
		while(product <= 0){
			System.out.println("缺货！");

			try {
				this.wait();
			} catch (InterruptedException e) {
			}
		}

		System.out.println(Thread.currentThread().getName() + " : " + --product);
		this.notifyAll();
	}
}

//生产者
class Productor2 implements Runnable{
	private Clerk2 clerk;

	public Productor2(Clerk2 clerk) {
		this.clerk = clerk;
	}

	@Override
	public void run() {
		for (int i = 0; i < 20; i++) {
			try {
				Thread.sleep(200);
			} catch (InterruptedException e) {
			}

			clerk.get();
		}
	}
}

//消费者
class Consumer2 implements Runnable{
	private Clerk2 clerk;

	public Consumer2(Clerk2 clerk) {
		this.clerk = clerk;
	}

	@Override
	public void run() {
		for (int i = 0; i < 20; i++) {
			clerk.sale();
		}
	}
}

```

输出结果：

```
缺货！
缺货！
2生产者 A : 1
2消费者 D : 0
缺货！
缺货！
2生产者 C : 1
2消费者 B : 0
缺货！
缺货！
2生产者 A : 1
2消费者 D : 0
//中间省略很多行
缺货！
2生产者 C : 1
2消费者 D : 0
缺货！
2生产者 A : 1
2消费者 B : 0
```
分析：只有当生产者生产了数据，消费者才能消费，否则就缺货！

例2：await、signal方法

```
public class TestProductorAndConsumerForLock {

    public static void main(String[] args) {
        Clerk clerk = new Clerk();

        Productor pro = new Productor(clerk);
        Consumer con = new Consumer(clerk);

        new Thread(pro, "生产者 A").start();
        new Thread(con, "消费者 B").start();

        new Thread(pro, "生产者 C").start();
        new Thread(con, "消费者 D").start();
    }

}

class Clerk {
    private int product = 0;

    private Lock lock = new ReentrantLock();
    private Condition condition = lock.newCondition();

    // 进货
    public void get() {
        lock.lock();

        try {
            while (product >= 1) { // 为了避免虚假唤醒，应该总是使用在循环中。
                System.out.println("产品已满！");

                try {
                    condition.await();
                } catch (InterruptedException e) {
                }

            }
            System.out.println(Thread.currentThread().getName() + " : "
                    + ++product);

            condition.signalAll();
        } finally {
            lock.unlock();
        }

    }

    // 卖货
    public void sale() {
        lock.lock();

        try {
            while (product <= 0) {
                System.out.println("缺货！");

                try {
                    condition.await();
                } catch (InterruptedException e) {
                }
            }

            System.out.println(Thread.currentThread().getName() + " : "
                    + --product);

            condition.signalAll();

        } finally {
            lock.unlock();
        }
    }
}

// 生产者
class Productor implements Runnable {

    private Clerk clerk;

    public Productor(Clerk clerk) {
        this.clerk = clerk;
    }

    @Override
    public void run() {
        for (int i = 0; i < 5; i++) {
            try {
                Thread.sleep(200);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }

            clerk.get();
        }
    }
}

// 消费者
class Consumer implements Runnable {

    private Clerk clerk;

    public Consumer(Clerk clerk) {
        this.clerk = clerk;
    }

    @Override
    public void run() {
        for (int i = 0; i < 5; i++) {
            clerk.sale();
        }
    }

}

```

输出结果：

```
缺货！
缺货！
生产者 A : 1
产品已满！
消费者 B : 0
缺货！
缺货！
生产者 C : 1
消费者 B : 0
缺货！
缺货！
生产者 C : 1
产品已满！
消费者 B : 0
缺货！
缺货！
生产者 A : 1
消费者 B : 0
缺货！
缺货！
生产者 C : 1
消费者 B : 0
缺货！
生产者 A : 1
消费者 D : 0
缺货！
生产者 A : 1
消费者 D : 0
缺货！
生产者 C : 1
消费者 D : 0
缺货！
生产者 A : 1
消费者 D : 0
缺货！
生产者 C : 1
消费者 D : 0

```

<br>
<br>
本人才疏学浅，若有错误，还请指出 
谢谢！
