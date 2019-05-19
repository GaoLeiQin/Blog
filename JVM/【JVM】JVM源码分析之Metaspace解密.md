###**概述**

metaspace，顾名思义，元数据空间，专门用来存元数据的，它是jdk8里特有的数据结构用来替代perm，这块空间很有自己的特点，前段时间公司这块的问题太多了，主要是因为升级了中间件所致，看到大家讨论来讨论去，看得出很多人对metaspace还是模棱两可，不是很了解它，因此我觉得有必要写篇文章来介绍一下它，解开它神秘的面纱，当我们再次碰到它的相关问题的时候不会再感到束手无策。

###**为什么会有metaspace**

metaspace的由来民间已有很多传说，不过我这里只谈我自己的理解，因为我不是oracle参与这块的开发者，所以对其真正的由来不怎么了解。

我们都知道jdk8之前有perm这一整块内存来存klass等信息，我们的参数里也必不可少地会配置-XX:PermSize以及-XX:MaxPermSize来控制这块内存的大小，jvm在启动的时候会根据这些配置来分配一块连续的内存块，但是随着动态类加载的情况越来越多，这块内存我们变得不太可控，到底设置多大合适是每个开发者要考虑的问题，如果设置太小了，系统运行过程中就容易出现内存溢出，设置大了又总感觉浪费，尽管不会实质分配这么大的物理内存。基于这么一个可能的原因，于是metaspace出现了，希望内存的管理不再受到限制，也不要怎么关注元数据这块的OOM问题，虽然到目前来看，也并没有完美地解决这个问题。

或许从JVM代码里也能看出一些端倪来，比如MaxMetaspaceSize默认值很大，CompressedClassSpaceSize默认也有1G，从这些参数我们能猜到metaspace的作者不希望出现它相关的OOM问题。

###**metaspace的组成**

metaspace其实由两大部分组成

* Klass Metaspace
* NoKlass Metaspace

Klass Metaspace就是用来存klass的，klass是我们熟知的class文件在jvm里的运行时数据结构，不过有点要提的是我们看到的类似A.class其实是存在heap里的，是java.lang.Class的一个对象实例。这块内存是紧接着Heap的，和我们之前的perm一样，这块内存大小可通过-XX:CompressedClassSpaceSize参数来控制，这个参数前面提到了默认是1G，但是这块内存也可以没有，假如没有开启压缩指针就不会有这块内存，这种情况下klass都会存在NoKlass Metaspace里，另外如果我们把-Xmx设置大于32G的话，其实也是没有这块内存的，因为会这么大内存会关闭压缩指针开关。还有就是这块内存最多只会存在一块。

NoKlass Metaspace专门来存klass相关的其他的内容，比如method，constantPool等，这块内存是由多块内存组合起来的，所以可以认为是不连续的内存块组成的。这块内存是必须的，虽然叫做NoKlass Metaspace，但是也其实可以存klass的内容，上面已经提到了对应场景。

Klass Metaspace和NoKlass Mestaspace都是所有classloader共享的，所以类加载器们要分配内存，但是每个类加载器都有一个SpaceManager，来管理属于这个类加载的内存小块。如果Klass Metaspace用完了，那就会OOM了，不过一般情况下不会，NoKlass Mestaspace是由一块块内存慢慢组合起来的，在没有达到限制条件的情况下，会不断加长这条链，让它可以持续工作。

###**metaspace的几个参数**

如果我们要改变metaspace的一些行为，我们一般会对其相关的一些参数做调整，因为metaspace的参数本身不是很多，所以我这里将涉及到的所有参数都做一个介绍，也许好些参数大家都是有误解的

* UseLargePagesInMetaspace
* InitialBootClassLoaderMetaspaceSize
* MetaspaceSize
* MaxMetaspaceSize
* CompressedClassSpaceSize
* MinMetaspaceExpansion
* MaxMetaspaceExpansion
* MinMetaspaceFreeRatio

**UseLargePagesInMetaspace**

默认false，这个参数是说是否在metaspace里使用LargePage，一般情况下我们使用4KB的page size，这个参数依赖于UseLargePages这个参数开启，不过这个参数我们一般不开。

**InitialBootClassLoaderMetaspaceSize**

64位下默认4M，32位下默认2200K，metasapce前面已经提到主要分了两大块，Klass Metaspace以及NoKlass Metaspace，而NoKlass Metaspace是由一块块内存组合起来的，这个参数决定了NoKlass Metaspace的第一个内存Block的大小，即2*InitialBootClassLoaderMetaspaceSize，同时为bootstrapClassLoader的第一块内存chunk分配了InitialBootClassLoaderMetaspaceSize的大小

