# Hadoop集群中NameNode单点故障的解决方案对比

Hadoop的设计理念是被安装在廉价物理机节点上的高可用性分布式系统，但是HDFS中NameNode的单点故障却与此存在矛盾。

解决单点故障问题可以从以下几个方向入手：

### 使用公共缓存

- 原理：所有服务器节点都将客户端的任务信息写入缓存。
- 优点：实现简单。
- 缺点：公共缓存成为单点。

### 服务端内存共享

- 原理：服务端之间实现内存共享，各自保存客户端的实例，但是模板实例不共享，模板本身不存在于多个服务节点。
- 优点：服务端之间相对独立，单节点故障不影响服务。
- 缺点：整体服务的吞吐有一定限制，服务水平只是比单节点的容量稍微高一点点。

### 服务端各自独立

- 原理：服务端各自独立，提供一个服务寻址算法(类似Hash分段)，客户端实现算法搜寻服务。 
- 优点：分布式的服务的基本实现思路了 
- 缺点：实现比较复杂，开发成本过高

那这个问题有着怎样的解决方案呢？

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

- 原理：同时启动2个NameNode。其中一个处于工作状态(Active)，另一个处于随时待命状态(Standby)。当一个Active NameNode所在的服务器宕机时，可以在数据不丢失的情况下，手工或者自动将另一个Standby NameNode切换到Active 并继续提供服务。 
- 优点：信息不会丢失，恢复时间短，部署简单。

**JournalNode**

1. 为了使备用节点（Standby NameNode）将其状态与Active节点保持同步，两个节点与一组名为“JournalNode”（JN）的单独守护程序进行通信。 
2. 当Active NameNode执行任何命名空间修改时，会把最近的操作记录写到本地的一个edits文件中（edits file），并传输到大部分JournalNode（写入2n+1个journalnode上，只要有n+1个写入成功就认为这次写入操作成功了）。 
3. Standby NameNode定期的检查，从JournalNode把最近的edit文件读过来，然后把edits文件和fsimage文件合并成一个新的fsimage。Standby NameNode可以确保在集群出错时，NameNode命名空间状态已经完全同步了。 
4. Standby NameNode合并生成新的fsimage后会通知Active NameNode获取这个新fsimage。Active NameNode获得这个新的fsimage文件之后，替换原来旧的fsimage文件。 

**主备节点的切换**

- 为了提供快速故障切换，还需要备用节点具有关于集群中块的位置的最新信息。为了实现这一点，DataNodes被配置为具有两个NameNodes的位置，并且向两者发送块位置信息和心跳。 
- 人工切换是通过执行HA管理的命令来改变namenode的状态，从standby到active，或者从active到standby。自动切换则在active namenode挂掉的时候，standby namenode自动切换成active状态，取代原来的active namenode成为新的active namenode，HDFS继续正常工作。
- 对于HA群集的正确操作至关重要，因此一次只能有一个NameNodes处于活动状态。否则，命名空间状态将在两者之间迅速分歧，冒数据丢失或其他不正确的结果。为了确保这个属性并防止所谓的“脑裂”，JournalNodes将只允许一个NameNode作为一个作者向JournalNodes写数据。在故障切换期间，要变为活动状态的NameNode将简单地接管写入JournalNodes的角色，这将有效地防止其他NameNode继续处于活动状态，允许新的Active安全地进行故障切换。
- 主备节点的自动切换需要配置zookeeper。active namenode和standby namenode把他们的状态实时记录到zookeeper中，zookeeper监视他们的状态变化。当zookeeper发现active namenode挂掉后，会自动把standby namenode切换成active namenode。
