###**网络相关命令**

|序号|命令|功能|
|:-----|:-----|:------|
|1|ifconfig|查看网卡信息|
|2|ping|测试与目标主机的连通性|
|3|telnet|登录远程主机|
|4|apt-get|安装软件|
|5|netstat|显示网络系统的状态信息|
|6|ss|查看端口信息，比netstat更快|
<br>

**1.ifconfig**（network interfaces configuring）：查看网卡信息

![这里写图片描述](http://img.blog.csdn.net/20170701203621318?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYmFpeWVfeGluZw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

分析：可以看出输出信息分为两部分：eth0 和 lo


* eth0    ：表示第一块网卡  
```
Link encap:Ethernet  HWaddr 00:0c:29:b8:25:8e 
//  Link encap: 表示网络连接类型（以太网）， HWaddr : 表示网卡的mac地址，这是出厂时唯一的地址。
inet addr:192.168.236.134  Bcast:192.168.236.255  Mask:255.255.255.0
//inet addr :IPV4版本的IP地址； Bcast: 广播地址；Mask：子网掩码
//从上图可以看出我的IPV4地址：192.168.236.134 广播地址：192.168.236.255 子网掩码：255.255.255.0
inet6 addr: fe80::20c:29ff:feb8:258e/64 Scope:Link
//IPV6地址为：fe80::20c:29ff:feb8:258e/64 范围：Link
UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
//网卡正处在的工作状态,UP代表网卡开启状态,RUNNING代表网卡的网线被接上,MULTICAST表示支持组播,
//MTU:1500表示最大传输单元：1500字节 Metric：用于为接口创建的路由表分配度量值为1
RX packets:3532 errors:0 dropped:0 overruns:0 frame:0
//接收数据包情况统计
TX packets:2177 errors:0 dropped:0 overruns:0 carrier:0
//发送数据包情况统计
collisions:0 txqueuelen:1000 
//数据碰撞情况
RX bytes:4395455 (4.3 MB)  TX bytes:171750 (171.7 KB)
Interrupt:19 Base address:0x2000 
```

* lo    ：表示主机的回坏地址
 用来测试一个网络程序，但不想让局域网或外网的用户看到，只能在主机上运行和查看所有的网络接口，
```
Link encap:Local Loopback  
//连接类型：本地回环网络 ，HWaddr（硬件mac地址）
inet addr:127.0.0.1  Mask:255.0.0.0
//网卡的IP地址、子网掩码
inet6 addr: ::1/128 Scope:Host
//IPV6地址为：::1/128 范围：仅主机
UP LOOPBACK RUNNING  MTU:65536  Metric:1
//UP表示网卡正处于开启状态,LOOPBACK表示网卡处于回环状态，RUNNING代表网卡的网线被接上,
//MTU:65535表示最大传输单元：65535字节 Metric：用于为接口创建的路由表分配度量值为1
RX packets:262 errors:0 dropped:0 overruns:0 frame:0
//接收数据包情况统计
TX packets:262 errors:0 dropped:0 overruns:0 carrier:0
//发送数据包情况统计
collisions:0 txqueuelen:0 
//数据碰撞情况
RX bytes:21747 (21.7 KB)  TX bytes:21747 (21.7 KB)
```

**2.ping**（Packet InterNet Grouper ）:测试与目标主机的连通性

* -c n 显示n次，默认只要连通，就一直显示下去
* -R 记录路由过程
* -s 字节数：指定发送的数据字节数，预设值是56，加上8字节的ICMP头，一共是64ICMP数据字节

![这里写图片描述](http://img.blog.csdn.net/20170701210925365?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYmFpeWVfeGluZw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

**3.telnet**（TEminaL over Network）：登录远程主机

|参数|功能|
|:-----|:----|
|-d| 使用Socket的SO_DEBUG功能。|
|-f | 极限检测。大量且快速地送网络封包给一台机器，看它的回应。|
|-n |只输出数值。|
|-q |不显示任何传送封包的信息，只显示最后的结果。|
|-r |忽略普通的Routing Table，直接将数据包送到远端主机上。通常是查看本机的网络接口是否有问题。|
|-R| 记录路由过程。|
|-v |详细显示指令的执行过程。|
|<p>-c |数目：在发送指定数目的包后停止。|
|-i |秒数：设定间隔几秒送一个网络封包给一台机器，预设值是一秒送一次。|
|-I |网络界面：使用指定的网络界面送出数据包。|
|-l |前置载入：设置在送出要求信息之前，先行发出的数据包。|
|-p| 范本样式：设置填满数据包的范本样式。|
|-s| 字节数：指定发送的数据字节数，预设值是56，加上8字节的ICMP头，一共是64ICMP数据字节。|
|-t| 存活数值：设置存活数值TTL的大小。|

<br>
**4.apt-get**（Advanced Packaging Tool -get）：安装软件

例：我以redis为例

* sudo apt-get install redis  
安装软件

* sudo apt-get remove redis     
卸载软件，软件不可用，但保留软件配置文件  

* sudo apt-get remove --purge redis     
卸载软件，软件不可用，同时删除软件配置文件

* sudo apt-get autoremove redis     
卸载软件及依赖redis的软件包(redis-common)，软件不可用，但保留软件配置文件 

* sudo apt-get autoremove --purge redis   
 卸载软件及被依赖的软件包(redis -common)，软件不可用，同时删除软件配置文件


apt默认会把已安装和已卸载的软件都备份起来，如果用不到，无疑是占用了硬盘空间，使用以下指令可以清除：

* sudo apt-get autoclean    
清除已卸载软件的备份

* sudo apt-get clean    
清除已安装软件的备份

* sudo apt-get update   
同步软件源，这样才能获得最新的软件包。

* sudo apt-get upgrade   
更新已安装的软件，更新之后的版本就是本地索引的，所以upgrade之前一定要执行update，才是最新的

* apt-cache search string   
搜索字符串



**5.netstat**：显示网络系统的状态信息

|命令|功能|
|:------|:------|
|-a (all)|显示所有选项，默认不显示LISTEN相关|
|-t (tcp)|仅显示tcp相关选项|
|-u (udp)|仅显示udp相关选项|
|-n |拒绝显示别名，能显示数字的全部转化成数字。|
|-l| 仅列出有在 Listen (监听) 的服務状态|
|-p |显示建立相关链接的程序名|
|-r| 显示路由信息，路由表|
|-e |显示扩展信息，例如uid等|
|-s| 按各个协议进行统计|
|-c| 每隔一个固定时间，执行该netstat命令。|

<br>
实例：

![这里写图片描述](http://img.blog.csdn.net/20170701214858615?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYmFpeWVfeGluZw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

**6.ss**：查看端口状态

|命令|功能|
|:------|:------|
|-n （numeric）|不解析服务名称|
|-r （resolve）|解析主机名|
|-l （listening）| 显示监听状态的套接字|
|-a （all）|显示所有套接字|
|-o（options）|　　显示计时器信息|
|-e（extended）　|　显示详细的套接字（socket）的内存使用情况|
|-p（processed）|　　显示使用套接字的进程|
|-i（info） |　　显示 tcp 内部信息|
|-s（summary）|　　显示套接字（socket）使用概况|
|-t（tcp）　　|仅显示 TCP 套接字|
|-u（udp）　|　仅显示  UDP套接字|
|-d（dccp）　|仅显示 DCCP 套接字|
|-w（raw）　|　仅显示  RAW 套接字|
|-x（Unix） | 仅显示 Unix 套接字|

<br>

**实例**

![这里写图片描述](http://img.blog.csdn.net/20170726100723104?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYmFpeWVfeGluZw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)


###**进程相关命令**

|序号|命令|功能|
|:-----|:-----|:------|
|1|ps|查看进程|
|2|kill|终止进程|
|3|lsof|列出当前系统打开的文件|
|4|strace|跟踪系统|
|5|free|显示内存的使用情况|
|6|sysctl|动态地修改内核的运行参数|
|7|nice|改变进程的优先级|
|8|renice|改变运行的进程优先级|

<br>
**1.ps**（Process Status）：查看进程

|参数|描述|
|:-----|:------|
|-a| 显示所有用户的所有进程（包括其它用户）|
|-d |显示所有进程，但省略所有的会话引线(utility)|
|-e |显示所有进程，包括没有控制终端的程序|
|-f |用树形格式来显示进程；|
|-g| gid or groupname 显示组的所有进程。|
|-H| 显示进程的层次(和其它的命令合用|
|-j| 用任务格式来显示进程；-p pid 进程使用cpu的时间|
|-l| 长格式输出（有F,wchan,C 等字段）|
|-o| 用户自定义格式。-m 显示所有的线程|
|-r| 显示运行中的进程；-u 按用户名和启动时间的顺序来显示进程|
|-s| 以信号格式显示|
|-U| 显示该用户下的所有进程，且显示各个命令的详细路径|
|-v |以虚拟存储器格式显示|
|-w |宽行显示|
|-x| 显示没有控制终端的进程，同时显示各个命令的具体路径。dx不可合用。|

<br>
实例

![这里写图片描述](http://img.blog.csdn.net/20170701221304826?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYmFpeWVfeGluZw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

几种常用的参数组合：

![这里写图片描述](http://img.blog.csdn.net/20170704150843101?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYmFpeWVfeGluZw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

**2.kill** ：终止进程

|参数|描述|
|:-----|:------|
| -l  |列出全部的信号名称|
|-a  |当处理当前进程时，不限制命令名和进程号的对应关系|
|-p|  指定kill 命令只打印相关进程的进程号，而不发送任何信号|
|-s  |指定发送信号|
|-u | 指定用户 |

<br>
实例：

![这里写图片描述](http://img.blog.csdn.net/20170702075413444?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYmFpeWVfeGluZw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

**3.lsof**：（List Open Files）列出当前系统打开的文件

|参数|描述|
|:-----|:------|
|-a| 列出打开文件存在的进程|
|-c<进程名> |列出指定进程所打开的文件|
|-g | 列出GID号进程详情|
|-d<文件号>| 列出占用该文件号的进程|
|+d<目录>|  列出目录下被打开的文件|
|-i<条件> | 列出符合条件的进程。（4、6、协议、:端口、 @ip ）|
|-p<进程号> |列出指定进程号所打开的文件|
|-u  < username>| 查看用户打开哪些文件|
|-v| 显示版本信息|

<br>
实例：

![这里写图片描述](http://img.blog.csdn.net/20170702075927108?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYmFpeWVfeGluZw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

![这里写图片描述](http://img.blog.csdn.net/20170702081250291?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYmFpeWVfeGluZw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

![这里写图片描述](http://img.blog.csdn.net/20170702081631210?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYmFpeWVfeGluZw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

分析：每列所代表的含义如下：

* COMMAND：进程的名称
* PID：进程标识符
* USER：进程所有者
* FD：文件描述符，应用程序通过文件描述符识别该文件。如cwd、txt等
* TYPE：文件类型，如DIR、REG等
* DEVICE：指定磁盘的名称
* SIZE：文件的大小
* NODE：索引节点（文件在磁盘上的标识）
* NAME：打开文件的确切名称

**4.strace**（system trace）：系统追踪

|参数|描述|
|:-----|:------|
|-c| 统计每一次系统调用的所执行的时间,次数和出错的次数等.|
|-e| 仅仅被用来展示特定的系统调用（例如，open，write等等）|
|-f -F|选项告诉strace同时跟踪fork和vfork出来的进程
|-o < filename> |将strace的输出写入文件filename|
|-t| 在每行的输出之前添加时间戳|
|-p < pid> |跟踪指定的进程pid|
|-u < username>| 以username 的UID和GID执行被跟踪的命令|

<br>
实例：

![这里写图片描述](http://img.blog.csdn.net/20170702084927591?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYmFpeWVfeGluZw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

![这里写图片描述](http://img.blog.csdn.net/20170702085139735?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYmFpeWVfeGluZw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

![这里写图片描述](http://img.blog.csdn.net/20170702085038045?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYmFpeWVfeGluZw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

![这里写图片描述](http://img.blog.csdn.net/20170702085052979?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYmFpeWVfeGluZw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

**5.free**：显示内存的使用情况

|参数|描述|
|:-----|:------|
|-b |　以Byte为单位显示内存使用情况。|
|-k |　以KB为单位显示内存使用情况。|
|-m |　以MB为单位显示内存使用情况。|
|-h |  以比较人性化的方式显示内存使用情况|
|-V|  显示版本信息|

<br>
实例：

![这里写图片描述](http://img.blog.csdn.net/20170702225643956?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYmFpeWVfeGluZw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

分析：

1)Mem：表示物理内存统计

* total：表示物理内存总量(total = used + free)
* used：表示总计分配给缓存（包含buffers 与cache ）使用的数量，但其中可能部分缓存并未实际使用。
* free：未被分配的内存。
* shared：共享内存，一般系统不会用到，这里也不讨论。
* buffers：系统分配但未被使用的buffers 数量。
* cached：系统分配但未被使用的cache 数量

2)-/+ buffers/cache：表示物理内存的缓存统计

* -492 = used - buffers - cached
* +508 = free + buffers + cached

3)Swap：表示硬盘上交换分区的使用情况

**6.sysctl**：（system  control）：动态地修改内核的运行参数

* -a ：可以查看内核所有导出的变量
* -w：当改变sysctl设置

实例

![这里写图片描述](http://img.blog.csdn.net/20170702231931698?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYmFpeWVfeGluZw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

**7.nice**：改变进程的优先级

* -n  < number>设置优先级为number，取值的范围 -20 ~ 19。（优先级最高为-20，默认是0）

![这里写图片描述](http://img.blog.csdn.net/20170702233445436?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYmFpeWVfeGluZw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

**8.renice**：改变运行的进程优先级

* -u：指定开启进程的用户名
* -p < 程序识别码>：改变该程序的优先权等级

![这里写图片描述](http://img.blog.csdn.net/20170702234005033?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYmFpeWVfeGluZw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)


###**其它命令**

**1.帮助命令**

（1）man （manual pages ）：查看Linux中的指令帮助、配置文件帮助和编程帮助等信息

执行：man ulimit命令 输出结果如下

![这里写图片描述](http://img.blog.csdn.net/20170702234733859?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYmFpeWVfeGluZw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

（2）whatis：查询一个命令的功能

![这里写图片描述](http://img.blog.csdn.net/20170702235111724?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYmFpeWVfeGluZw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

（3）参数 --help ：查看命令的所有参数

![这里写图片描述](http://img.blog.csdn.net/20170702235409580?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYmFpeWVfeGluZw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

**2.关机命令**

（1）shutdown：安全的关闭/重启计算机

- k 不执行任何关机操作，只发出警告信息给所有用户
- r 重新启动计算机
- h 关机并彻底断电
- f 快速关机且重启动时跳过fsck
- n 快速关机不经过init程序
- c 取消之前的定时关机

实例：

```
//例1：系统在十分钟后关机并且马上重新启动
 shutdown –r +10
//例2：系统马上关机并且不重新启动
shutdown –h now
//例3：系统定时5分钟后关机，并将定时关机的命令放在后台，则：
shutdown -h +5 &
//例4：定时在23：30关闭计算机：
shutdown -h 23：30
//例5：终止之前执行的定时关机命令
shutdown -c
```



（2）reroot：重新启动系统

参数

* -f ：强制重新开机，不调用shutdown指令的功能。

* -i：关闭网络设置之后再重新启动系统

* -n：保存数据后再重新启动系统

（3）init：进程初始化工具

|参数|功能|
|:----|:------|
|0|关机|
|1|单用户|
|2|多用户，不联网|
|3|完全多用户模式（需联网）|
|4|不使用的|
|5|xwindows：有界面的|
|6|重启|

<br>
<br>
本人才疏学浅，若有错，请指出，谢谢！ 
如果你有更好的建议，可以留言我们一起讨论，共同进步！ 
衷心的感谢您能耐心的读完本篇博文


