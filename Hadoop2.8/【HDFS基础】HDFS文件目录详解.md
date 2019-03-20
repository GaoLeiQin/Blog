###**HDFS的文件目录图**

![这里写图片描述](http://img.blog.csdn.net/20170728144741006?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYmFpeWVfeGluZw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

分析：从上图可以看出，HDFS的文件目录主要由NameNode、SecondaryNameNode和DataNode组成，而NameNode和DataNode之间由心跳机制通信。

注：

* HDFS(Hadoop Distributed File System)默认的存储单位是128M的数据块。
可以执行命令vim /home/qingaolei/hadoop/hadoop-2.8.0/share/doc/hadoop/hadoop-project-dist/hadoop-hdfs/hdfs-default.xml查看

![这里写图片描述](http://img.blog.csdn.net/20170728151620769?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYmFpeWVfeGluZw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

* 当然也可以通过修改配置文件进行修改，通过命令vim /home/qingaolei/hadoop/hadoop-2.8.0/etc/hadoop/hdfs-site.xml进入到配置文件进行修改。

![这里写图片描述](http://img.blog.csdn.net/20170728152520153?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYmFpeWVfeGluZw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

###**NameNode**

####**1.NameNode的文件结构**

![这里写图片描述](http://img.blog.csdn.net/20170728145447807?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYmFpeWVfeGluZw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)
//中间省略很多行
![这里写图片描述](http://img.blog.csdn.net/20170728145500381?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYmFpeWVfeGluZw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

分析：从上图可以看出，NameNode的文件结构包含edits、fsimage、seen_txid、VERSION

####**2.edits**

* 编辑日志（edit log）：当客户端执行写操作时，首先NameNode会在编辑日志中写下记录，并在内存中保存一个文件系统元数据，这个描述符会在编辑日志改动之后更新。

* edits_start transaction ID-end transaction ID
finalized edit log segments，在HA（高可用）环境中，Standby Namenode只能读取finalized log segments，

* edits_inprogress__start transaction ID
当前正在被追加的edit log，HDFS默认会为该文件提前申请1MB空间以提升性能

####**3.fsimage**

* 文件系统镜像（fsimage）:文件系统元数据的持久检查点，包含以序列化格式（从Hadoop-2.4.0起，FSImage开始采用Google Protobuf编码格式）存储的文件系统目录和文件inodes，每个inodes表征一个文件或目录的元数据信息以及文件的副本数、修改和访问时间等信息。

* fsimage_end transaction ID
每次checkpoing（合并所有edits到一个fsimage的过程）产生的最终的fsimage，同时会生成一个.md5的文件用来对文件做完整性校验（详细过程见下文）。

####**4.seen_txid**

* seen_txid是存放transactionId的文件，format之后是0，它代表的是namenode里面的edits_*文件的尾数，namenode重启的时候，会按照seen_txid的数字，循序从头跑edits_0000001~到seen_txid的数字。

* 当hdfs发生异常重启的时候，一定要比对seen_txid内的数字是不是你edits最后的尾数，不然会发生建置namenode时metaData的资料有缺少，导致误删Datanode上多余Block的资讯。

####**5.VERSION**

VERSION文件是java属性文件，保存了HDFS的版本号。

![这里写图片描述](http://img.blog.csdn.net/20170728145732467?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYmFpeWVfeGluZw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

•	namespaceID是文件系统的唯一标识符，是在文件系统初次格式化时生成的。
•   clusterID是系统生成或手动指定的集群ID
•	cTime表示NameNode存储时间的创建时间，升级后会更新该值。
•	storageType表示此文件夹中保存的是元数据节点的数据结构。
•	blockpoolID：针对每一个Namespace所对应blockpool的ID,该ID包括了其对应的NameNode节点的ip地址。
•  layoutVersion是一个负整数，保存了HDFS的持续化在硬盘上的数据结构的格式版本号。

####**6.in_use.lock**

防止一台机器同时启动多个Namenode进程导致目录数据不一致

###**SecondaryNameNode**

####**1.SecondaryNameNode的文件结构**

![这里写图片描述](http://img.blog.csdn.net/20170728155454981?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYmFpeWVfeGluZw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

分析：从上图可以看出，SecondaryNameNode主要包括edits、fsimage、VERSION

####**2.edits**

从NameNode复制的日志文件

####**3.fsimage**

从NameNode复制的镜像文件

####**4.VERSION**

![这里写图片描述](http://img.blog.csdn.net/20170728155655831?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYmFpeWVfeGluZw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

注：SecondaryNameNode和NameNode的VERSION系相同，不再赘述。

####**5.in_use.lock**

防止一台机器同时启动多个SecondaryNameNode进程导致目录数据不一致

###**check point 过程**

####**1.图例：检查点处理过程**

![这里写图片描述](http://img.blog.csdn.net/20170728160343233?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYmFpeWVfeGluZw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

####**2.过程分析**

1）Secondary NameNode首先请求原NameNode进行edits的滚动，这样新的编辑操作就能够进入新的文件中。

2）Secondary NameNode通过HTTP方式读取原NameNode中的fsimage及edits。

3）Secondary NameNode读取fsimage到内存中，然后执行edits中的每个操作，并创建一个新的统一的fsimage文件。

4）Secondary NameNode通过HTTP方式将新的fsimage发送到原NameNode。

5）原NameNode用新的fsimage替换旧的fsimage，旧的edits文件用步骤1）中的edits进行替换（将edits.new改名为edits）。同时系统会更新fsimage文件到记录检查点的时间。
这个过程结束后，NameNode就有了最新的fsimage文件和更小的edits文件


注：可执行hadoop dfsadmin –saveNamespace命令运行上图的过程Secondary NameNode（NameNode的冷备份）每隔一小时会插入一个检查点，如果编辑日志达到64MB，则间隔时间更短，每隔5分钟检查一次。


###**DataNode**

####**1.DataNode的文件结构**

![这里写图片描述](http://img.blog.csdn.net/20170728152902313?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYmFpeWVfeGluZw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

分析：从上图可以看出，.DataNode的文件结构主要由blk_前缀文件、BP-random integer-NameNode-IP address-creation time和VERSION构成。

####**2.BP-random integer-NameNode-IP address-creation time**

* BP代表BlockPool的，就是Namenode的VERSION中的集群唯一blockpoolID，

* 从上图可以看出我的DataNode是两个BP，这是因为我的HDFS是Federation HDFS，所以该目录下有两个BP开头的目录，IP部分和时间戳代表创建该BP的NameNode的IP地址和创建时间戳

####**3.finalized/rbw**

* 这两个目录都是用于实际存储HDFS BLOCK的数据，里面包含许多block_xx文件以及相应的.meta文件，.meta文件包含了checksum信息。

* rbw是“replica being written”的意思，该目录用于存储用户当前正在写入的数据。

####**4.blk_前缀文件**

* HDFS中的文件块本身，存储的是原始文件内容。

* 块的元数据信息（使用.meta后缀标识）。一个文件块由存储的原始文件字节组成，元数据文件由一个包含版本和类型信息的头文件和一系列块的区域校验和组成。

注：当目录中存储的块数据量增加到一定规模时，DataNode会创建一个新的目录，用于保存新的块及元数据。当目录中的块数据量达到64（可由dfs.DataNode.numblocks属性确定）时，便会新建一个子目录，这样就会形成一个更宽的文件树结构，避免了由于存储大量数据块而导致目录很深，使检索性能免受影响。通过这样的措施，数据节点可以确保每个目录中的文件块都可控的，也避免了一个目录中存在过多文件。

####**5.VERSION**

![这里写图片描述](http://img.blog.csdn.net/20170728153339586?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYmFpeWVfeGluZw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

•	storageID相对于DataNode来说是唯一的，用于在NameNode处标识DataNode
•	 clusterID是系统生成或手动指定的集群ID
•	cTime表示NameNode存储时间的创建时间
•	datanodeUuid表示DataNode的ID号
•	storageType将这个目录标志位DataNode数据存储目录。
•  layoutVersion是一个负整数，保存了HDFS的持续化在硬盘上的数据结构的格式版本号。

####**6.in_use.lock**

防止一台机器同时启动多个Datanode进程导致目录数据不一致

<br>
<br>
本人才疏学浅，若有错，请指出，谢谢！ 
如果你有更好的建议，可以留言我们一起讨论，共同进步！ 
衷心的感谢您能耐心的读完本篇博文！

参考链接：[ Secondary NameNode:它究竟有什么作用？](http://blog.csdn.net/xh16319/article/details/31375197)