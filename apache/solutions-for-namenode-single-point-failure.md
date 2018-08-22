# Hadoop集群中NameNode单点故障的解决方案对比

Hadoop的设计理念是被安装在廉价物理机节点上的高可用性分布式系统，但是HDFS中NameNode的单点故障却与此存在矛盾。那这个问题有着怎样的解决方案呢？

### Secondary NameNode

- 原理：文件系统客户端在执行写操作时，这些操作首先被记录到编辑日志中，然后更新NameNode内存中维护的文件系统的元数据；命名空间镜像文件（fsimage）是文件系统元数据的一个永久检查点，因为fsimage文件是一个大型文件，如果频繁的执行写操作，会使系统运行极为缓慢。但是如果不进行操作，editlog文件会无限增长，一旦NameNode需要恢复，则需要花费非常长的时间，所以HDFS引入了辅助NameNode，为主NameNode内存中的文件系统元数据创建检查点。实现将主NameNode的编辑日志和命名空间镜像文件合并。形成一个更小的editlog文件和最新的fsimage文件。客户端执行写操作的时候，首先更新编辑日志（edits），然后修改内存中的文件系统映射。随着时间的推移，编辑日志会非常大。这个时候，就需要将编辑日志和文件系统映射持久检查点（fsimage）进行合并，以减少编辑日志的大小。这个工作就是由辅助NameNode完成的。
- 优点：Hadoop早期版本中内置，配置相对简单，基本不需要额外资源。
- 缺点：从Secondary NameNode中恢复是一个非常耗时的过程，并且会有部分数据丢失。

### Backup NameNode

- 原理：Backup NameNodes实时得到editlog，当NameNode宕机后，手动切换到Backup NameNode。
- 优点：Hadoop从0.2.1版本开始提供此解决方案，恢复不会有数据的丢失。
- 缺点：因为需要从DataNode中得到block和location信息，在切换到Backup NameNode的时候比较慢。

### Avatar NameNode

- 原理：将client访问hadoop的editlog放在NFS中，Standby NN能够实时拿到editlog；DataNode需要同时与Active NN和Standby NN report block信息；
- 优点：信息不会丢失，恢复时间短。
- 缺点：部署麻烦，需要额外的机器资源，NFS虽然故障率低，但依然又成为了一个单点。

### Standby NameNode

- 原理：Hadoop2.0在Avatar NameNode上做了改进，直接支持Standby NameNode。
- 优点：信息不会丢失，恢复时间短，部署简单。