**MetaspaceSize**

默认20.8M左右(x86下开启c2模式)，主要是控制metaspaceGC发生的初始阈值，也是最小阈值，但是触发metaspaceGC的阈值是不断变化的，与之对比的主要是指Klass Metaspace与NoKlass Metaspace两块committed的内存和。

**MaxMetaspaceSize**

默认基本是无穷大，但是我还是建议大家设置这个参数，因为很可能会因为没有限制而导致metaspace被无止境使用(一般是内存泄漏)而被OS Kill。这个参数会限制metaspace(包括了Klass Metaspace以及NoKlass Metaspace)被committed的内存大小，会保证committed的内存不会超过这个值，一旦超过就会触发GC，这里要注意和MaxPermSize的区别，MaxMetaspaceSize并不会在jvm启动的时候分配一块这么大的内存出来，而MaxPermSize是会分配一块这么大的内存的。

**CompressedClassSpaceSize**

默认1G，这个参数主要是设置Klass Metaspace的大小，不过这个参数设置了也不一定起作用，前提是能开启压缩指针，假如-Xmx超过了32G，压缩指针是开启不来的。如果有Klass Metaspace，那这块内存是和Heap连着的。

**MinMetaspaceExpansion**

MinMetaspaceExpansion和MaxMetaspaceExpansion这两个参数或许和大家认识的并不一样，也许很多人会认为这两个参数不就是内存不够的时候，然后扩容的最小大小吗？其实不然

这两个参数和扩容其实并没有直接的关系，也就是并不是为了增大committed的内存，而是为了增大触发metaspace GC的阈值

这两个参数主要是在比较特殊的场景下救急使用，比如gcLocker或者should_concurrent_collect的一些场景，因为这些场景下接下来会做一次GC，相信在接下来的GC中可能会释放一些metaspace的内存，于是先临时扩大下metaspace触发GC的阈值，而有些内存分配失败其实正好是因为这个阈值触顶导致的，于是可以通过增大阈值暂时绕过去

默认332.8K，增大触发metaspace GC阈值的最小要求。假如我们要救急分配的内存很小，没有达到MinMetaspaceExpansion，但是我们会将这次触发metaspace GC的阈值提升MinMetaspaceExpansion，之所以要大于这次要分配的内存大小主要是为了防止别的线程也有类似的请求而频繁触发相关的操作，不过如果要分配的内存超过了MaxMetaspaceExpansion，那MinMetaspaceExpansion将会是要分配的内存大小基础上的一个增量

**MaxMetaspaceExpansion**

默认5.2M，增大触发metaspace GC阈值的最大要求。假如说我们要分配的内存超过了MinMetaspaceExpansion但是低于MaxMetaspaceExpansion，那增量是MaxMetaspaceExpansion，如果超过了MaxMetaspaceExpansion，那增量是MinMetaspaceExpansion加上要分配的内存大小

注：每次分配只会给对应的线程一次扩展触发metaspace GC阈值的机会，如果扩展了，但是还不能分配，那就只能等着做GC了

**MinMetaspaceFreeRatio**

MinMetaspaceFreeRatio和下面的MaxMetaspaceFreeRatio，主要是影响触发metaspaceGC的阈值

默认40，表示每次GC完之后，假设我们允许接下来metaspace可以继续被commit的内存占到了被commit之后总共committed的内存量的MinMetaspaceFreeRatio%，如果这个总共被committed的量比当前触发metaspaceGC的阈值要大，那么将尝试做扩容，也就是增大触发metaspaceGC的阈值，不过这个增量至少是MinMetaspaceExpansion才会做，不然不会增加这个阈值

这个参数主要是为了避免触发metaspaceGC的阈值和gc之后committed的内存的量比较接近，于是将这个阈值进行扩大

一般情况下在gc完之后，如果被committed的量还是比较大的时候，换个说法就是离触发metaspaceGC的阈值比较接近的时候，这个调整会比较明显

注：这里不用gc之后used的量来算，主要是担心可能出现committed的量超过了触发metaspaceGC的阈值，这种情况一旦发生会很危险，会不断做gc，这应该是jdk8在某个版本之后才修复的bug

**MaxMetaspaceFreeRatio**

默认70，这个参数和上面的参数基本是相反的，是为了避免触发metaspaceGC的阈值过大，而想对这个值进行缩小。这个参数在gc之后committed的内存比较小的时候并且离触发metaspaceGC的阈值比较远的时候，调整会比较明显

###**jstat里的metaspace字段**

