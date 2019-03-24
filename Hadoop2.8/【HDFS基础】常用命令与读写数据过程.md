###**HDFS常用命令**

|命令|功能|
|:----|:------|
|start-dfs.sh| 启动HDFS|
|start-balancer.sh|执行负载均衡|
|hadoop fs –put example.txt /suger |将本地文件复制到HDFS中|
|hadoop fs -copyFromLocal /home/time.txt /suger|将本地文件复制到HDFS中|
|hadoop fs –get /suger/example.txt copyexample.txt|将文件从HDFS复制到本地| 
|hadoop fs –copyToLocal /suger/example.txt /home/copytime.txt|将文件从HDFS复制到本地|
|hadoop fs –mkdir testdir|创建空文件夹|
|hadoop fs –cat example.txt |查看文件|
| hadoop fs –tail example.txt|查看文件的最后10行|
|hadoop fs –ls  / |列出本地文件系统根目录下的文件|
|hadoop fs –lsr /|查看HDFS全部文件列表（递归输出）|
|hadoop fs –rmr /usr/admin/files.har|递归删除文件|
|hadoop dfsadmin –report|查看HDFS的基本统计信息|
|hadoop dfsadmin –safemode leave|退出安全模式|
|hadoop dfsadmin –safemode enter|进入安全模式|

<br>

当然，还有许多参数，就不在此一一列举了，
例：cat、chown、chgrp、chmod、moveFromLocal、moveToLocal、count、cp、du、df、ls、stat、text、touchz等等

注：setrep命令设置副本数量，记录在元数据中，并不真实。
NameNode在启动时会自动进入安全模式


###**客户端从HDFS中读取数据**

![这里写图片描述](http://img.blog.csdn.net/20170728132001860?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYmFpeWVfeGluZw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

**1.分析一（类+方法名）：**

•	客户端(client)用FileSystem的open()函数打开文件。

•	DistributedFileSystem用RPC调用元数据节点，得到文件的数据块信息。

•	对于每一个数据块，元数据节点返回保存数据块的数据节点的地址。

•	DistributedFileSystem返回FSDataInputStream给客户端，用来读取数据。

•	客户端调用stream的read()函数开始读取数据。

•	DFSInputStream连接保存此文件第一个数据块的最近的数据节点。

•	Data从数据节点读到客户端(client)。

•	当此数据块读取完毕时，DFSInputStream关闭和此数据节点的连接，然后连接此文件下一个数据块的最近的数据节点。

•	当客户端读取完毕数据的时候，调用FSDataInputStream的close函数。

•	在读取数据的过程中，如果客户端在与数据节点通信出现错误，则尝试连接包含此数据块的下一个数据节点。

•	失败的数据节点将被记录，以后不再连接。
　
**2.分析二（纯文本）**

•	使用HDFS提供的客户端开发库，向远程的Namenode发起RPC请求；

•	Namenode会视情况返回文件的部分或者全部block列表，对于每个block，Namenode都会返回有该block拷贝的datanode地址；

•	客户端开发库会选取离客户端最接近的datanode来读取block；

•	读取完当前block的数据后，关闭与当前的datanode连接，并为读取下一个block寻找最佳的datanode；

•	当读完列表的block后，且文件读取还没有结束，客户端开发库会继续向Namenode获取下一批的block列表。

•	读取完一个block都会进行checksum验证，如果读取datanode时出现错误，客户端会通知Namenode，然后再从下一个拥有该block拷贝的datanode继续读。

###**客户端从HDFS中写入数据**

![这里写图片描述](http://img.blog.csdn.net/20170728132027732?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYmFpeWVfeGluZw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

**1.分析一（类+方法名）**

* 客户端调用create()来创建文件

* DistributedFileSystem用RPC调用元数据节点，在文件系统的命名空间中创建一个新的文件。

* 元数据节点首先确定文件原来不存在，并且客户端有创建文件的权限，然后创建新文件。

* DistributedFileSystem返回DFSOutputStream，客户端用于写数据。

* 客户端开始写入数据，DFSOutputStream将数据分成块，写入data queue。

* Data queue由Data Streamer读取，并通知元数据节点分配数据节点，用来存储数据块(每块默认复制3块)。分配的数据节点放在一个pipeline里。

* Data Streamer将数据块写入pipeline中的第一个数据节点。第一个数据节点将数据块发送给第二个数据节点。第二个数据节点将数据发送给第三个数据节点。

* DFSOutputStream为发出去的数据块保存了ack queue，等待pipeline中的数据节点告知数据已经写入成功。

* 如果数据节点在写入的过程中失败：
 
 * 关闭pipeline，将ack queue中的数据块放入data queue的开始。
 * 当前的数据块在已经写入的数据节点中被元数据节点赋予新的标示，则错误节点重启后能够察觉其数据块是过时的，会被删除。
 * 失败的数据节点从pipeline中移除，另外的数据块则写入pipeline中的另外两个数据节点。
 * 元数据节点则被通知此数据块是复制块数不足，将来会再创建第三份备份。

* 当客户端结束写入数据，则调用stream的close函数。此操作将所有的数据块写入pipeline中的数据节点，并等待ack queue返回成功。最后通知元数据节点写入完毕。

**2.分析二（纯文本）**

•	使用HDFS提供的客户端开发库，向远程的Namenode发起RPC请求；

•	Namenode会检查要创建的文件是否已经存在，创建者是否有权限进行操作，成功则会为文件创建一个记录，否则会让客户端抛出异常；

•	当客户端开始写入文件的时候，开发库会将文件切分成多个packets，并在内部 以"data queue"的形式管理这些packets，并向Namenode申请新的blocks，获取用来存储replicas的合适的datanodes列表， 列表的大小根据在Namenode中对replication的设置而定。

•	开始以pipeline（管道）的形式将packet写入所有的replicas 中。开发库把packet以流的方式写入第一个datanode，该datanode把该packet存储之后，再将其传递给在此pipeline中的下 一个datanode，直到最后一个datanode，这种写数据的方式呈流水线的形式。

•	最后一个datanode成功存储之后会返回一个ack packet，在pipeline里传递至客户端，在客户端的开发库内部维护着"ack queue"，成功收到datanode返回的ack packet后会从"ack queue"移除相应的packet。

•	如果传输过程中，有某个datanode出现了故障，那么当前的pipeline 会被关闭，出现故障的datanode会从当前的pipeline中移除，剩余的block会继续剩下的datanode中继续以pipeline的形式 传输，同时Namenode会分配一个新的datanode，保持replicas设定的数量。

注：客户端上传数据时DataNode的选择策略：第一个副本选择离客户端最近的DataNode，第二个副本选择跨机架的一个DataNode，第三个副本选择第一个副本的机架中的另一个DataNode。

<br>
<br>
本人才疏学浅，若有错，请指出，谢谢！ 
如果你有更好的建议，可以留言我们一起讨论，共同进步！ 
衷心的感谢您能耐心的读完本篇博文！

参考链接：[HDFS介绍（图）](http://www.cnblogs.com/chybin/p/5361994.html)