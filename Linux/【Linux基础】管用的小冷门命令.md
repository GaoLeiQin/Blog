
**前言**


* 有些命令我们可能不经常使用，甚至都没听说过，但在某些时候还是很有用的，下面我列举了一些我从网上爬来的几个命令，希望看了对你有所帮助，同时也希望你在底下评论区告诉我几个特别的命令!

|命令|功能|
|:------|:-----|
|ipcs|查看共享内存，消息队列，信号的信息|
|ipcrm|移除一个消息对象、共享内存段或信号集|
|stat|显示指定文件的详细信息，比ls更详细|
|hostname  |显示主机名|
|uname|   显示系统信息|
|source|使修改的文件立即生效，不用重启系统，一般在环境变量配置后执行|
|wall | （ write all）广播|
|cal|日历|
|time|执行命令，并计算执行时间|
|file|确定文件类型|
|w|当前登陆用户|
|env|执行一个命令（脚本文件中很有用）|
|expr|计算表达式、布尔操作或正则匹配|
|m4|简单地宏处理器|
|yes|多次打印字符串|
|printenv| 打印环境变量(在调试时或者脚本中很管用)|
|look| 找出以某字符串开头的英文单词(或者文件中的某一行)|
|cut, paste 和 join |数据处理|
|fmt| 格式化文本段落|
|pr | 将文本格式化为页数据或者列数据|
|fold | 封装文本中的行【比如 -w 指定宽度，不使用默认的80】|
|Column| 将文本格式化为列或者表数据|
|expand 和 unexpand|制表符与空格之间转换|
|seq|打印序列数字|
|bc|计算器|
|factor|分解因数 |
|gpg|加密并签名文件|
|toe|终端类型列表|
|nc|网络调试及数据传输|
|socat|套接字代理，与 netcat 类似|
|slurm|网络负载监视器|
|tree|以树的形式显示路径和文件，类似于 ls，不过这条命令会递归显示|
|Shuf |将文件中的数据随机选择排列|
|comm|逐行比较已排序的文件|
|pv|监控通过管道的数据|
|hd 和 bvi|保存或者编辑二进制文件|
|strings| 提取二进制文件的文本内容|
|Tr| 字符转换与处理|
|Iconv 或 uconv|文本编码的转换|
|Spit 和scplit | 分割文件|
|Sponge| 在写之前读取所有输入，在对同一个文件读写很管用|
|units|单位转化与计算；将一种计量单位转换为另一种等效的计量单位|
|7z| 一种高效的压缩工具|
|Ldd| 查看动态库的信息|
|Nm| 提取可执行文件或者 obj 文件的符号|
|Ab|web 服务器性能分析工具|
|Mtr|网络调试跟踪工具|
|Cssh| 可视化的并发 shell|
|Rsync |可用于远程文件目录同步|
|Wireshark 和tshark  抓取包与网络调试|
|Ngrep| 网络层的 grep 工具|
|Host 和 dig| DNS 查找|
|dstat | 通用的系统统计工具|
|glances|高层次的多子系统概览|
|last|登入历史记录|
|id|用户/组 ID 信息|
|sar| 系统历史数据统计|
|iftop 或 nethogs|套接字及进程的网络利用率|
|dmesg|引导及系统错误信息|
|hdparm：SATA/ATA |磁盘操作及性能分析|
|lsb_release|Linux 发行版信息|
|lsblk|块设备信息：树状图展示你的磁盘以及磁盘分区信息|
|lshw，lscpu，lspci，lsusb |查看硬件信息，包括 CPU、BIOS、RAID、显卡、其他设备等|


<br>

**实例**


下面是我试了几个命令的输出结果，如果你有兴趣可以都试一下哟！

![这里写图片描述](http://img.blog.csdn.net/20170703203506887?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYmFpeWVfeGluZw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)
![这里写图片描述](http://img.blog.csdn.net/20170703203514000?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYmFpeWVfeGluZw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)
![这里写图片描述](http://img.blog.csdn.net/20170703204848959?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYmFpeWVfeGluZw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

**time命令**

![这里写图片描述](http://img.blog.csdn.net/20170703204904791?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYmFpeWVfeGluZw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

分析：有三个参数

* real：实际时间，从command命令行开始执行到运行终止的消逝时间；

* user：用户CPU时间，命令执行完成花费的用户CPU时间，即命令在用户态中执行时间总和；

* system：系统CPU时间，命令执行完成花费的系统CPU时间，即命令在核心态中执行时间总和；

本人才疏学浅，若有错，请指出，谢谢！
若你还知道别的实用的命令，麻烦你在底下评论区告诉我一下，感激不尽！
