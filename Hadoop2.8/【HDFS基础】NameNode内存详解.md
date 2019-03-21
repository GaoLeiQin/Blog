###**定义**

* NameNode管理着整个HDFS文件系统的元数据。

* 从架构设计上看，元数据大致分成两个层次：
 
 * Namespace管理层：负责管理文件系统中的树状目录结构以及文件与数据块的映射关系；
 * 块管理层：负责管理文件系统中文件的物理块与实际存储位置的映射关系BlocksMap。

* Namespace管理的元数据除内存常驻外，也会周期Flush到持久化设备上FsImage文件；BlocksMap元数据只在内存中存在；当NameNode发生重启，首先从持久化设备中读取FsImage构建Namespace，之后根据DataNode的汇报信息重新构造BlocksMap。这两部分数据结构是占据了NameNode大部分JVM Heap空间。

![这里写图片描述](http://img.blog.csdn.net/20170728170753483?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYmFpeWVfeGluZw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)


###**NameNode内存的组成**

NameNode内存主要由Namespace、BlocksMap、NetworkTopology及其它部分组成

* Namespace：维护整个文件系统的目录树结构及目录树上的状态变化；
* BlockManager：维护整个文件系统中与数据块相关的信息及数据块的状态变化；
* NetworkTopology：维护机架拓扑及DataNode信息，机架感知的基础；
* 其它：
 * LeaseManager：读写的互斥同步就是靠Lease实现，支持HDFS的Write-Once-Read-Many的核心数据结构；
 * CacheManager：Hadoop 2.3.0引入的集中式缓存新特性，支持集中式缓存的管理，实现memory-locality提升读性能；
 * SnapshotManager：Hadoop 2.1.0引入的Snapshot新特性，用于数据备份、回滚，以防止因用户误操作导致集群出现数据问题；
 * DelegationTokenSecretManager：管理HDFS的安全访问；
另外还有临时数据信息、统计信息metrics等等。

注：NameNode常驻内存主要被Namespace和BlockManager使用，二者使用占比分别接近50%。其它部分内存开销较小且相对固定，与Namespace和BlockManager相比基本可以忽略。


###**各部分内存的解析**

####**1.Namespace**

* HDFS对文件系统的目录结构按照树状结构维护，Namespace保存了目录树及每个目录/文件节点的属性。除在内存常驻外，这部分数据会定期flush到持久化设备上，生成一个新的FsImage文件，方便NameNode发生重启时，从FsImage及时恢复整个Namespace。

![这里写图片描述](http://img.blog.csdn.net/20170728171313532?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYmFpeWVfeGluZw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)


* Namespace目录树中存在两种INode数据结构：INodeDirectory和INodeFile。
 * INodeDirectory标识的是目录树中的目录，
 * INodeFile标识的是目录树中的文件。

**INode继承关系图解**
![这里写图片描述](http://img.blog.csdn.net/20170728171623055?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYmFpeWVfeGluZw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)


####**2.BlockManager**

* BlockManager用于管理BlocksMap。Namespace与BlockManager之间通过INodeFile有序Blocks数组关联。

* BlockManager管理的内存结构
![这里写图片描述](http://img.blog.csdn.net/20170728172120557?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYmFpeWVfeGluZw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

* 每一个INodeFile都会包含数量不等的Block，具体数量由文件大小及每一个Block大小（默认为64M）比值决定，这些Block按照所在文件的先后顺序组成BlockInfo数组，

* BlockInfo维护的是Block的元数据，数据本身是由DataNode管理，所以BlockInfo需要包含实际数据到底由哪些DataNode管理的信息，这里的核心是名为triplets的Object数组，大小为3*replicas，其中replicas是Block副本数量。

* 但不能通过blockid快速定位Block，所以引入了BlocksMap，而BlockMap的核心功能就是通过BlockID快速定位到具体的BlockInfo。 

* BlocksMap底层通过LightWeightGSet实现，本质是一个链式解决冲突的哈希表。为了避免rehash过程带来的性能开销，初始化时，索引空间直接给到了整个JVM可用内存的2%，并且不再变化。

* 集群启动过程，DataNode会进行BR（BlockReport），根据BR的每一个Block计算其HashCode，之后将对应的BlockInfo插入到相应位置逐渐构建起来巨大的BlocksMap。

####**3.NetworkTopology**

* NameNode需要管理所有DataNode，由于数据写入前需要确定数据块写入位置，NameNode还维护着整个机架拓扑NetworkTopology。

* 内存中机架拓扑图
![这里写图片描述](http://img.blog.csdn.net/20170728173038740?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYmFpeWVfeGluZw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

* Networktopology由机架拓扑结构NetworkTopology和DataNode节点信息组成。

* 树状的机架拓扑是根据机架感知（一般都是外部脚本计算得到）在集群启动完成后建立起来，整个机架的拓扑结构在NameNode的生命周期内一般不会发生变化

* DataNode节点信息，DataNode的每一个存储单元DatanodeStorageInfo上的所有Blocks集合会形成一个双向链表，这个链表的入口就是机架拓扑结构叶子节点即DataNode管理的DatanodeStorageInfo。

* DatanodeStorageInfo下所有Block之间会以双向链表组织，因为对每一个DatanodeStorageInfo下Block的操作集中在快速增加/删除（Block动态增减变化）及顺序遍历（BlockReport期间），所以双向链表是非常合适的数据结构。

####**4.LeaseManager**

* Lease 机制是重要的分布式协议，广泛应用于各种实际的分布式系统中。

* HDFS支持Write-Once-Read-Many，对文件写操作的互斥同步靠Lease实现。

* Lease实际上是时间约束锁，其主要特点是排他性。客户端写文件时需要先申请一个Lease，一旦有客户端持有了某个文件的Lease，其它客户端就不可能再申请到该文件的Lease，这就保证了同一时刻对一个文件的写操作只能发生在一个客户端。

* NameNode的LeaseManager是Lease机制的核心，维护了文件与Lease、客户端与Lease的对应关系，这类信息会随写数据的变化实时发生对应改变。

###**出现的问题**

随着集群中数据规模的不断积累，NameNode内存占用随之成比例增长。不可避免的NameNode内存将逐渐成为集群发展的瓶颈，并开始暴漏诸多问题。

####**1、启动时间变长**

NameNode的启动过程可以分成FsImage数据加载、editlogs回放、Checkpoint、DataNode的BlockReport几个阶段。数据规模较小时，启动时间可以控制在10min以内，当元数据规模很大时，非常耗时。

####**2、性能下降**

当数据规模的增加致内存占用变大后，元数据的增删改查性能会出现下降。

####**3、NameNode JVM FGC（Full GC）风险较高**

* FGC频率增加

* FGC时间增加且风险不可控。

* 对NameNode内存的回收使用CMS内存回收算法

####**4、超大JVM Heap Size调试问题**

如果线上集群性能表现变差，不得不通过分析内存才能得到结论时，Dump本身极其费时费力，Dump超大内存时存在极大概率使NameNode不可服务。

###**解决方法**

####**1.扩展NameNode分散单点负载**

对NameNode进行水平扩展分散单点负载的方式解决NameNode的问题，例如HDFS Federation方案，可以达到良好的隔离性，不会因为单一应用压垮整集群。

####**2.引入外部系统支持NameNode内存数据**

将Namespace存储值外部的KV存储系统如LevelDB[4]，从而降低NameNode内存负载

####**3.合并小文件**

因为目录/文件和Block均会占用NameNode内存空间，大量小文件会降低内存使用效率，小文件的读写性能远远低于大文件的读写，对小文件读写需要在多个数据源切换，严重影响性能。

####**4.调整合适的BlockSize**

针对集群内文件较大的业务场景，可以通过调整默认的Block Size大小（参数：dfs.blocksize，默认128M），降低NameNode的内存增长趋势。

####**5.公司的解决方案**

* 百度
将元数据管理通过主从架构的集群形式提供服务，本质上是将原生NameNode管理的Namespace和BlockManagement进行物理拆分。其中Namespace负责管理整个文件系统的目录树及文件到BlockID集合的映射关系，BlockID到DataNode的映射关系是按照一定的规则分到多个服务节点分布式管理，这种方案与Lustre有相似之处（Hash-based Partition）。

* 淘宝
借助高速存储设备，将元数据通过外存设备进行持久化存储，保持NameNode完全无状态，实现NameNode无限扩展的可能。

<br>
<br>
本人才疏学浅，若有错，请指出
谢谢！

参考链接：[HDFS NameNode内存详解——美团点评技术博客](https://tech.meituan.com/namenode-memory-detail.html)