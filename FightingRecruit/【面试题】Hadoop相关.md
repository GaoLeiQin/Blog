####**1.HDFS的特点？**

* Hadoop是一套开源的软件平台，利用服务器集群，根据用户的自定义业务逻辑，对海量数据进行分布式处理，核心组件分为：HDFS（分布式文件系统）、MapRuduce（分布式运算编程框架）、YARN（运算资源调度系统）

* HDFS是Hadoop体系中数据存储管理的基础。它是一个高度容错的系统，能检测和应对硬件故障，用于在低成本的通用硬件上运行。HDFS简化了文件的一致性模型，通过流式数据访问，提供高吞吐量应用程序数据访问功能，具有高容错、高可靠性、高可扩展性、高获得性、高吞吐率等等，适合带有大型数据集的应用程序

* HDFS采用主从结构模型，一个HDFS集群是由一个NameNode和若干个DataNode组成的。NameNode管理文件系统的命名空间，维护文件系统树及整棵树内所有文件和目录，即命名空间镜像文件和编辑日志文件。SecondaryNameNode的任务是定期合并fsimage和editlog，并传输给NameNode，为NameNode提供冷备份。DataNode 是文件系统的工作节点，根据需要存储并检索数据块（客户端或NameNode调度），定期向NameNode发送它们所存储块的列表。

* Client通过同NameNode和DataNodes的交互访问文件系统。客户端联系NameNode以获取文件的元数据，而真正的文件I/O操作是直接和DataNode进行交互的。

* HDFS的优点：适合存储大数据文件、文件分块存储、流式数据访问：一次写入多次读写、廉价硬件、硬件故障 、即Hadoop可以流式访问大文件，部署在廉价的集群上。缺点：不适合低延迟数据访问（Hbase）、无法高效存储大量小文件（归档Hbase）、不支持多用户写入及任意修改文件。

* 常用命令：hadoop fs –put/get/lsr/mkdir/rmr等，

####**2.客户端从HDFS中读写数据过程？**

* 读取数据

• 客户端(client)用FileSystem的open()函数打开文件。

• DistributedFileSystem用RPC调用元数据节点，得到文件的数据块信息。

• 对于每一个数据块，元数据节点返回保存数据块的数据节点的地址。

• DistributedFileSystem返回FSDataInputStream给客户端，用来读取数据。

• 客户端调用stream的read()函数开始读取数据。

• DFSInputStream连接保存此文件第一个数据块的最近的数据节点。

• Data从数据节点读到客户端(client)。

• 当此数据块读取完毕时，DFSInputStream关闭和此数据节点的连接，然后连接此文件下一个数据块的最近的数据节点。

• 当客户端读取完毕数据的时候，调用FSDataInputStream的close函数。

• 在读取数据的过程中，如果客户端在与数据节点通信出现错误，则尝试连接包含此数据块的下一个数据节点。

• 失败的数据节点将被记录，以后不再连接。

* 写入数据

• 客户端调用create()来创建文件

• DistributedFileSystem用RPC调用元数据节点，在文件系统的命名空间中创建一个新的文件。

• 元数据节点首先确定文件原来不存在，并且客户端有创建文件的权限，然后创建新文件。

• DistributedFileSystem返回DFSOutputStream，客户端用于写数据。

• 客户端开始写入数据，DFSOutputStream将数据分成块，写入data queue。

• Data queue由Data Streamer读取，并通知元数据节点分配数据节点，用来存储数据块(每块默认复制3块)。分配的数据节点放在一个pipeline里。

• Data Streamer将数据块写入pipeline中的第一个数据节点。第一个数据节点将数据块发送给第二个数据节点。第二个数据节点将数据发送给第三个数据节点。

• DFSOutputStream为发出去的数据块保存了ack queue，等待pipeline中的数据节点告知数据已经写入成功。

• 如果数据节点在写入的过程中失败：

 * 关闭pipeline，将ack queue中的数据块放入data queue的开始。
 * 当前的数据块在已经写入的数据节点中被元数据节点赋予新的标示，则错误节点重启后能够察觉其数据块是过时的，会被删除。
 * 失败的数据节点从pipeline中移除，另外的数据块则写入pipeline中的另外两个数据节点。
 * 元数据节点则被通知此数据块是复制块数不足，将来会再创建第三份备份。

• 当客户端结束写入数据，则调用stream的close函数。此操作将所有的数据块写入pipeline中的数据节点，并等待ack queue返回成功。最后通知元数据节点写入完毕。

####**3.HDFS的文件目录结构？**

* HDFS的文件目录主要由NameNode、SecondaryNameNode和DataNode组成，而NameNode和DataNode之间由心跳机制通信。默认的存储单位是128M的数据块

* NameNode的文件结构包含edits、fsimage、seen_txid、VERSION
 * edits：当客户端执行写操作时，首先NameNode会在编辑日志中写下记录，并在内存中保存一个文件系统元数据，这个描述符会在编辑日志改动之后更新。
 * fsimage:文件系统元数据的持久检查点，包含以protobuf序列化格式存储的文件系统目录和文件inodes，每个inodes表征一个文件或目录的元数据信息以及文件的副本数、修改和访问时间等信息。
 * seen_txid是存放transactionId的文件，代表namenode的edits_*文件尾数
 * VERSION，保存了HDFS的版本号：namespaceID、clusterID、blockpoolID、cTimestorageType等
 * in_use.lock：防止一台机器同时启动多个Namenode进程导致目录数据不一致

* SecondaryNameNode主要包括edits、fsimage、VERSION
 * .edits：从NameNode复制的日志文件
 * fsimage：从NameNode复制的镜像文件
 * VERSION：与NameNode的VERSION相同

