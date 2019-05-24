###**线程安全**

**1.定义**

当多个线程访问一个对象时，如果不用考虑这些线程在运行时环境下的调度和交替行，也不需要进行额外的同步，或者在调用方进行任何其他的协调操作，调用这个对象的行为都可以获得正确的结果

**2.分类**

（1）不可变

不可变的对象一定是线程安全的，只要一个不可变对象被正确地构建出来（没有发生this引用逃逸情况），那其外部的可见状态永远也不会改变 ，永远也不会看到它在多个线程之中处于不一致的状态。如Integer类，内部状态变量value定义为final来保障状态不变。

例1： JDK 中 Integer 类的构造函数
```
   private final int value;  
  
  /** 
   * Constructs a newly allocated {@code Integer} object that 
   * represents the specified {@code int} value. 
   * 
   * @param   value   the value to be represented by the 
   *                  {@code Integer} object. 
   */  
  public Integer(int value) {  
      this.value = value;  
  }  

```

 

（2）绝对线程安全

要达到不管运行的环境如何，调用者都不需要任何额外的同步措施。在Java API中标注自己是线程安全的类，大多数都不是绝对线程安全的。

 
例2：加入同步以保证 Vector 访问的线程安全性
```
Thread removeThread = new Thread(new Runnable() {  
    @Override  
    public void run() {  
        synchronized(vector) {  
            for (int i = 0; i < vector.size(); i++) {  
                vector.remove(i);  
            }  
        }  
    }  
});  
  
Thread printThread = new Thread(new Runnable() {  
    @Override  
    public void run() {  
        synchronized(vector) {  
            for (int i = 0; i < vector.size(); i++) {  
                System.out.println(vector.get(i));  
            }  
        }  
    }  
});
```

（3）相对线程安全

通常意义上的线程安全，需要保证对这个对象的单独操作是线程安全的，在调用的时候不需要做额外的保障措施，但对于一些特定顺序的连续调用，就可能需要在调用端使用额外的同步手段来保证调用的正确性。如Vector、HashTable、Collections的synchronizedCollection（）方法包装的集合。


（4）线程兼容

指对象本身不是线程安全的，但是可以通过调用端正确地使用同步手段来保证对象在并发环境中可以安全地使用，Java API大部分类属于线程兼容，如ArrayList、HashMap。

 

（5）线程对立

无论调用端是否采取了同步措施，都无法在多线程环境下并发使用的代码。有害的，应该尽量避免，如Thread类的suspend（）和resume（）方法，如果两个线程同时持有一个线程对象，一个尝试去中断线程，一个尝试去恢复线程，并且并发进行，无论调用时是否进行了同步，目标线程都存在死锁风险，所以这两个方法已经被声明废弃。

###**锁优化**

**1.定义**

在阻塞式的情况下，如何让性能不要变得太差

**2.锁的分类**

锁分为：轻量级锁、自旋锁、偏向锁、

* 对象头：描述对象的hash、锁信息、垃圾回收标记。指向锁记录的指针、指向monitor的指针、GC标记、偏向锁线程ID

（1）偏向锁：锁会偏向于当前已经占有锁的线程

* 将对象头Mark的标记设置为偏向，并将线程ＩＤ写入对象头Mark

* 只要没有竞争，获取偏向锁的线程，在将来进入同步块时，不需要同步

* 当其他线程请求相同的锁时，偏向模式结束

* -XX:+UseBiasedLocking  默认启用

* 在竞争激烈的场合，偏向锁会增加系统的负担

（2）轻量级锁

*  在代码进入同步块的时候，如果此同步对象没有被锁定，虚拟机首先将在当前线程的栈帧中建立一个名为锁记录的空间，用于存储锁对象目前的Mark Word的拷贝，称为Displaced Mark Word，

* 然后使用CAS操作尝试将对象的Mark Word更新为指向Lock Record的指针，如果更新动作成功，那么线程就拥有了该对象的锁，并且对象Mark Word的锁标志位转变成“00”，表示处于轻量级锁定状态。

* 如果更新动作失败，先检查对象的Mark Word是否指向当前线程的栈帧，如果是，就可以直接进入同步块继续执行，否则说明这个锁对象被其他线程抢占了，如果有两条以上线程争用同一个锁，那轻量级锁就不再有效，要膨胀为重量级锁，锁标志的状态值转变为“10”。

