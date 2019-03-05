####**1.常用命令有哪些**

* 文件相关命令
 * head -n、tail、ln、locate、wc、more、less、ll、dd、df、du、tar-zxvf、ls、nl、cat、pwd、mkdir、rm 、cd、cp、touch、	awk、
 * date由localtime函数将时间数值转换为日期，是非线程安全的（静态变量）。 而localtime_r和strftime是线程安全的。
 * find -name、-user、-mtime -n、-size -n
 * ln：硬链接：以文件副本的形式存在，但同步更新，不占用实际空间，不允许给目录创建硬链接，只有在同一个文件系统中才能创建，共享i节点，i节点引用计数递增，删除任意一个文件不会对其他文件造成影响，因为只有引用计数为零时才真正删除文件。
 * ln-s：软连接，以路径的形式存在。可跨文件系统 ，可以对一个不存在的文件名进行链接、可以对目录进行链接


* 权限相关命令
chmod、chgrp、chown、sudo、useradd。

* 网络相关命令
 * iptables、tcpdump、wget -b、netstat-an、 ss-ln、ping-c、ifconfig、telnet、apt-get
 * tcpdump -i	指定监听的网络接口，例tcpdump -nvvv -i any -c 3 port 22 and port 8080：只输出端口号是 22 和 8080 的数据包

* 进程相关命令
ps-ef(aux)、kill-9、lsof、strace、free-m、sysctl-w、nice-n	、renice-p、vmstat 3、ulimit -a -n -s、 top @duck1

* 计划任务命令
at、crontab、2>

* vim常用命令
 * 命令模式：ZZ、dd、D、yy、p、gg、G、
 * 进入插入模式：a、i、o、r.
 * 编辑模式：:setnu 、:setnonu、:/、:?、:5,10s/cat/dog/g

####**2.Linux的五种IO模型**

* BIO：能够及时返回数据，无延迟，性能低。
* NIO：能够在等待任务完成的过程中处理其他事件，需要不断询问数据是否准备好了，但拷贝数据的整个过程，进程仍然是阻塞的，延迟会增加
* 多路复用IO：有数据可读或可写（就绪事件），就通知用户进程
* 信号驱动IO：当数据准备完成之后，会主动的通知用户进程
* AIO：当用户进程发起系统调用后，立刻就可以开始去做其它的事情，读写操作由内核完成，完成后内核将数据放到指定的缓冲区，通知应用程序来取

####**3.IO多路复用select/poll/epoll？**

* select
 * 参数nfds：被监听的文件描述符的总数；readfds：可读；writefds：可写；exceptfds：异常；timeout：超时时间（s、us），
 * fd_set结构体仅包含一个整型数组，能容纳的文件描述符数量由FD_SETSIZE指定，所以select文件描述符的数量存在最大限制
 * 对socket进行线性扫描，即采用轮询的方法，O(n)，用户空间和内核空间在传递fd时复制开销大
 * 若timeout调用失败，它的值是不确定的，所以说不能完全信任该值

* pselect
新增sigmask参数，为null时和select相同，timeout参数具有精确（s、ns）、不变的特性 

* poll
 * 参数struct pollfd *fds表示fd、events、revents；nfds：指定被监听事件集合fds的大小；timeout：指定poll的超时值（ms）
 * 没有最大连接数的限制，因为它是基于链表来存储的，
 * 大量的fd的数组被整体复制于用户态和内核地址空间之间，开销大，轮询方式，O(n)

* dev / poll是Solaris的推荐轮询替换
打开后，轮询进程必须先初始化一个pollfd结构，可以预先设置感兴趣描述符的列表，然后循环等待事件发生，而select和poll需要每次轮询所有fd，它是级别触发的，比poll快。

* epoll
 * 三个函数epoll_create(size)；epoll_ctl(epfd, op, fd, event)；epoll_wait(epfd, events, maxevents, timeout);
 * 没有最大并发连接的限制，1G的内存上能监听约10万个端口
 * 回调机制：活跃的FD才会调用callback函数
 * 内存拷贝：利用一个文件描述符管理多个描述符，将用户关系的文件描述符的事件存放到内核的一个事件表中，在用户空间和内核空间的copy只需一次。即epoll使用mmap（共享内存）减少复制开销，
 * LT（默认）：fd可读或可写就通知应用程序，若不做任何操作，下次还会再通知，支持阻塞和非阻塞接口。
 * ET：fd有可读或可写事件通知应用程序，只通知一次，必须使用非阻塞接口，以避免文件描述符的饥饿情况发生。
 * epoll适用于同时需要保持大量长连接，而且连接的关闭很频繁时，但其中只有少数的连接是活跃的。select/poll适用于连接数少并且连接都十分活跃，

* Kqueue专用于FreeBSD系统，只能用于UFS文件系统
 * 函数 kqueue()类似于epoll_create()。而kevent函数集成了epoll_ctl()和epoll_wait() 的角色，kevent可以一次系统调用多次更新兴趣集。而epoll不能在单次系统调用中多次更新兴趣集，kevent结构体支持多种非文件事件，而epoll只能基于文件描述符工作
 * Kqueue支持磁盘文件，而epoll并不支持所有的文件描述符，它是基于准备就绪模型的