* CheckPoint过程：Secondary NameNode首先请求原NameNode进行edits的滚动，这样新的编辑操作就能够进入新的文件中。通过HTTP方式读取原NameNode中的fsimage及edits。然后将fsimage读取到内存中，然后执行edits中的每个操作，并创建一个新的统一的fsimage.chpt文件。通过HTTP方式将新的fsimage发送到原NameNode。原NameNode用新的fsimage替换旧的fsimage，旧的edits文件用第一步的edits进行替换。同时系统会更新fsimage文件到记录检查点的时间。 最后，NameNode就有了最新的fsimage文件和更小的edits文件

* 可执行hadoop dfsadmin –saveNamespace命令运行上图的过程Secondary NameNode（NameNode的冷备份）每隔一小时会插入一个检查点，如果编辑日志达到64MB，则间隔时间更短，每隔5分钟检查一次。

* DataNode的文件结构主要由blk_前缀文件、BP-number和VERSION构成。
 * BP代表NameNode的VERSION中的BlockPoolID，number表示该BP的NameNode的IP地址和创建时间戳
 * finalized：存储HDFS BLOCK的数据，里面包含许多block_xx文件以及相应的.meta文件，.meta文件包含了checksum信息。rbw目录用于存储用户当前正在写入的数据。
 * blk_前缀文件：存储的是原始文件内容。若目录中的块数据量达到64，便会新建一个子目录
 * VERSION： storageID（在NN标识DN）、clusterID、cTime、datanodeUuid、layoutVersion
 
#### **4.NameNode的内存结构？**

* NameNode内存主要由Namespace+BlocksMap（50%）NetworkTopology及其它部分组成

* Namespace：维护整个文件系统的目录树结构及目录树上的状态变化；目录树中存在INodeDirectory和INodeFile两种INode数据结构，

* BlockManager：维护整个文件系统中与数据块相关的信息及数据块的状态变化；管理BlocksMap，BlockMap的核心功能就是通过BlockID快速定位到具体的BlockInfo，BlockInfo维护的是Block的元数据。集群启动过程，DataNode会进行BR（BlockReport），根据BR的每一个Block计算其HashCode，之后将对应的BlockInfo插入到相应位置逐渐构建起来巨大的BlocksMap。

* NetworkTopology：维护机架拓扑及DataNode信息，机架感知的基础；DataNode节点信息的每一个存储单元DatanodeStorageInfo上的所有Blocks集合会形成一个双向链表，链表的入口就是机架拓扑结构叶子节点

* 其它：LeaseManager：读写的互斥同步就是靠Lease实现；CacheManager：支持集中式缓存的管理；SnapshotManager：用于数据备份、回滚。

####**5.NameNode的重启优化？**

* NameNode内存占用增长出现的问题：启动时间变长、性能下降、NameNode Full GC风险较高（CMS）、超大JVM Heap Size调试问题

* 解决方法：扩展NameNode分散单点负载（SNN或SBN）、引入外部系统支持NameNode内存数据、合并小文件、调整合适的BlockSize（默认128M）、将原生NameNode管理的Namespace和BlockManagement进行物理拆分、借助高速存储设备，将元数据通过外存设备进行持久化存储。

* NameNode重启流程：加载FSImage、回放EditLog、执行CheckPoint（可选）、DataNode重新注册RegisterDataNode，然后汇报所有数据块BlockReport。

* 解决方法：重启过程尽可能避免出现CheckPoint、优化BlockReport处理逻辑（若汇报的数据块相关元数据还没有加载，先暂存消息队列，当NameNode完成加载相关元数据后，再处理该消息队列）、防止SBN或SNN长时间未正常运行堆积大量Editlog拖慢NameNode重启时间 、调整Block数量阈值，将一次BlockReport分成多盘分别汇报。

####**6.Git的使用？**

* 常用命令：git init 初始化版本库、git add/rm suger.md 添加/删除文件到暂存区、git commit -m "first" 将文件从暂存区提交到仓库中、git branch列出所有本地分支、git checkout  slave 切换到slave分支、git merge slave 合并指定分支到当前分支、git branch -d slave 删除slave分支、git tag 列出所有标签、git status 显示有变更的文件、git log 显示当前分支的版本信息、git diff 显示工作区与暂存区的差异、git remote add [shortname] [url] 增加一个新的远程仓库，并命名、git pull [remote] [branch]取回远程仓库的变化，并与本地分支合并、git push [remote] --force 强行推送当前分支到远程仓库、git checkout suger.md 恢复暂存区的指定文件到工作区、git reset --hard 重置暂存区与工作区，与上一次commit保持一致、git fetch [remote] 下载远程仓库的所有变动


####**7.Maven的使用**

* maven是一个项目构建和管理的工具，将项目过程规范化、自动化、高效化以及强大的可扩展性，利用maven自身及其插件还可以获得代码检查报告、单元测试覆盖率、实现持续集成等等，依赖管理是它的特点。

* 坐标：groupId: 组织ID，一般是公司、团体名称，artifactId：实际项目的ID，一般是项目、模块名称，version:版本，开发中的版本一般打上 SNAPSHOT 标记（快照）

* 三级仓库结构：远程公用仓库、内部中央仓库(私服)、本地仓库（Maven 会将工程中依赖的构件(Jar包)从远程下载到本机一个目录下管理）。

* maven的生命周期：clean：在真正的构建之前进行一些清理工作；default：构件项目的核心部分，validate、compile、test、package、install、deploy等；site：生成项目报告、站点、发布站点。

<br>
<br>
本人才疏学浅，若有错，请指出，谢谢！ 
如果你有更好的建议，可以留言我们一起讨论，共同进步！ 
衷心的感谢您能耐心的读完本篇博文！