* 解锁过程也是通过CAS操作进行，如果对象的Mark Word仍然指向线程的锁记录，就用CAS操作把对象当前的Mark Word和线程中复制的Displaced Mark Word替换回来，

* 如果替换成功，整个同步过程就完成了，如果替换失败，说明有其他线程尝试过获取锁，那就要在释放锁的同时，唤醒被挂起的线程。

* 轻量级锁使用CAS操作避免使用互斥量的开销，如果存在锁竞争，除了互斥量的开销，还额外发生了CAS操作。

（3）自旋锁

* 若线程可以很快获得锁，不用在OS挂起锁，让线程自旋。

* 共享数据的锁定状态可能只持续很短的一段时间，为了这段时间去挂起和恢复线程不值得。如果物理机器有一个以上的处理器，能让两个或以上的线程同时并行执行，可以让后面请求锁的线程“等待一段时间”，但不放弃处理器的执行时间，看看持有锁的线程是否很快释放锁，为了让线程等待，让线程执行一个忙循环（自旋），这项技术就是自旋锁。

* JDK 1.6引入自适应自旋，自旋时间不固定，由前一次在同一个锁上的自旋时间及锁的拥有者的状态来决定。如果在同一个锁对象上，自旋等待刚刚成功获得过锁，并且持有锁的线程正在运行，那么虚拟机会认为这次自旋也很可能再次成功，进而允许自旋等待持续相对更长的时间，另外，如果对于某个锁，自旋很少成功，那么在以后获取这个锁时将可能省略掉自旋过程，以避免浪费处理器资源。

（4）内置于JVM中获取锁的步骤

① 偏向锁可用会尝试偏向锁

② 轻量级锁可用会尝试轻量级锁

③ 若①②失败，尝试自旋锁

④ 若①②③都失败，尝试普通锁

图解：

![这里写图片描述](http://img.blog.csdn.net/20170626110700578?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYmFpeWVfeGluZw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

**2.锁优化的方法**

（1）减少锁的持有时间（方法锁改为锁对象）

例1

优化前：在进入方法前就要得到锁，其他线程就要在外面等待

```
public synchronized void syncMethod(){  
		othercode1();  
		mutextMethod();  
		othercode2(); 
	}
```
优化后：减少其他线程等待的时间

```
public void syncMethod(){  
		othercode1();  
		synchronized(this)
		{
			mutextMethod();  
		}
		othercode2(); 
	}
```


（2）减少锁粒度：将大对象（这个对象可能会被很多线程访问），拆成小对象，大大增加并行度，降低锁竞争

例：concurrentHashMap的分段锁

（3）锁分离：读写锁

例：LinkedBlockingQueue  

![这里写图片描述](http://img.blog.csdn.net/20170626110441067?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYmFpeWVfeGluZw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

从头部取出，从尾部放数据

（4）锁粗化：使用完公共资源后，立即释放锁

如果一系列的连续操作都对同一个对象反复加锁和解锁，甚至加锁出现在循环体中，那即使没有线程竞争，频繁地进行互斥同步操作也会导致不必要的性能损耗。如果虚拟机检测到有一串零碎的操作都对同一个对象加锁，将会把加锁同步的范围扩展到整个操作序列的外部。

例2：

优化前：

```
public void demoMethod(){  
		synchronized(lock){   
			//do sth.  
		}  
		//做其他不需要的同步的工作，但能很快执行完毕  
		synchronized(lock){   
			//do sth.  
		} 
	}
```

优化后：

```
public void demoMethod(){  
		//整合成一次锁请求 
		synchronized(lock){   
			//do sth.   
			//做其他不需要的同步的工作，但能很快执行完毕  
		}
	}
```

（5）锁清除：

指在虚拟机即时编译器运行时，对一些代码上要求同步，但被检测到不可能存在共享数据竞争的锁进行消除，判定依据来源于逃逸分析的数据支持，如果一段代码，堆上的所有数据都不会逃逸出去被其他线程访问到，那就可以把它们当做栈上数据对待，认为是线程私有的，同步加锁无需进行。

（6）无锁：无锁的实现方式（CAS）

<br>
<br>
本人才疏学浅，若有错，请指出 
谢谢！