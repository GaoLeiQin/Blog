###**Linux发展史**


####**1.Linux的诞生**

Linux最初是由芬兰赫尔辛基大学学生Linus Torvalds由于自己不满意教学中使用的MINIX操作系统，所以在1990年底由于个人爱好设计出了LINUX系统核心。后来发布于芬兰最大的ftp服务器上，用户可以免费下载，所以它的周边的程序越来越多，Linux本身也逐渐发展壮大起来。

 
####**2.Linux的发展简史**

1968年 

> Internet的先驱ARPANET建立。虽然ARPANET的设计目的是使研究人员在合作一个项目时可以共享代码和信息，但是它也成为了对开放源代码可行的一个展示。

1969年

> 贝尔实验室的研究员Ken Thompson编写了Unix的第一个版本，这是一个多用户、多任务的操作系统。整个七十年代，Unix的代码都在免费传播，它迅速成为了在大学和研究机构中很流行的系统。

1971年

> 作为开放源码的先驱，Richard Stallman加入了麻省理工学院的一个专门研究免费软件的组织。作为Emacs文本编辑程序的开发者，他后来建立了GNU项目，最终导致了免费的Linux操作系统的诞生。

1972年

>     来自DARPA(Defense Advanced Research Projects Agency)的Vinton Cerf和Bob Kahn开发了TCP/IP协议-该协议成为了Internet的网络基础。十年后，美国国防部为Internet命名，并且要求连入Internet的计算机都是用TCP/IP协议

1979年

> AT&T宣布了使Unix商业化的计划。这导致加州大学伯克利分校建立自己的Unix版本，成为BSD(Berkeley Software Distributions)Unix。BSD Unix被DEC和SUN这样的商业公司所接受。后来AT&T和SUN同意将各自的Unix版本合并，并且推进其竞争对手(DEC、HP，以及IBM)共同建立开放软件基金。

1983年

> 为了反对软件所有权私有化的趋势，Stallman建立了GNU计划来推进免费软件模型，并为此开发了个免费的操作系统、应用程序以及开发工具。更重要的是，GNU建立了General Public License(GPL)，也就是Copyleft，它成为许多开放源码软件所采用的模型。

1986年

> Larry Wall建立了Perl(Practical Extraction and Report Language)，这是一种编写CGI程序广泛采用的通用变成语言。CGI为Web带来了更多动态内容。

1987年

> 开发者Andrew Tanenbaum发布了Minix，这是一个为PC、Mac、Amiga以及Atari ST设计的Unix版本，在发布时带有完整的源代码。

1989年

> 芬兰赫尔辛基大学的一名学生Linus Torvalds为了超越Minix，发布了一个新的Unix变种----Linux。三年后，Linux正式接受GPL。今天，按照Red Hat Software的说法，全球有大约1000万Linux用户。

1993年

> FreeBSD 1.0发布。这个系统以BSD Unix为基础，包括网络、虚拟内存、任务切换以及长文件名等功能。BSD许可不需要开发者反馈任何东西。

1994年

> Marc Ewing建立Red Hat Linux，用以解决Linux易用性方面的问题。Red Hat包含Linux、第三方软件、文档以及初级技术支持，售价为50美元。Red Hat迅速成为领先的Linux发行人。同年，Bryan Sparks在前Novell的首席执行长官Ray Noorda的支持下建立Caldera。

1995年

> Apache Group建立了一种新的Web Server-Apache，该服务器以NCSA(National Center for Supercomputing Applications)的HTTPd 1.3以及一系列的补丁为基础。这种免费的Web Server成为最流行的HTTP server。

1997年

>  程序员Eric S. Raymond(也是《新黑客字典》的作者)发表了名为《大教堂和集市》的文章，对比了商业开发模型以及开放源码开发模型，该文章成为网景公司开办Mozilla.org的灵感之源。

1998年

    

> Netscape宣布不仅Communicator 5.0是免费的，而且还将发布其源代码。几个主要的软件厂商，包括CA、Corel、IBM、Informix、Interbase、Oracle以及Sybase，宣布了支持Linux的产品计划。 陷入反托拉斯诉讼的Microsoft，在一份声明中引Linux为例用以说明其在操做系统方面没有垄断地位。不久以后，万圣节文档----一系列Microsoft内部讨论开放源码软件和Linux威胁的备忘录被泄漏给了开放源码团体，并且在Web上公布。

