###**引入**

在讲Socket编程之前，先来回顾一下[TCP的三握手与四挥手过程](http://blog.csdn.net/baiye_xing/article/details/73864507)，这样就更易理解Socket编程了。

**1.TCP三握手建立连接过程**

![这里写图片描述](http://img.blog.csdn.net/20170628200233128?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYmFpeWVfeGluZw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)



**2.TCP四挥手释放连接过程**

![这里写图片描述](http://img.blog.csdn.net/20170628200811862?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYmFpeWVfeGluZw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

###**Socket编程**

先来张图，看看其过程如何：

![这里写图片描述](http://img.blog.csdn.net/20170703214159846?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYmFpeWVfeGluZw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

针对上图出现的函数或其它重要的函数，我在此解释下：

**1.socket()**：申请了一个描述符（本地可见）,但不能对外提供通讯

（1）函数

int socket（int domain , int type , int protocol）;

（2）参数

* domain 需要被设置为“AF_INET”（或者其他值）

* type 告诉内核这个socket 是什么类型，“SOCK_STREAM”或是“SOCK_DGRAM”；

* protocol 设置为0

**2.bind()**：组合IP和端口，对外提供连接

（1）函数

int bind (int sockfd , struct sockaddr *my_addr , int addrlen)

（2）参数

* sockfd 是由socket()函数返回的套接字描述符。

* my_addr 是一个指向struct sockaddr 的指针，包含有关你的地址的信息：名称、端口和IP 地址。

* addrlen 可以设置为sizeof(struct sockaddr)。


**3.listen()**：监听绑定的IP端口上的数据，将半连接转到全连接中

（1）函数

int listen(int sockfd, int backlog)

（2）参数

* sockfd 是一个套接字描述符，由socket()系统调用获得。

* backlog 是未经过处理的连接请求队列可以容纳的最大数目。每一个连入请求都要进入一个连入请求队列，即请求队列的最大值。


**4.accept()**：从listen监听的全连接队列中取连接，申请文件描述符，

（1）函数

int accept(int sockfd, void *addr, int *addrlen)

（2）参数

* sockfd 是正在listen() 的一个套接字描述符。

* addr 一个指向struct sockaddr_in 结构的指针；里面存储着远程连接过来的计算机的信息的IP 地址和端口

* addrlen 是一个本地的整型数值，在它的地址传给accept() 前它的值应该是sizeof(struct sockaddr_in)；accept()不会在addr 中存储多余addrlen bytes 大小的数据。如果accept()函数在addr 中存储的数据量不足addrlen，则accept()函数会改变addrlen 的值来反应这个情况。

**5.connect()**：连接到指定的IP

（1）函数

int connect (int sockfd, struct sockaddr *serv_addr, int addrlen)

（2）参数

* sockfd ：套接字文件描述符，由socket()函数返回的。

* serv_addr 是一个存储远程计算机的IP 地址和端口信息的结构。

* addrlen 应该是sizeof(struct sockaddr)。


**6.write()**：写入数据

（1）函数

ssize_t write(int fd, const void*buf,size_t nbytes)

（2）参数

* fd 为要写入的文件的描述符
* buf 为要写入的数据的缓冲区地址，
* nbytes 为要写入的数据的字节数。

（3）返回值

* ssize_t 是通过 typedef 声明的 signed int 类型，即size_t 是通过 typedef 声明的 unsigned int 类型；ssize_t 在 "size_t" 前面加了一个"s"，代表 signed。

* write() 函数会将缓冲区 buf 中的 nbytes 个字节写入文件 fd，成功则返回写入的字节数，失败则返回 -1。

**7.read()**：读取数据

（1）函数

ssize_t read(int fd,void *buf,size_t nbyte)

（2）参数

* fd 为要读取的文件的描述符，
* buf 为要接收数据的缓冲区地址，
* nbytes 为要读取的数据的字节数。

（3）返回值

read() 函数会从 fd 文件中读取 nbytes 个字节并保存到缓冲区 buf，成功则返回读取到的字节数（但遇到文件结尾则返回0），失败则返回 -1。

**8.send()**：发送数据（通过TCP）

（1）函数

int send(int sockfd, const void *msg, int len, int flags)

（2）参数

* sockfd 是代表你与远程程序连接的套接字描述符。

* msg 是一个指针，指向你想发送的信息的地址。

* len 是你想发送信息的长度。

* flags 发送标记。一般都设为0。

**9.recv()**：接收数据（通过TCP）

（1）函数

int recv(int sockfd, void *buf, int len, unsigned int flags）

（2）参数

* sockfd 是你要读取数据的套接字描述符。

* buf 是一个指针，指向你能存储数据的内存缓存区域。

* len 是缓存区的最大尺寸。

* flags 是recv() 函数的一个标志，一般都为0 。

**10.close()** ：关闭连接

（1）使用方法

close(sockfd)

（2）功能

执行close()之后，套接字将不会在允许进行读操作和写操作

（3）底层实现：引用计数

**11.shutdown()**

（1）函数

int shutdown（int sockfd, int how）

（2）参数

* sockfd 是一个你所想关闭的套接字描述符．
* how 可以取：0 表示不允许以后数据的接收操；1 表示不允许以后数据的发送操作；2 表示和close()一样，不允许以后的任何操作（包括接收，发送数据）

（3）how的方式

* SHUT_RD：关闭连接的读端。
* SHUT_WR:关闭连接的写端。
* SHUT_RDWR:相当于调用shutdown两次：首先是SHUT_RD,然后SHUT_WR

**12.sendto()**：发送数据（通过UDP）

（1）函数

int sendto（int sockfd, const void *msg, int len, unsigned int flags,const struct sockaddr *to, int tolen）;

（2）参数

* sockfd 是代表你与远程程序连接的套接字描述符。
* msg 是一个指针，指向你想发送的信息的地址。
* len 是你想发送信息的长度。
* flags 发送标记。一般都设为0。

**13.recvfrom()**：接收数据（通过UDP）

（1）函数

int recvfrom(int sockfd, void *buf, int len, unsigned int flags,struct sockaddr *from, int *fromlen)

（2）参数 

* sockfd 是你要读取数据的套接字描述符。
* buf 是一个指针，指向你能存储数据的内存缓存区域。
* len 是缓存区的最大尺寸。
* flags 是recv() 函数的一个标志，一般都为0 
* from 是一个本地指针，指向一个struct sockaddr 的结构（里面存有源IP 地址和端口数）．
* fromlen 是一个指向一个int 型数据的指针，它的大小应该是sizeof （ struct sockaddr）．当函数返回的时候，formlen 指向的数据是form 指向的struct sockaddr 的实际大小．


###**小疑问**

**1.一个客户端对外最多提供多少连接（一台服务器最多接收多少端口）？**

答：由内核内存上限（listen所能监听的两个队列大小）和文件描述符大小（进程所能打开的文件个数上限）决定。取两个值中较小的值

注：使用ulimit -n 命令可以设置文件描述符的上限

**2.服务器怎么处理百万连接（c10k问题）?**

答：一种方法是采用一个进程/线程一个连接，但是这样太消耗性能，另一种方法则是一个进程/线程创建多个连接（io多路复用），即使用epoll技术


**3.为什么要设置TCP缓冲区?**

答：①	减少系统调用次数（减少内核读取次数）；②	保证网络带宽利用率

**4.send/recv函数分别会在什么时候阻塞?**

答：send函数（发送缓冲区）在buffer的允许发送但未发送区已满时阻塞；而recv函数（接受缓冲区）会在已确认未读取的数据为空时阻塞。

**5.五元组指的是什么？**

答：五元分别指：网络协议、源IP、源端口、目的IP、目的端口
	
例：通过命令 netstat –an | grep tcp 获取五元组

![这里写图片描述](http://img.blog.csdn.net/20170704132251482?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYmFpeWVfeGluZw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)


分析：从上图可以看出，每列分别表示Protocol、Recv-Q 、Send-Q、源IP：源端口、目的IP：目的端口 State，从而可以清晰的看出tcp的五元组，

**6.怎样获取内核缓冲区的大小？**

![这里写图片描述](http://img.blog.csdn.net/20170704133949804?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYmFpeWVfeGluZw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)


分析：从上图可以看出，内核缓冲区为163840、tcp接收缓冲区为87380、 tcp 发送缓冲区为16384


**7.什么是带外数据**

答：带外数据拥有比一般数据高的优先级，但也是通过连接传输数据的，TCP 是由一种叫做“紧急模式”的方法来传输带外数据的，当TCP 收到一个包含URG 标志的数据段时，，TCP 会检查“紧急指针”来验证是否紧急指针所指的数据已经到达本地，然后通过PSH标志立即发送数据。

**8.listen（fd,backlog）函数的作用**

答：用于获取两个队列（半连接、全连接），前三次连接是在半连接队列，之后转入全连接，

注：文件位置在：/proc/syn/net/ipv4/

**9.为什么不将listen和accept组合为一个函数？**

答：因为listen是非阻塞的。处理三次握手，accept负责listen处理过的连接，如果组合为一个函数，那么就只能是顺序的三握手四挥手，没有并发的功能了。

**10. listen为什么是非阻塞的？为什么维护两个连接？**

答：非阻塞是因为编程需要，阻塞时数据有交互；两个连接是因为要维护数据转换过程，accept需要的时候就可以从listen中拿到，不用循环遍历

**11.accept为什么是阻塞的？**

答：若是非阻塞的，就要等待listen的全连接队列、轮询会消耗资源。所以设置accept函数为阻塞式函数。


<br>
<br>
本人才疏学浅，若有错，请指出，谢谢！ 
如果你有更好的建议，可以留言我们一起讨论，共同进步！ 
衷心的感谢您能耐心的读完本篇博文！ 