我们看GC是否异常，除了通过GC日志来做分析之外，我们还可以通过jstat这样的工具展示的数据来分析，我们通过jstat可以看到metaspace相关的这么一些指标，分别是M，CCS，MC，MU，CCSC，CCSU，MCMN，MCMX，CCSMN，CCSMX

它们的定义如下：

```
 column {
    header "^M^"    /* Metaspace - Percent Used */
    data (1-((sun.gc.metaspace.capacity - sun.gc.metaspace.used)/sun.gc.metaspace.capacity)) * 100
    align right
    width 6
    scale raw
    format "0.00"
  }
  column {
    header "^CCS^"    /* Compressed Class Space - Percent Used */
    data (1-((sun.gc.compressedclassspace.capacity -
   sun.gc.compressedclassspace.used)/sun.gc.compressedclassspace.capacity)) * 100
    align right
    width 6
    scale raw
    format "0.00"
  }

  column {
    header "^MC^"    /* Metaspace Capacity - Current */
    data sun.gc.metaspace.capacity
    align center
    width 6
    scale K
    format "0.0"
  }
  column {
    header "^MU^"    /* Metaspae Used */
    data sun.gc.metaspace.used
    align center
    width 6
    scale K
    format "0.0"
  }
   column {
    header "^CCSC^"    /* Compressed Class Space Capacity - Current */
    data sun.gc.compressedclassspace.capacity
    width 8
    align right
    scale K
    format "0.0"
  }
  column {
    header "^CCSU^"    /* Compressed Class Space Used */
    data sun.gc.compressedclassspace.used
    width 8
    align right
    scale K
    format "0.0"
  }
  column {
    header "^MCMN^"    /* Metaspace Capacity - Minimum */
    data sun.gc.metaspace.minCapacity
    scale K
    align right
    width 8
    format "0.0"
  }
  column {
    header "^MCMX^"    /* Metaspace Capacity - Maximum */
    data sun.gc.metaspace.maxCapacity
    scale K
    align right
    width 8
    format "0.0"
  }
  column {
    header "^CCSMN^"    /* Compressed Class Space Capacity - Minimum */
    data sun.gc.compressedclassspace.minCapacity
    scale K
    align right
    width 8
    format "0.0"
  }
  column {
    header "^CCSMX^"    /* Compressed Class Space Capacity - Maximum */
    data sun.gc.compressedclassspace.maxCapacity
    scale K
    align right
    width 8
    format "0.0"
  }
```

我这里对这些字段分类介绍下

**MC & MU & CCSC & CCSU**

* MC表示Klass Metaspace以及NoKlass Metaspace两者总共committed的内存大小，单位是KB，虽然从上面的定义里我们看到了是capacity，但是实质上计算的时候并不是capacity，而是committed，这个是要注意的

* MU这个无可厚非，说的就是Klass Metaspace以及NoKlass Metaspace两者已经使用了的内存大小

* CCSC表示的是Klass Metaspace的已经被commit的内存大小，单位也是KB

* CCSU表示Klass Metaspace的已经被使用的内存大小

**M & CCS**

* M表示的是Klass Metaspace以及NoKlass Metaspace两者总共的使用率，其实可以根据上面的四个指标算出来，即(CCSU+MU)/(CCSC+MC)

* CCS表示的是NoKlass Metaspace的使用率，也就是CCSU/CCSC算出来的

PS：所以我们有时候看到M的值达到了90%以上，其实这个并不一定说明metaspace用了很多了，因为内存是慢慢commit的，所以我们的分母是慢慢变大的，不过当我们committed到一定量的时候就不会再增长了

**MCMN & MCMX & CCSMN & CCSMX**

* MCMN和CCSMN这两个值大家可以忽略，一直都是0

* MCMX表示Klass Metaspace以及NoKlass Metaspace两者总共的reserved的内存大小，比如默认情况下Klass Metaspace是通过CompressedClassSpaceSize这个参数来reserved 1G的内存，NoKlass Metaspace默认reserved的内存大小是2* InitialBootClassLoaderMetaspaceSize

* CCSMX表示Klass Metaspace reserved的内存大小

综上所述，其实看metaspace最主要的还是看MC，MU，CCSC，CCSU这几个具体的大小来判断metaspace到底用了多少更靠谱

本来还想写metaspace内存分配和GC的内容，不过那块说起来又是一个比较大的话题，因为那块大家看起来可能会比较枯燥，有机会再写


原文链接：[JVM源码分析之Metaspace解密](http://lovestblog.cn/blog/2016/10/29/metaspace/)