1999年

> Linux 2.2发布，GNOME 1.0发布，支持Linux 2.2的Red Hat 6.0发布。IBM推出全面支持Linux计划，HP宣布支持Linux。

1999年-2017年

> 各种Linux版本不断发布，不断发展壮大!


####**3.Linux发展过程图解**

![这里写图片描述](http://img.blog.csdn.net/20170701100746975?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYmFpeWVfeGluZw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)



###**Linux介绍**

####**1.定义**

* Linux是一套免费使用和自由传播的类Unix操作系统，是一个基于POSIX和UNIX的多用户、多任务、支持多线程和多CPU的操作系统。

* Linux能运行主要的UNIX工具软件、应用程序和网络协议。它支持32位和64位硬件。Linux继承了Unix以网络为核心的设计思想，是一个性能稳定的多用户网络操作系统

####**2.Linux的组成**

* 内核：是系统的心脏，是运行程序和管理像磁盘和打印机等硬件设备的核心程序。

* Shell：是系统的用户界面，提供了用户和内核进行交互操作的一种接口。它接收用户输入的命令并把它送入内核去执行，是一个命令解释器。但它不仅使命令解释器，而且还是高级编程语言，shell编程。

* 文件系统：文件系统是文件存放在磁盘等存储设备上的组织方法，Linux支持多种文件系统，如ext3,ext2,NFS,SMB,iso9660等

* 应用程序：标准的Linux操作系统都会有一套应用程序例如X-Window,Open Office等

####**3.Linux的发行版**

* 目前市面上较知名的发行版有：Ubuntu（应用于个人桌面）、RedHat（应用于服务器）、CentOS、Debian、Fedora、SuSE、OpenSUSE、TurboLinux、BluePoint、RedFlag、Xterm、SlackWare等

* Linux内核版本号分为稳定版本（偶数）和开发版本（奇数）


###**Linux 的特点**

####**1.系统稳定**

Linux是基于Unix的思想开发出来的操作系统，因此，Linux具有与Unix系统相似的的程序接口跟操作方式，当然也继承了Unix稳定并且有效率的特点。从用户的使用过程中，听到安装 Linux 的主机连续运作一年以上而不曾当机、不必关机是稀松平常的事，而不会出现当用户使用Windows时，类似蓝屏死机那样的现象。 

####**2.费用便宜**

由于Linux是基于GPL( General Public License )的架构之下，因此他是 Free 的，也就是任何人都可以免费的使用或者是修改其中的源码！

####**3.安全性高**

 Linux 由于支持者日众，有相当多的热心团体、个人参与其中的开发，因此可以随时获得最新的安全信息，并给予随时的更新，亦即是具有相对的较安全！ 

####**4.多用户、多任务**

与Windows系统不同的，Linux主机上可以同时允许多人上线来工作，并且资源的分配较为公平。这个多人多任务特点可是Linux系统中相当好的一个功能，我们可以在一台 Linux 主机上面规划出不同等级的使用者，而且每个使用者登入系统时的工作环境都可以不相同，此外，还可以允许不同的使用者在同一个时间登入主机，以同时使用主机的资源

####**5.应用丰富**

由于目前有很多的软件逐渐被这套操作系统所使用，而更多的软件套件也正在Linux系统上面进行着发展和测试，因此，Linux近来已经可以独力完成几乎所有的工作站或服务器的服务了，例如Web, Mail, FTP，DNS,Proxy服务等等。但有些专业软件没有Linux版本；

####**6.可移植性强**

Unix和Linux的代码是由90%的c语言和10%的汇编组成，因此只需要稍加修改，就能移植到其他硬件上（适合嵌入式系统）；Linux对于硬件的需求是很低的，


###**Linux目录结构**


![这里写图片描述](http://img.blog.csdn.net/20170701102903006?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYmFpeWVfeGluZw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)



###**Linux 与 Windows/Unix 的区别**

####**1.Linux 与 Windows 的区别**


|比较	|Windows	|Linux|
|:------|:---------|:--------|
|界面	|界面统一，外壳程序固定所有Windows程序菜单几乎一致，快捷键也几乎相同	|图形界面风格依发布版不同而不同，可能互不兼容。GNU/Linux的终端机是从UNIX传承下来，基本命令和操作方法也几乎一致|
|驱动程序	|驱动程序丰富，版本更新频繁。默认安装程序里面一般包含有该版本发布时流行的硬件驱动程序，之后所出的新硬件驱动依赖于硬件厂商提供。对于一些老硬件，如果没有了原配的驱动有时很难支持。另外，有时硬件厂商未提供所需版本的Windows下的驱动，也会比较头痛。	|由志愿者开发，由Linux核心开发小组发布，很多硬件厂商基于版权考虑并未提供驱动程序，尽管多数无需手动安装，但是涉及安装则相对复杂，使得新用户面对驱动程序问题（是否存在和安装方法）会一筹莫展。但是在开源开发模式下，许多老硬件尽管在Windows下很难支持的也容易找到驱动。HP、Intel、AMD等硬件厂商逐步不同程度支持开源驱动，问题正在得到缓解|
|使用	|使用比较简单，容易入门。图形化界面对没有计算机背景知识的用户使用十分有利。|	图形界面使用简单，容易入门。文字界面，需要学习才能掌握。|
|学习	|系统构造复杂、变化频繁，且知识、技能淘汰快，深入学习困难。|	系统构造简单、稳定，且知识、技能传承性好，深入学习相对容易。|
|软件	|每一种特定功能可能都需要商业软件的支持，需要购买相应的授权。|	大部分软件都可以自由获取，同样功能的软件选择较少。|
|价格|对源代码实行知识产权保护的传统商业系统，收费|开源系统，廉价甚至免费|
|进程分配|一个准微内核操作系统，许多功能都是以单独的进程来实现，从而提高了系统的模块化程度，而在进程切换时的开销要大一些|一个单一块式的操作系统，操作系统通常在单用户进程的内存空间内进行，可免去发生系统调用时候进程切换导致的资源消耗。|
|进程间通信|在基本IPC机制的基础上，提供了许多直接面向应用程序的高级IPC机制，IPC机制有管道、命名管道、消息传递、共享内存、信号量等|在进程间通信机制上，Linux提供了标准的UNIX IPC机制，提供了更大的灵活性|
|内存管理|Windows支持DOS和Win16程序的执行，但为了提供这种兼容性，它的内存管理付出了极高的性能代价|Linux内核为用户管理了非常多的细节问题，用户可以认为自己真正拥有4GB地址空间，而不用关心虚拟内存是否提交物理存储等问题|

<br>
####**2.Linux和UNIX区别**

（1）开放性

Linux是开放源代码的自由软件，而UNIX是对源代码实行知识产权保护的传统商业软件。这种不同体现在用户对Linux有很高的自主权，而对UNIX却只能去被动的适应；

（2）价格

Linux廉价，而UNIX昂贵。与Windows以及其它商品化UNIX操作系统相比，Linux的一个显而易见的优势就是廉价。硬件的花销加上很少的软件费用就可以拥有一个PC工作站或服务器，这方面显然是其它操作系统无法比拟的。而且Linux对于硬件的要求比Windows要低得多。一般的用户也可以利用Linux来构造一个高性能的集群。

（3）应用

* UNIX在操作系统和服务器性能上稍占优势。一般比较有经济实力的企业还是选择了UNIX操作系统

* Linux应用在嵌入式设备、超级计算机和服务器，并且在服务器领域确定了地位，通常服务器使用LAMP（Linux + Apache + MySQL + PHP）或LNMP（Linux + Nginx+ MySQL + PHP）组合。目前Linux不仅在家庭与企业中使用，并且在政府中也很受欢迎（巴西）

注：大部分Linux是由UNIX兼容系统变化而来的，有时也称为UNIX的克隆系统，它具备UNIX系统的全部特征。在Linux操作系统上的命令、用户管理方式、各种服务的配置方法等几乎和UNIX操作系统上一样。当然，Linux和UNIX操作系统的核心思想也绝对是相同的。



<br>
<br>

本人才疏学浅，若有错，请指出，谢谢！ 
如果你有更好的建议，可以留言我们一起讨论，共同进步！ 
衷心的感谢您能耐心的读完本篇博文

参考资料：[Linux概述](http://www.360doc.com/content/17/0701/10/44947949_667918207.shtml)