* IOCP：专用于Windows
支持asynchronous I/O的系统，是Proactor模式，但由于局限性，大型服务器部署在UNIX。

####**4.看过epoll的源码吗？**

* epoll_create()函数的size参数没用，只是为了兼容（之前是hash表），该函数由宏定义（系统限制了参数个数和传参方式），它的主要功能是通过sys_epoll_create1()函数来完成的

* epoll_create1()函数通过ep_alloc生成一个eventpoll对象，并初始化eventpoll的三个等待队列，wait，poll_wait以及rdlist 。同时还会初始化被监视fs的rbtree 根节点。

* evet_init()函数会做一些系统化的工作，创建文件信息等。

* epoll_ctl()函数：通过epoll_create生成一个eventpoll后，可以通过epoll_ctl提供的相关操作（ep_find），根据op的类型对eventpoll进行ADD（ep_insert），MOD（ep_modify），DEL（ep_remove）操作。

* ep_insert()函数：从slub中分配一个epitem的对象。并初始化epitem的三个list头指针，指向传入的eventpoll，将epitem插入目标文件的polllist并注册回调函数；将epitem插入eventpoll的rbtree。

* Poll()函数是系统函数，一般由设备驱动提供。

* poll_wait()函数调用 ep_ptable_queue_proc()函数

* ep_ptable_queue_proc()：使用等待队列实现epitem和callback函数的关联。然后通过目标文件的poll函数调用callback函数。

* ep_poll_callback()函数：当fd状态改变时（感兴趣事件），调用该函数，首先会把文件描述符epitem 放到eventpoll 的rdllist中，然后调用wake_up函数唤醒epitem上wq的进程。这样就可以返回到epoll_wait的调用者，将他唤醒。

* epoll_wait()函数：首先会检测传入参数的合法性，epfd是否合法等。然后通过file->private_data获取eventpoll。获取epoll后调用ep_poll函数完成真正的epoll_wait工作。

* ep_poll()函数：首先根据timeout的值判断是否是无限等待，然后判断eventpoll的rdlist是否为空，如果为空，那么将current进程通过一个waitquene entry加入eventpoll的waitlist（wq）。如果rdlist非空，那么通过ep_send_events将event转发到用户空间。

* ep_send_events函数调用ep_scan_ready_list对ready_list进行扫描

* ep_scan_ready_list对ready_list进行扫描。将所有的epitem都转移到了txlist上, 而rdllist被清空了。然后调用ep_send_events_proc()函数。

* ep_send_events_proc扫描rdlist从头上面拿出epitem，然后调用ep_eventpoll_poll函数，完成转发。 
先把就绪事件链表转移到中间链表,然后挨个遍历拷贝到用户空间,并且判断是否为水平触发,是的话再次插入到就绪链表。

* LT与ET的区别：通过ep_send_events_proc()函数实现，用if判断如果没有标上EPOLLET(默认的LT)且“事件被关注”的fd就会被重新放回了rdllist。那么下次epoll_wait当然会又把rdllist里的fd拿来拷给用户了。

####**5.进程的调度算法？**

* 进程分类
 * 进程区分为实时进程：不会被低优先级的进程阻塞. 响应时间尽可能短；交互式进程：当接受了用户的输入后, 进程必须很快被唤醒, 否则用户会感觉系统反应迟钝；批处理进程：经常在后台运行。
 * 按优先级分，时间片：CPU分配给各个程序的时间，每个线程被分配一个时间段，

* 调度策略
 * 先来先服务：相同优先级的任务先到先服务，高优先级的任务可以抢占低优先级的任务
 * 短作业优先：从后备队列中选择一个或若干个估计运行时间最短的作业，将它们调入内存运行
 * 优先级调度：具有最高优先权的进程会被分配到CPU，用老化解决饥饿问题。
 * 时间片轮转：相同优先级的任务当用完时间片会被放到队列尾部，以保证公平性，高优先级可以抢占低优先级的任务
 * 多级队列调度：分为多个独立队列，交互式（轮转调度）和批处理（FCFS），每个队列都有自己的调度算法。
 * 多级反馈队列调度：允许进程在队列之间移动，以此实现老化

####**6.进程间通信方式？**

* 管道：一种半双工的通信方式，数据只能单向流动
 * 无名管道：只能在具有亲缘关系的进程间使用。即父子进程关系。
 * 有名管道： 允许无亲缘关系进程间的通信。

* 信号： 信号是一种比较复杂的通信方式，用于通知接收进程某个事件已经发生。

* 信号量： 信号量是一个计数器，可以用来控制多个进程对共享资源的访问。常作为一种锁机制,防止某进程在访问资源时其它进程也访问该资源。

* 消息队列 ： 消息队列是由消息的链表，存放在内核中并由消息队列标识符标识。消息队列克服了信号传递信息少、管道只能承载无格式字节流以及缓冲区大小受限等缺点。

* 共享内存：映射一段能被其他进程所访问的内存，这段共享内存由一个进程创建，但多个进程都可以访问。共享内存是最快的 IPC 方式，

* 套接字： 它可用于不同机器间的进程通信。

<br>
<br>
本人才疏学浅，若有错，请指出，谢谢！ 
如果你有更好的建议，可以留言我们一起讨论，共同进步！ 
衷心的感谢您能耐心的读完本篇博文！
