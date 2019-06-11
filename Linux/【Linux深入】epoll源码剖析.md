###**引入**

* 之前讲了[select、poll、epoll的区别](http://blog.csdn.net/baiye_xing/article/details/74353066)，由于许多应用中都用到了epoll，例如Netty、Redis等等，所以就来深入学习一下，现在我们就来剖析一下epoll的源码

* 我先来剖析理解epoll源码的基础：主要的数据结构，然后再来解析epoll主要的三个方法：epoll_create()、epoll_ctl()、epoll_wait()。

###**主要的数据结构**

####**1.eventpoll**

```
// epoll的核心实现对应于一个epoll描述符  
struct eventpoll {  
    spinlock_t lock;  
    struct mutex mtx;  
    wait_queue_head_t wq; // sys_epoll_wait() 等待在这里  
    // f_op->poll()  使用的, 被其他事件通知机制利用的wait_address  
    wait_queue_head_t poll_wait;  
    //已就绪的需要检查的epitem 列表 
    struct list_head rdllist;  
    //保存所有加入到当前epoll的文件对应的epitem  
    struct rb_root rbr;  
    // 当正在向用户空间复制数据时, 产生的可用文件  
    struct epitem *ovflist;  
    /* The user that created the eventpoll descriptor */  
    struct user_struct *user;  
    struct file *file;  
    //优化循环检查，避免循环检查中重复的遍历
    int visited;  
    struct list_head visited_list_link;  
}  
```

####**2.epitem**

```
// 对应于一个加入到epoll的文件  
struct epitem {  
    // 挂载到eventpoll 的红黑树节点  
    struct rb_node rbn;  
    // 挂载到eventpoll.rdllist 的节点  
    struct list_head rdllink;  
    // 连接到ovflist 的指针  
    struct epitem *next;  
    /* 文件描述符信息fd + file, 红黑树的key */  
    struct epoll_filefd ffd;  
    /* Number of active wait queue attached to poll operations */  
    int nwait;  
    // 当前文件的等待队列(eppoll_entry)列表  
    // 同一个文件上可能会监视多种事件,  
    // 这些事件可能属于不同的wait_queue中  
    // (取决于对应文件类型的实现),  
    // 所以需要使用链表  
    struct list_head pwqlist;  
    // 当前epitem 的所有者  
    struct eventpoll *ep;  
    /* List header used to link this item to the &quot;struct file&quot; items list */  
    struct list_head fllink;  
    /* epoll_ctl 传入的用户数据 */  
    struct epoll_event event;  
};  
```

####**3.eppoll_entry** 

```
// 与一个文件上的一个wait_queue_head 相关联，因为同一文件可能有多个等待的事件，
//这些事件可能使用不同的等待队列  
struct eppoll_entry {  
    // List struct epitem.pwqlist  
    struct list_head llink;  
    // 所有者  
    struct epitem *base;  
    // 添加到wait_queue 中的节点  
    wait_queue_t wait;  
    // 文件wait_queue 头  
    wait_queue_head_t *whead;  
}; 
```

###**epoll_create()**

####**1.epoll_create()**

```
//先进行判断size是否>=0，若是则直接调用epoll_create1
SYSCALL_DEFINE1(epoll_create, int, size)
{
        if (size <= 0)
                return -EINVAL;
        return sys_epoll_create1(0);
}
```

注：SYSCALL_DEFINE1是一个宏，用于定义有一个参数的系统调用函数，上述宏展开后即成为： int sys_epoll_create(int size)，这就是epoll_create系统调用的入口。至于为何要用宏而不是直接声明，主要是因为系统调用的参数个数、传参方式都有严格限制，最多六个参数, 

####**2.epoll_create1()**

```
/* 这才是真正的epoll_create啊~~ */
SYSCALL_DEFINE1(epoll_create1, int, flags)
{
    int error;
    struct eventpoll *ep = NULL;//主描述符
    /* Check the EPOLL_* constant for consistency.  */
    /* 这句没啥用处... */
    BUILD_BUG_ON(EPOLL_CLOEXEC != O_CLOEXEC);
    /* 对于epoll来讲, 目前唯一有效的FLAG就是CLOEXEC */
    if (flags & ~EPOLL_CLOEXEC)
        return -EINVAL;
    /*
     * Create the internal data structure ("struct eventpoll").
     */
    /* 分配一个struct eventpoll, 分配和初始化细节我们随后深聊~ */
    error = ep_alloc(&ep);
    if (error < 0)
        return error;
    /*
     * Creates all the items needed to setup an eventpoll file. That is,
     * a file structure and a free file descriptor.
     */
    /* 这里是创建一个匿名fd, 说起来就话长了...长话短说:
     * epollfd本身并不存在一个真正的文件与之对应, 所以内核需要创建一个
     * "虚拟"的文件, 并为之分配真正的struct file结构, 而且有真正的fd.
     * 这里2个参数比较关键:
     * eventpoll_fops, fops就是file operations, 就是当你对这个文件(这里是虚拟的)进行操作(比如读)时,
     * fops里面的函数指针指向真正的操作实现, 类似C++里面虚函数和子类的概念.
     * epoll只实现了poll和release(就是close)操作, 其它文件系统操作都有VFS全权处理了.
     * ep, ep就是struct epollevent, 它会作为一个私有数据保存在struct file的private指针里面.
     * 其实说白了, 就是为了能通过fd找到struct file, 通过struct file能找到eventpoll结构.
     * 如果懂一点Linux下字符设备驱动开发, 这里应该是很好理解的,
     * 推荐阅读 <Linux device driver 3rd>
     */
    error = anon_inode_getfd("[eventpoll]", &eventpoll_fops, ep,
                 O_RDWR | (flags & O_CLOEXEC));
    if (error < 0)
        ep_free(ep);
    return error;
}
```

####**3.eventpoll_init**

```
// epoll 文件系统的相关实现  
// epoll 文件系统初始化, 在系统启动时会调用  
  
static int __init eventpoll_init(void)  
{  
    struct sysinfo si;  
  
    si_meminfo(&si);  
    // 限制可添加到epoll的最多的描述符数量  
  
    max_user_watches = (((si.totalram - si.totalhigh) / 25) << PAGE_SHIFT) /  
                       EP_ITEM_COST;  
    BUG_ON(max_user_watches < 0);  
  
    // 初始化递归检查队列  
   ep_nested_calls_init(&poll_loop_ncalls);  
    ep_nested_calls_init(&poll_safewake_ncalls);  
    ep_nested_calls_init(&poll_readywalk_ncalls);  
    // epoll 使用的slab分配器分别用来分配epitem和eppoll_entry  
    epi_cache = kmem_cache_create("eventpoll_epi", sizeof(struct epitem),  
                                  0, SLAB_HWCACHE_ALIGN | SLAB_PANIC, NULL);  
    pwq_cache = kmem_cache_create("eventpoll_pwq",  
                                  sizeof(struct eppoll_entry), 0, SLAB_PANIC, NULL);  
  
    return 0;  
}  
```


###**epoll_ctl()**

####**1.epoll_crl()**

```
//创建好epollfd后, 接下来添加fd
//epoll_ctl的参数：epfd 表示epollfd；op 有ADD,MOD,DEL，
//fd 是需要监听的描述符，event 我们感兴趣的events
SYSCALL_DEFINE4(epoll_ctl, int, epfd, int, op, int, fd,  
                struct epoll_event __user *, event)  
{  
    int error;  
    int did_lock_epmutex = 0;  
    struct file *file, *tfile;  
    struct eventpoll *ep;  
    struct epitem *epi;  
    struct epoll_event epds;  
  
    error = -EFAULT;  
    //错误处理以及从用户空间将epoll_event结构copy到内核空间.
    if (ep_op_has_event(op) &&  
            // 复制用户空间数据到内核  
            copy_from_user(&epds, event, sizeof(struct epoll_event))) {  
        goto error_return;  
    }  
  
    // 取得 epfd 对应的文件  
    error = -EBADF;  
    file = fget(epfd);  
    if (!file) {  
        goto error_return;  
    }  
  
    // 取得目标文件  
    tfile = fget(fd);  
    if (!tfile) {  
        goto error_fput;  
    }  
  
    // 目标文件必须提供 poll 操作  
    error = -EPERM;  
    if (!tfile->f_op || !tfile->f_op->poll) {  
        goto error_tgt_fput;  
    }  
  
    // 添加自身或epfd 不是epoll 句柄  
    error = -EINVAL;  
    if (file == tfile || !is_file_epoll(file)) {  
        goto error_tgt_fput;  
    }  
  
    // 取得内部结构eventpoll  
    ep = file->private_data;  
  
    // EPOLL_CTL_MOD 不需要加全局锁 epmutex  
    if (op == EPOLL_CTL_ADD || op == EPOLL_CTL_DEL) {  
        mutex_lock(&epmutex);  
        did_lock_epmutex = 1;  
    }  
    if (op == EPOLL_CTL_ADD) {  
        if (is_file_epoll(tfile)) {  
            error = -ELOOP;  
            // 目标文件也是epoll 检测是否有循环包含的问题  
            if (ep_loop_check(ep, tfile) != 0) {  
                goto error_tgt_fput;  
            }  
        } else  
        {  
            // 将目标文件添加到 epoll 全局的tfile_check_list 中  
            list_add(&tfile->f_tfile_llink, &tfile_check_list);  
        }  
    }  
  
    mutex_lock_nested(&ep->mtx, 0);  
  
    // 以tfile 和fd 为key 在rbtree 中查找文件对应的epitem  
    epi = ep_find(ep, tfile, fd);  
  
    error = -EINVAL;  
    switch (op) {  
    case EPOLL_CTL_ADD:  
        if (!epi) {  
            // 没找到, 添加额外添加ERR HUP 事件  
            epds.events |= POLLERR | POLLHUP;  
            error = ep_insert(ep, &epds, tfile, fd);  
        } else {  
            error = -EEXIST;  
        }  
        // 清空文件检查列表  
        clear_tfile_check_list();  
        break;  
    case EPOLL_CTL_DEL:  
        if (epi) {  
            error = ep_remove(ep, epi);  
        } else {  
            error = -ENOENT;  
        }  
        break;  
    case EPOLL_CTL_MOD:  
        if (epi) {  
            epds.events |= POLLERR | POLLHUP;  
            error = ep_modify(ep, epi, &epds);  
        } else {  
            error = -ENOENT;  
        }  
        break;  
    }  
    mutex_unlock(&ep->mtx);  
  
error_tgt_fput:  
    if (did_lock_epmutex) {  
        mutex_unlock(&epmutex);  
    }  
  
    fput(tfile);  
error_fput:  
    fput(file);  
error_return:  
  
    return error;  
}  
```

####**2.ep_insert()**

```
//ep_insert()在epoll_ctl()中被调用, 完成往epollfd里面添加一个监听fd的工作
static int ep_insert(struct eventpoll *ep, struct epoll_event *event,  
                     struct file *tfile, int fd)  
{  
    int error, revents, pwake = 0;  
    unsigned long flags;  
    long user_watches;  
    struct epitem *epi;  
    struct ep_pqueue epq;  
    /* 
    struct ep_pqueue { 
        poll_table pt; 
        struct epitem *epi; 
    }; 
    */  
  
    // 增加监视文件数  
    user_watches = atomic_long_read(&ep->user->epoll_watches);  
    if (unlikely(user_watches >= max_user_watches)) {  
        return -ENOSPC;  
    }  
  
    // 分配初始化 epi  
    if (!(epi = kmem_cache_alloc(epi_cache, GFP_KERNEL))) {  
        return -ENOMEM;  
    }  
  
    INIT_LIST_HEAD(&epi->rdllink);  
    INIT_LIST_HEAD(&epi->fllink);  
    INIT_LIST_HEAD(&epi->pwqlist);  
    epi->ep = ep;  
    // 初始化红黑树中的key  
    ep_set_ffd(&epi->ffd, tfile, fd);  
    // 直接复制用户结构  
    epi->event = *event;  
    epi->nwait = 0;  
    epi->next = EP_UNACTIVE_PTR;  
  
    // 初始化临时的 epq  
    epq.epi = epi;  
    init_poll_funcptr(&epq.pt, ep_ptable_queue_proc);  
    // 设置事件掩码  
    epq.pt._key = event->events;  
    //  内部会调用ep_ptable_queue_proc, 在文件对应的wait queue head 上  
    // 注册回调函数, 并返回当前文件的状态  
    revents = tfile->f_op->poll(tfile, &epq.pt);  
  
    // 检查错误  
    error = -ENOMEM;  
    if (epi->nwait < 0) { // f_op->poll 过程出错  
        goto error_unregister;  
    }  
    // 添加当前的epitem 到文件的f_ep_links 链表  
    spin_lock(&tfile->f_lock);  
    list_add_tail(&epi->fllink, &tfile->f_ep_links);  
    spin_unlock(&tfile->f_lock);  
  
    // 插入epi 到rbtree  
    ep_rbtree_insert(ep, epi);  
  
    /* now check if we've created too many backpaths */  
    error = -EINVAL;  
    if (reverse_path_check()) {  
        goto error_remove_epi;  
    }  
  
    spin_lock_irqsave(&ep->lock, flags);  
  
    /* 文件已经就绪插入到就绪链表rdllist */  
    if ((revents & event->events) && !ep_is_linked(&epi->rdllink)) {  
        list_add_tail(&epi->rdllink, &ep->rdllist);  
  
  
        if (waitqueue_active(&ep->wq))  
            // 通知sys_epoll_wait , 调用回调函数唤醒sys_epoll_wait 进程  
        {  
            wake_up_locked(&ep->wq);  
        }  
        // 先不通知调用eventpoll_poll 的进程  
        if (waitqueue_active(&ep->poll_wait)) {  
            pwake++;  
        }  
    }  
  
    spin_unlock_irqrestore(&ep->lock, flags);  
  
    atomic_long_inc(&ep->user->epoll_watches);  
  
    if (pwake)  
        // 安全通知调用eventpoll_poll 的进程  
    {  
        ep_poll_safewake(&ep->poll_wait);  
    }  
  
    return 0;  
  
	error_remove_epi:  
    spin_lock(&tfile->f_lock);  
    // 删除文件上的 epi  
    if (ep_is_linked(&epi->fllink)) {  
        list_del_init(&epi->fllink);  
    }  
    spin_unlock(&tfile->f_lock);  
  
    // 从红黑树中删除  
    rb_erase(&epi->rbn, &ep->rbr);  
  
	error_unregister:  
    // 从文件的wait_queue 中删除, 释放epitem 关联的所有eppoll_entry  
    ep_unregister_pollwait(ep, epi);  
     
    spin_lock_irqsave(&ep->lock, flags);  
    if (ep_is_linked(&epi->rdllink)) {  
        list_del_init(&epi->rdllink);  
    }  
    spin_unlock_irqrestore(&ep->lock, flags);  
  
    // 释放epi  
    kmem_cache_free(epi_cache, epi);  
  
    return error;  
}
```

####**3.ep_eventpoll_poll()**

```
static unsigned int ep_eventpoll_poll(struct file *file, poll_table *wait)  
{  
    int pollflags;  
    struct eventpoll *ep = file->private_data;  
    // 插入到wait_queue  
    poll_wait(file, &ep->poll_wait, wait);  
    // 扫描就绪的文件列表, 调用每个文件上的poll 检测是否真的就绪,  
    // 然后复制到用户空间  
    // 文件列表中有可能有epoll文件, 调用poll的时候有可能会产生递归,  
    // 调用所以用ep_call_nested 包装一下, 防止死循环和过深的调用  
    pollflags = ep_call_nested(&poll_readywalk_ncalls, EP_MAX_NESTS,  
                               ep_poll_readyevents_proc, ep, ep, current);  
    // static struct nested_calls poll_readywalk_ncalls;  
    return pollflags != -1 ? pollflags : 0;  
}  
```

####**4.poll_wait()**

```
// 通用的poll_wait 函数, 文件的f_ops->poll 通常会调用此函数  
static inline void poll_wait(struct file * filp, wait_queue_head_t * wait_address, poll_table *p)  
{  
    if (p && p->_qproc && wait_address) {  
        // 调用_qproc 在wait_address 上添加节点和回调函数  
        // 调用 poll_table_struct 上的函数指针向wait_address添加节点, 并设置节点的func  
        // (如果是select或poll 则是 __pollwait, 如果是 epoll 则是 ep_ptable_queue_proc),  
        p->_qproc(filp, wait_address, p);  
    }  
}
```

####**5.ep_ptable_queue_proc()**

```
/* 
 * 该函数在调用f_op->poll()时会被调用.
 * 也就是epoll主动poll某个fd时, 用来将epitem与指定的fd关联起来的.
 * 关联的办法就是使用等待队列(waitqueue)
 */
static void ep_ptable_queue_proc(struct file *file, wait_queue_head_t *whead,
                 poll_table *pt)
{
    struct epitem *epi = ep_item_from_epqueue(pt);
    struct eppoll_entry *pwq;
    if (epi->nwait >= 0 && (pwq = kmem_cache_alloc(pwq_cache, GFP_KERNEL))) {
        /* 初始化等待队列, 指定ep_poll_callback为唤醒时的回调函数,
         * 当我们监听的fd发生状态改变时, 也就是队列头被唤醒时,
         * 指定的回调函数将会被调用. */
        init_waitqueue_func_entry(&pwq->wait, ep_poll_callback);
        pwq->whead = whead;
        pwq->base = epi;
        /* 将刚分配的等待队列成员加入到头中, 头是由fd持有的 */
        add_wait_queue(whead, &pwq->wait);
        list_add_tail(&pwq->llink, &epi->pwqlist);
        /* nwait记录了当前epitem加入到了多少个等待队列中,
         * 我认为这个值最大也只会是1... */
        epi->nwait++;
    } else {
        /* We have to signal that an error occurred */
        epi->nwait = -1;
    }
}
```

####**6.ep_poll_callback()**

```
//回调函数, 当我们监听的fd发生状态改变时, 它会被调用.
static int ep_poll_callback(wait_queue_t *wait, unsigned mode, int sync, void *key)
{
    int pwake = 0;
    unsigned long flags;
    //从等待队列获取epitem.需要知道哪个进程挂载到这个设备
    struct epitem *epi = ep_item_from_wait(wait);
    struct eventpoll *ep = epi->ep;//获取
    spin_lock_irqsave(&ep->lock, flags);

    if (!(epi->event.events & ~EP_PRIVATE_BITS))
        goto out_unlock;

    /* 没有我们关心的event... */
    if (key && !((unsigned long) key & epi->event.events))
        goto out_unlock;

    /* 
     * 这里看起来可能有点费解, 其实干的事情比较简单:
     * 如果该callback被调用的同时, epoll_wait()已经返回了,
     * 也就是说, 此刻应用程序有可能已经在循环获取events,
     * 这种情况下, 内核将此刻发生event的epitem用一个单独的链表
     * 链起来, 不发给应用程序, 也不丢弃, 而是在下一次epoll_wait
     * 时返回给用户.
     */
    if (unlikely(ep->ovflist != EP_UNACTIVE_PTR)) {
        if (epi->next == EP_UNACTIVE_PTR) {
            epi->next = ep->ovflist;
            ep->ovflist = epi;
        }
        goto out_unlock;
    }

    /* 将当前的epitem放入ready list */
    if (!ep_is_linked(&epi->rdllink))
        list_add_tail(&epi->rdllink, &ep->rdllist);

    /* 唤醒epoll_wait... */
    if (waitqueue_active(&ep->wq))
        wake_up_locked(&ep->wq);
    /* 如果epollfd也在被poll, 那就唤醒队列里面的所有成员. */
    if (waitqueue_active(&ep->poll_wait))
        pwake++;
		out_unlock:
    spin_unlock_irqrestore(&ep->lock, flags);
    
    /* We have to call this outside the lock */
    if (pwake)
        ep_poll_safewake(&ep->poll_wait);
    return 1;
}
```

####**7.ep_remove()**

```
static int ep_remove(struct eventpoll *ep, struct epitem *epi)  
{  
    unsigned long flags;  
    struct file *file = epi->ffd.file;  
  
    /* 
     * Removes poll wait queue hooks. We _have_ to do this without holding 
     * the "ep->lock" otherwise a deadlock might occur. This because of the 
     * sequence of the lock acquisition. Here we do "ep->lock" then the wait 
     * queue head lock when unregistering the wait queue. The wakeup callback 
     * will run by holding the wait queue head lock and will call our callback 
     * that will try to get "ep->lock". 
     */  
    ep_unregister_pollwait(ep, epi);  
  
    /* Remove the current item from the list of epoll hooks */  
    spin_lock(&file->f_lock);  
    if (ep_is_linked(&epi->fllink))  
        list_del_init(&epi->fllink);  
    spin_unlock(&file->f_lock);  
  
    rb_erase(&epi->rbn, &ep->rbr);  
  
    spin_lock_irqsave(&ep->lock, flags);  
    if (ep_is_linked(&epi->rdllink))  
        list_del_init(&epi->rdllink);  
    spin_unlock_irqrestore(&ep->lock, flags);  
  
    /* At this point it is safe to free the eventpoll item */  
    kmem_cache_free(epi_cache, epi);  
  
    atomic_long_dec(&ep->user->epoll_watches);  
  
    return 0;  
}  
```

####**8.ep_modify()**

```
static int ep_modify(struct eventpoll *ep, struct epitem *epi, struct epoll_event *event)  
{  
    int pwake = 0;  
    unsigned int revents;  
    poll_table pt;  
  
    init_poll_funcptr(&pt, NULL);  
  
    /* 
     * Set the new event interest mask before calling f_op->poll(); 
     * otherwise we might miss an event that happens between the 
     * f_op->poll() call and the new event set registering. 
     */  
    epi->event.events = event->events;  
    pt._key = event->events;  
    epi->event.data = event->data; /* protected by mtx */  
  
    /* 
     * Get current event bits. We can safely use the file* here because 
     * its usage count has been increased by the caller of this function. 
     */  
    revents = epi->ffd.file->f_op->poll(epi->ffd.file, &pt);  
  
    /* 
     * If the item is "hot" and it is not registered inside the ready 
     * list, push it inside. 
     */  
    if (revents & event->events) {  
        spin_lock_irq(&ep->lock);  
        if (!ep_is_linked(&epi->rdllink)) {  
            list_add_tail(&epi->rdllink, &ep->rdllist);  
  
            /* Notify waiting tasks that events are available */  
            if (waitqueue_active(&ep->wq))  
                wake_up_locked(&ep->wq);  
            if (waitqueue_active(&ep->poll_wait))  
                pwake++;  
        }  
        spin_unlock_irq(&ep->lock);  
    }  
  
    /* We have to call this outside the lock */  
    if (pwake)  
        ep_poll_safewake(&ep->poll_wait);  
  
    return 0;  
}  
```


###**epoll_wait()**

####**1.epoll_wait()**

```
SYSCALL_DEFINE4(epoll_wait, int, epfd, struct epoll_event __user *, events,
        int, maxevents, int, timeout)
{
    int error;
    struct file *file;
    struct eventpoll *ep;
    /* The maximum number of event must be greater than zero */
    if (maxevents <= 0 || maxevents > EP_MAX_EVENTS)
        return -EINVAL;
    /* Verify that the area passed by the user is writeable */
    /* 这个地方有必要说明一下:
     * 内核对应用程序采取的策略是"绝对不信任",
     * 所以内核跟应用程序之间的数据交互大都是copy, 不允许(也时候也是不能...)指针引用.
     * epoll_wait()需要内核返回数据给用户空间, 内存由用户程序提供,
     * 所以内核会用一些手段来验证这一段内存空间是不是有效的.
     */
    if (!access_ok(VERIFY_WRITE, events, maxevents * sizeof(struct epoll_event))) {
        error = -EFAULT;
        goto error_return;
    }
    /* Get the "struct file *" for the eventpoll file */
    error = -EBADF;
    /* 获取epollfd的struct file, epollfd也是文件嘛 */
    file = fget(epfd);
    if (!file)
        goto error_return;

    error = -EINVAL;
    /* 检查一下它是不是一个真正的epollfd... */
    if (!is_file_epoll(file))
        goto error_fput;

    /* 获取eventpoll结构 */
    ep = file->private_data;

    /* 等待事件到来~~ */
    error = ep_poll(ep, events, maxevents, timeout);
	error_fput:
    fput(file);
	error_return:
    return error;
}
```

####**2.ep_poll()**

```
/* 这个函数真正将执行epoll_wait的进程带入睡眠状态... */
static int ep_poll(struct eventpoll *ep, struct epoll_event __user *events,
           int maxevents, long timeout)
{
    int res, eavail;
    unsigned long flags;
    long jtimeout;
    wait_queue_t wait;//等待队列

    /* 计算睡觉时间, 毫秒要转换为HZ */
    jtimeout = (timeout < 0 || timeout >= EP_MAX_MSTIMEO) ?
        MAX_SCHEDULE_TIMEOUT : (timeout * HZ + 999) / 1000;
retry:
    spin_lock_irqsave(&ep->lock, flags);
    res = 0;
    /* 如果ready list不为空, 就不睡了, 直接干活... */
    if (list_empty(&ep->rdllist)) {
  
        /* OK, 初始化一个等待队列, 准备直接把自己挂起,
         * 注意current是一个宏, 代表当前进程 */
        init_waitqueue_entry(&wait, current);//初始化等待队列,wait表示当前进程
        __add_wait_queue_exclusive(&ep->wq, &wait);//挂载到ep结构的等待队列
        for (;;) {
            /* 将当前进程设置位睡眠, 但是可以被信号唤醒的状态,
             * 注意这个设置是"将来时", 我们此刻还没睡! */
            set_current_state(TASK_INTERRUPTIBLE);
            /* 如果这个时候, ready list里面有成员了,
             * 或者睡眠时间已经过了, 就直接不睡了... */
            if (!list_empty(&ep->rdllist) || !jtimeout)
                break;
            /* 如果有信号产生, 也起床... */
            if (signal_pending(current)) {
                res = -EINTR;
                break;
            }
            /* 啥事都没有,解锁, 睡觉... */
            spin_unlock_irqrestore(&ep->lock, flags);
            /* jtimeout这个时间后, 会被唤醒,
             * ep_poll_callback()如果此时被调用,
             * 那么我们就会直接被唤醒, 不用等时间了... 
             * 再次强调一下ep_poll_callback()的调用时机是由被监听的fd
             * 的具体实现, 比如socket或者某个设备驱动来决定的,
             * 因为等待队列头是他们持有的, epoll和当前进程
             * 只是单纯的等待...
             **/
            jtimeout = schedule_timeout(jtimeout);//睡觉
            spin_lock_irqsave(&ep->lock, flags);
        }
        __remove_wait_queue(&ep->wq, &wait);
        /* OK 我们醒来了... */
        set_current_state(TASK_RUNNING);
    }
    /* Is it worth to try to dig for events ? */
    eavail = !list_empty(&ep->rdllist) || ep->ovflist != EP_UNACTIVE_PTR;
    spin_unlock_irqrestore(&ep->lock, flags);
   
    /* 如果一切正常, 有event发生, 就开始准备数据copy给用户空间了... */
    if (!res && eavail &&
        !(res = ep_send_events(ep, events, maxevents)) && jtimeout)
        goto retry;
    return res;
}
```
####**3.ep_send_events()**

```
//调用p_scan_ready_list()
static int ep_send_events(struct eventpoll *ep,
              struct epoll_event __user *events, int maxevents)
{
    struct ep_send_events_data esed;
    esed.maxevents = maxevents;
    esed.events = events;
    return ep_scan_ready_list(ep, ep_send_events_proc, &esed);
}
```

####**4.ep_scan_ready_list()**

```
//由ep_send_events()调用本函数
static int ep_scan_ready_list(struct eventpoll *ep,
                  int (*sproc)(struct eventpoll *,
                       struct list_head *, void *),
                  void *priv)
{
    int error, pwake = 0;
    unsigned long flags;
    struct epitem *epi, *nepi;
    LIST_HEAD(txlist);
   
    mutex_lock(&ep->mtx);
   
    spin_lock_irqsave(&ep->lock, flags);
    /* 这一步要注意, 首先, 所有监听到events的epitem都链到rdllist上了,
     * 但是这一步之后, 所有的epitem都转移到了txlist上, 而rdllist被清空了,
     * 要注意哦, rdllist已经被清空了! */
    list_splice_init(&ep->rdllist, &txlist);
    /* ovflist, 在ep_poll_callback()里面我解释过, 此时此刻我们不希望
     * 有新的event加入到ready list中了, 保存后下次再处理... */
    ep->ovflist = NULL;
    spin_unlock_irqrestore(&ep->lock, flags);
  
    /* 在这个回调函数里面处理每个epitem
     * sproc 就是 ep_send_events_proc, 下面会注释到. */
    error = (*sproc)(ep, &txlist, priv);
    spin_lock_irqsave(&ep->lock, flags);
    
    /* 现在我们来处理ovflist, 这些epitem都是我们在传递数据给用户空间时
     * 监听到了事件. */
    for (nepi = ep->ovflist; (epi = nepi) != NULL;
         nepi = epi->next, epi->next = EP_UNACTIVE_PTR) {
       
        /* 将这些直接放入readylist */
        if (!ep_is_linked(&epi->rdllink))
            list_add_tail(&epi->rdllink, &ep->rdllist);
    }
   
    ep->ovflist = EP_UNACTIVE_PTR;
    
    /* 上一次没有处理完的epitem, 重新插入到ready list */
    list_splice(&txlist, &ep->rdllist);
    /* ready list不为空, 直接唤醒... */
    if (!list_empty(&ep->rdllist)) {
        
        if (waitqueue_active(&ep->wq))
            wake_up_locked(&ep->wq);
        if (waitqueue_active(&ep->poll_wait))
            pwake++;
    }
    spin_unlock_irqrestore(&ep->lock, flags);
    mutex_unlock(&ep->mtx);
    /* We have to call this outside the lock */
    if (pwake)
        ep_poll_safewake(&ep->poll_wait);
    return error;
}
```


**其他函数**

####**1.ep_send_events_proc()**

```
/* 该函数作为callbakc在ep_scan_ready_list()中被调用
 * head是一个链表, 包含了已经ready的epitem,
 * 这个不是eventpoll里面的ready list, 而是上面函数中的txlist.
 */
static int ep_send_events_proc(struct eventpoll *ep, struct list_head *head,
                   void *priv)
{
    struct ep_send_events_data *esed = priv;
    int eventcnt;
    unsigned int revents;
    struct epitem *epi;
    struct epoll_event __user *uevent;

    /* 扫描整个链表... */
    for (eventcnt = 0, uevent = esed->events;
         !list_empty(head) && eventcnt < esed->maxevents;) {
        /* 取出第一个成员 */
        epi = list_first_entry(head, struct epitem, rdllink);
        /* 然后从链表里面移除 */
        list_del_init(&epi->rdllink);
        /* 读取events, 
         * 注意events我们ep_poll_callback()里面已经取过一次了, 为啥还要再取?
         * 1. 我们当然希望能拿到此刻的最新数据, events是会变的~
         * 2. 不是所有的poll实现, 都通过等待队列传递了events, 有可能某些驱动压根没传
         * 必须主动去读取. */
        revents = epi->ffd.file->f_op->poll(epi->ffd.file, NULL) &
            epi->event.events;
        if (revents) {
            /* 将当前的事件和用户传入的数据都copy给用户空间,
             * 就是epoll_wait()后应用程序能读到的那一堆数据. */
            if (__put_user(revents, &uevent->events) ||
                __put_user(epi->event.data, &uevent->data)) {
                list_add(&epi->rdllink, head);
                return eventcnt ? eventcnt : -EFAULT;
            }
            eventcnt++;
            uevent++;
            if (epi->event.events & EPOLLONESHOT)
                epi->event.events &= EP_PRIVATE_BITS;
            else if (!(epi->event.events & EPOLLET)) {
                /* 嘿嘿, EPOLLET和非ET的区别就在这一步之差呀~
                 * 如果是ET, epitem是不会再进入到readly list,
                 * 除非fd再次发生了状态改变, ep_poll_callback被调用.
                 * 如果是非ET, 不管你还有没有有效的事件或者数据,
                 * 都会被重新插入到ready list, 再下一次epoll_wait
                 * 时, 会立即返回, 并通知给用户空间. 当然如果这个
                 * 被监听的fds确实没事件也没数据了, epoll_wait会返回一个0,
                 * 空转一次.
                 */
                list_add_tail(&epi->rdllink, &ep->rdllist);
            }
        }
    }
    return eventcnt;
}
```

**2.ep_free()**

```
/* ep_free在epollfd被close时调用,
 * 释放一些资源而已, 比较简单 */
static void ep_free(struct eventpoll *ep)
{
    struct rb_node *rbp;
    struct epitem *epi;
    /* We need to release all tasks waiting for these file */
    if (waitqueue_active(&ep->poll_wait))
        ep_poll_safewake(&ep->poll_wait);
   
    mutex_lock(&epmutex);
  
    for (rbp = rb_first(&ep->rbr); rbp; rbp = rb_next(rbp)) {
        epi = rb_entry(rbp, struct epitem, rbn);
        ep_unregister_pollwait(ep, epi);
    }
    
    /* 之所以在关闭epollfd之前不需要调用epoll_ctl移除已经添加的fd,
     * 是因为这里已经做了... */
    while ((rbp = rb_first(&ep->rbr)) != NULL) {
        epi = rb_entry(rbp, struct epitem, rbn);
        ep_remove(ep, epi);
    }
    mutex_unlock(&epmutex);
    mutex_destroy(&ep->mtx);
    free_uid(ep->user);
    kfree(ep);
}
```

###**函数的关系调用图**

![这里写图片描述](http://img.blog.csdn.net/20170729214921104?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYmFpeWVfeGluZw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

###**函数主要功能**

####**1.epoll_create**

* 从slab缓存中创建一个eventpoll对象,并且创建一个匿名的fd跟fd对应的file对象,而eventpoll对象保存在struct file结构的private指针中,并且返回,

* 该fd对应的file operations只是实现了poll跟release操作，创建eventpoll对象的初始化操作
获取当前用户信息,是不是root,最大监听fd数目等并且保存到eventpoll对象中

* 初始化等待队列,初始化就绪链表,初始化红黑树的头结点

####**2.epoll_ctl**

* 将epoll_event结构拷贝到内核空间中，并且判断加入的fd是否支持poll结(epoll,poll,selectI/O多路复用必须支持poll操作).

* 从epfd->file->privatedata获取event_poll对象,根据op区分是添加删除还是修改,

* 首先在eventpoll结构中的红黑树查找是否已经存在了相对应的fd,没找到就支持插入操作,否则报重复的错误，还有修改,删除操作。

* 插入操作时,会创建一个与fd对应的epitem结构,并且初始化相关成员，并指定调用poll_wait时的回调函数用于数据就绪时唤醒进程,(其内部,初始化设备的等待队列,将该进程注册到等待队列)完成这一步,

* epitem就跟这个socket关联起来了, 当它有状态变化时,会通过ep_poll_callback()来通知.

* 最后调用加入的fd的fileoperation->poll函数(最后会调用poll_wait操作)用于完注册操作，将epitem结构添加到红黑树中。

####**3.epoll_wait**

* 计算睡眠时间(如果有),判断eventpoll对象的链表是否为空,不为空那就干活不睡明.并且初始化一个等待队列,把自己挂上去,设置自己的进程状态

* 若是可睡眠状态.判断是否有信号到来(有的话直接被中断醒来,),如果没有那就调用schedule_timeout进行睡眠,

* 如果超时或者被唤醒,首先从自己初始化的等待队列删除,然后开始拷贝资源给用户空间了

* 拷贝资源则是先把就绪事件链表转移到中间链表,然后挨个遍历拷贝到用户空间,并且挨个判断其是否为水平触发,是的话再次插入到就绪链表


###**疑问**

#### **1.epoll_create中的size参数有什么作用？**

答：size这个参数其实没有任何用处，它只是为了保持兼容，因为之前的fd使用hash表保存，size表示hash表的大小，而现在使用红黑树保存，所以size就没用了。

####**2.LT和ET的区别（源码级别）？**

答：在源码中，两种模式的区别是一个if判断语句，通过ep_send_events_proc()函数实现，如果没有标上EPOLLET(即默认的LT)且“事件被关注”的fd就会被重新放回了rdllist。那么下次epoll_wait当然会又把rdllist里的fd拿来拷给用户了。

<br>
<br>
本人才疏学浅，若有错，请指出，谢谢！ 
如果你有更好的建议，可以留言我们一起讨论，共同进步！ 
衷心的感谢您能耐心的读完本篇博文！

> 参考链接：
> 1.[epoll源码分析](http://www.cnblogs.com/Przz/p/6876852.html)
> 2.[Epoll详解及源码分析](http://blog.csdn.net/chen19870707/article/details/42525887)
> 3.[linux 内核poll/select/epoll实现剖析](http://watter1985.iteye.com/blog/1614039)

