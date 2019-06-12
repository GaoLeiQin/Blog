####**引入**

我在上一篇博文中讲了[select、poll、epoll的区别](http://blog.csdn.net/baiye_xing/article/details/74353066)，接下来我再介绍几个比较“冷门”的IO复用函数。

####**1.select与pseletct的区别**

（1）函数

```
int select(int n, fd_set *readfds, fd_set *writefds, fd_set *exceptfds, struct timeval *timeout);

int pselect(int n,fd_set *readfds,fd_set *writefds,fd_set *exceptfds,const struct timespec *timeout, const sigset_t *sigmask);

```

（2）相同点

* Select和pselect都是等待一系列的文件描述符（int）的状态发生变化。

* pselect函数是一个 防止信号干扰的增强型 select函数

（3）不同点

* timeout参数的精确性
 select函数用的timeout参数，是一个timeval的结构体（包含秒和微秒），然而pselect用的是一个timespec结构体（包含秒和纳秒）

* timeout参数的不变性
select函数可能会为了指示还剩多长时间而更新timeout参数，然而pselect不会改变timeout参数

* 新增的第六个sigmask参数
select函数没有sigmask参数，当pselect的sigmask参数为null时，两者行为时一致的。有sigmask的时候，pselect相当于如下的select()函数，在进入select()函数之前手动将信号的掩码改变，并保存之前的掩码值；select()函数执行之后，再恢复为之前的信号掩码值。

####**2.Kqueue概述**

（1）定义

* Kqueue专用于FreeBSD系统，只能用于UFS文件系统

* kqueue类似epoll，支持每个进程中有多个上下文(兴趣集)。

* kqueue设计的目的并非是为了替代基于套接字事件复用技术的select()/poll(),而是提供一般化的机制来处理多种操作系统事件。

* kqueue API由两个函数调用（ kqueue()与kevent()）和一个辅助设置事件的宏组成。

（2） kqueue()函数：类似于epoll_create()。

```
int kqueue(void);
```

kqueue函数启动一个新的kqueue。如果调用成功，返回值将是一个用来和新创建的kqueue交互的描述符。每个kqueue都有一个与之关联的唯一的描述符。因此，一个程序可以同时打开多个kqueue。kqueue描述符的行为和常规文件描述符类似：它们也可以被复用。


（3）kevent()函数集成了epoll_ctl()(用于调整兴趣集)和epoll_wait()(获取事件) 的角色。

```
int kevent(int kq, const struct kevent *changelist, int nchanges, 
           struct kevent *eventlist, int nevents, const struct timespec *timeout);
```

（4）结构体kevent 

```
struct kevent {
     uintptr_t       ident;          /* 事件标识 */
     int16_t         filter;         /* 事件过滤器 */
     uint16_t        flags;          /* 通用标记 */
     uint32_t        fflags;         /* 特定过滤器标记 */
     intptr_t        data;           /* 特定过滤器数据 */
     void            *udata;         /* 不透明的用户数据标识 */
 };
```

（5）Kqueue的优点

* kevent调用中可以指定进行多次兴趣集更新。而epoll不能在单次系统调用中多次更新兴趣集。
如：当你的兴趣集中有100个文件描述符需要更新状态时，你不得不调用100次epoll_ctl()函数。

* kevent结构体支持多种非文件事件，而epoll的设计目的是为了提高select()/poll()的性能，epoll只能基于文件描述符工作，
虽然说Linux系统一切皆文件，但有好多事物不是文件，例如时钟，信号，信号量，进程、网络设备等都不是文件。

* Kqueue支持磁盘文件，而epoll并不支持所有的文件描述符，它是基于准备就绪模型的。监视的是准备就绪的套接字，因此套接字上的顺序IO调用不会发生阻塞。但是磁盘文件并不符合这种模型，因为它们总是处于就绪状态。

####**3.dev/poll的介绍**

（1）定义

/ dev / poll是Solaris的推荐轮询替换。

（2）原理

* 打开/dev/poll之后，轮询进程必须先初始化一个pollfd结构（即poll函数使用的结构，不过本机制不使用其中的revents成员）数组，再调用write往/dev/poll设备上写这个结构数组以把它传递给内核，然后执行DP_POLL命令阻塞自身以等待事件发生。

* 传递给ioctl调用的结构如下：

```
struct dvpoll {
    struct pollfd     *dp_fds;
    int                   dp_nfds;
    int                   dp_timeout;
};
```

分析：

* dp_fds成员指向一个缓冲区，供ioctl在返回时存放一个pollfd结构数组。

* dp_nfds成员指定该缓冲区。ioctl调用将一直阻塞到任何一个被轮询描述符上发生所关心的事件，或者流逝时间超过经由dp_timeout成员指定的毫秒数为止。

* dp_timeout指定为0将导致ioctl立即返回，从而提供了使用本接口的非阻塞手段。dp_timeout指定为-1表示没有超时设置。

（3）特点

* 使用/ dev / poll可以获得/ dev / poll的打开句柄，并通过写入该句柄来告诉操作系统一次您感兴趣的文件; 从那时起，您只需从该句柄中读取当前可用的文件描述符集。

* / dev / poll通常使用相同的参数来调用poll（）。

* / dev / poll是级别触发的

（4）与poll的区别

* Solaris的/dev/poll中的句柄提供了一个可扩展的轮询大量描述符的方法。轮询进程可以预先设置好待查询描述符的列表，然后进入一个循环等待事件发生，每次循环回来时不必再次设置该列表。而select和poll需要每次轮询fd。

* 在所有涉及套接字的情况下，/ dev / poll比poll（）快。

注：不建议在Linux上使用/ dev / poll

####**4.IOCP的介绍**

* Windows的IOCP非常出色，目前很少有支持asynchronous I/O的系统，但是由于其系统本身的局限性，大型服务器还是在UNIX下。

* kqueue/epoll 与 IOCP相比，就是多了一层从内核copy数据到应用层的阻塞，从而不能算作asynchronous I/O类。

* IOCP是Proactor模式



<br>
<br>
本人才疏学浅，若有错，请指出，谢谢！ 
如果你有更好的建议，可以留言我们一起讨论，共同进步！ 
衷心的感谢您能耐心的读完本篇博文！

> 参考链接：
> 1.[reeBSD 的kqueue](http://blog.csdn.net/robertzhouxh/article/details/8281879)
> 2.[pselect 和 select的比较](http://www.cnblogs.com/diegodu/p/3988103.html)
3.[可扩展的事件复用技术：epoll和kqueue](http://blog.csdn.net/stormbjm/article/details/49469191)
4.[再谈select, iocp, epoll,kqueue及各种I/O复用机制](http://blog.csdn.net/shallwake/article/details/5265287)
5.[Microbenchmark comparing poll, kqueue, and /dev/poll](http://www.kegel.com/dkftpbench/Poller_bench.html)