###**引入**

我们先来看一个小例子

```
public class CreatRunnable {

        public static void main(String[] args) {
            //定义售票窗口
            TicketWindow ticket = new TicketWindow();

            Thread t1 = new Thread(ticket);
            Thread t2 = new Thread(ticket);
            Thread t3 = new Thread(ticket);

            //启动线程
            t1.start();
            t2.start();
            t3.start();


        }
}

//售票窗口
class TicketWindow implements Runnable {
    //一共5张票
    private int nums = 5;

    public void show(){
        //先判断是否还有票
        if (nums > 0) {
            //显示售票信息
            System.out.println(Thread.currentThread().getName() + "正在售出第 " + nums + " 票");
            //出票速度1张/秒
            try {
                Thread.sleep(100);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            nums--;
        }
    }
    
    @Override
    public void run() {
        while (true) {
            show();
        }
    }
}

```

输出结果（不唯一）

```
Thread-0正在售出第 5 票
Thread-1正在售出第 5 票
Thread-2正在售出第 5 票
Thread-2正在售出第 4 票
Thread-1正在售出第 3 票
Thread-0正在售出第 2 票
Thread-0正在售出第 1 票
```

分析：从结果可以看出，第五张票被卖了三次，三个线程都去抢第五张票了，这如果发生在实际的售票系统中，可是要出很大问题的，我们如何解决这种多线程抢票问题呢，其实只要在售票的方法前加把锁（Synchronized）就行了。

###**Synchronized方法**

例2：

```
public class CreatRunnable {

    public static void main(String[] args) {
        //定义售票窗口
        TicketWindow ticket = new TicketWindow();

        Thread t1 = new Thread(ticket);
        Thread t2 = new Thread(ticket);
        Thread t3 = new Thread(ticket);

        //启动线程
        t1.start();
        t2.start();
        t3.start();


    }
}

//售票窗口
class TicketWindow implements Runnable {
    //一共5张票
    private int nums = 5;

	//注意：方法加锁了
    public synchronized void show(){
        //先判断是否还有票
        if (nums > 0) {
            //显示售票信息
            System.out.println(Thread.currentThread().getName() + "正在售出第 " + nums + " 票");
            //出票速度1张/秒
            try {
                Thread.sleep(100);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            nums--;
        }
    }

    @Override
    public void run() {
        while (true) {
            show();
        }
    }
}

```

运行结果（出票结果唯一）

```
Thread-1正在售出第 5 票
Thread-0正在售出第 4 票
Thread-2正在售出第 3 票
Thread-0正在售出第 2 票
Thread-1正在售出第 1 票
```

分析：我们可以看出，售票的结果正常了，因为我给售票的方法加锁了，同一时间只有一个线程可以去访问售票方法，Synchronized到底有什么作用呢，我会在接下来介绍。

###**Synchronized关键字分析**

####**1.定义**

Synchronize是Java语言的关键字，当它用来修饰一个方法或者一个代码块的时候，能够保证在同一时刻最多只有一个线程执行该段代码。synchronized关键字是不能继承的.

####**2.修饰的对象**

* 修饰代码块，被修饰的代码块称为同步语句块，其作用的范围是大括号{}括起来的代码，作用的对象是调用这个代码块的对象； 

* 修饰方法，被修饰的方法称为同步方法，其作用的范围是整个方法，作用的对象是调用这个方法的对象； 锁是当前实例对象。

* 修饰静态方法，其作用的范围是整个静态方法，作用的对象是这个类的所有对象； 锁是当前类的Class类对象。

* 修饰类，其作用的范围是synchronized后面括号括起来的部分，作用主的对象是这个类的所有对象。


####**3.实现原理**

* 执行完同步代码块就会释放对象锁 

* 在执行同步代码块的过程中，遇到异常而导致线程终止，所也会被释放 

* 在执行同步代码块的过程中，执行了锁对象所属对象的wait()方法， 这个线程会释放对象锁，而此线程对象会进入线程等待池中，等待被唤醒。

####**4.是重入锁**

（1）可重入锁的概念：自己可以再次获取自己内部的锁。比如有一条线程获得某个对象的锁，此时这个对象锁还没有释放，当其再次获取这个对象的锁的时候还是可以获取的，如果不可重入锁的话，就会造成死锁。

（2）synchronized拥有所重入的功能，也就是在使用synchronized时，当一个线程得到一个线程锁后，再次请求此对象锁时是可以再次得到该对象的锁的，这也证明在synchronized方法/块的内部调用本类的其他synchronized方法/块时，是永远可以得到锁的。

####**5.使用锁的规则**

* 当两个并发线程访问同一个对象object中的这个synchronized(this)同步代码块时，一个时间内只能有一个线程得到执行。另一个线程必须等待当前线程执行完这个代码块以后才能执行该代码块。

* 当一个线程访问object的一个synchronized(this)同步代码块时，另一个线程仍然可以访问该object中的非synchronized(this)同步代码块。

* 当一个线程访问object的一个synchronized(this)同步代码块时，其他线程对
object中所有其它synchronized(this)同步代码块的访问将被阻塞。

* 当一个线程访问object的一个synchronized(this)同步代码块时，它就获得了这个object的对象锁。结果，其它线程对该object对象所有同步代码部分的访问都被暂时阻塞。

* 以上规则对其它对象锁同样适用.

####**6.源码**

synchronized代码块通过javap生成的字节码中包含 monitorenter 和 monitorexit 指令。

###**Synchronized线程八锁**

例

```
public class Thread8MonitorTest {
    public static void main(String[] args) {
        Number number = new Number();
        Number number2 = new Number();

        new Thread(new Runnable() {
            @Override
            public void run() {
                number.getOne();
            }
        }).start();

        new Thread(new Runnable() {
            @Override
            public void run() {
//                number.getTwo();
                number2.getTwo();
            }
        }).start();

    }
}

class Number{

    public static synchronized void getOne(){//Number.class
        try {
            Thread.sleep(3000);
        } catch (InterruptedException e) {
        }

        System.out.println("one");
    }

    public static synchronized void getTwo(){//this
        System.out.println("two");
    }

//    public void getThree(){
//        System.out.println("three");
//    }

}

```

 针对不同情况判断打印的 是"one" 还是"two" ，顺序如何？
 
 1.两个普通同步方法，两个线程，标准打印，

输出结果： one  two 也有可能是 two one （概率小）

分析：存在线程竞争，因为One线程先开启，所以One先输出概率大。

2.给 getOne()方法里面新增 Thread.sleep()方法  

输出结果：one  two  

分析：因为one线程先开始后调用getone方法，睡眠ing，gettwo方法等待，若先开gettwo线程，则先打印two。

3.新增普通方法 getThree() , 

输出结果：three  one   two  

分析：因为one和two方法上锁，会阻塞，所以three先运行，若先开gettwo线程，则打印顺序为two three one

4.两个普通同步方法，两个 Number 对象，

输出结果：two  one  

分析：因为非静态方法的锁默认为this，有两个number对象，所以one不会阻塞two,两个线程独立运行。

 5.修改 getOne() 为静态同步方法，一个 Number 对象，

输出结果：two   one      

分析：因为是一个对象：非静态方法的锁默认为  this(哈希值1831557028),  静态方法的锁为 对应的 Class 实例(哈希值1676457073)

 6.修改两个方法均为静态同步方法，一个 Number 对象?  

输出结果：one   two   

分析：因为两个锁对应同一个Class实例（number）,two会阻塞，one先执行，若先开gettwo线程，则打印顺序为two one。

7.一个静态同步方法，一个非静态同步方法，两个 Number 对象? 

输出结果：two  one 

分析：因为是两个对象：两个锁分别对应两个对象，所以互不影响

8.两个静态同步方法，两个 Number 对象? 

输出结果：one  two   

分析：因为都是静态同步方法，两个锁对应是Class实例，虽然是两个对象，但都是Number类，two会阻塞,
 
**小结：**

* 非静态方法的锁默认为  this,  静态方法的锁为 对应的 Class 实例
* 某一个时刻内，只能有一个线程持有锁，无论几个方法。

<br>
<br>
本人才疏学浅，若有错误，还请指出 
谢谢！