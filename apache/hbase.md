# Hadoop数据库：HBase

## 概述

### NoSQL

NoSQL(NoSQL = Not Only SQL)，意即“不仅仅是SQL”。NoSQL数据库的四大分类：

- 键值数据库：键值数据库就像在传统语言中使用的哈希表。你可以通过key来添加、查询或者删除数据，鉴于使用主键访问，所以会获得不错的性能及扩展性。代表产品Redis。
- 列数据库：列存储数据库将数据储存在列族（column family）中，一个列族存储经常被一起查询的相关数据。代表产品HBase。
- 文档型数据库：面向文档数据库会将数据以文档的形式储存。每个文档都是自包含的数据单元，是一系列数据项的集合。代表产品MongoDB。
- 图数据库：图数据库允许我们将数据以图的方式储存。实体会被作为顶点，而实体之间的关系则会被作为边。代表产品Neo4J。

### 列数据库

数据库是以列相关存储架构进行数据存储的数据库，主要适合于批量数据处理和即时查询。相对应的是行式数据库，数据以行相关的存储体系架构进行空间分配，主要适合于大批量的数据处理，常用于联机事务型数据处理。行式存储下一张表的数据都是放在一起的，但列式存储下都被分开保存了。

| 优缺点 | 行式存储                                                  | 列式存储                                                                   |
| ------ | --------------------------------------------------------- | -------------------------------------------------------------------------- |
| 优点   | 数据被保存在一起，INSERT/UPDATE容易。                     | 查询时只有涉及到的列会被读取；投影(projection)很高效；任何列都能作为索引。 |
| 缺点   | 选择(Selection)时即使只涉及某几列，所有数据也都会被读取。 | 选择完成时，被选择的列要重新组装。INSERT/UPDATE比较麻烦。                  |

## 概念

### 简介

- 利用Hadoop HDFS作为其底层存储系统，提供高可靠性、高吞吐、列存储、可伸缩、实时读写的数据库系统。
- 利用Hadoop MapReduce来处理HBase中的海量数据。
- 利用Zookeeper作为协同服务。

### 特色

- 大：一个表可以有上亿行，上百万列。
- 面向列存储：面向列表（簇）的存储和权限控制，列（簇）独立检索。
- 稀疏：对于为空（NULL）的列，并不占用存储空间，因此，表可以设计的非常稀疏。（对于关系数据库，空值位置必须存储NULL值；然而对于HBase，对于空值直接省略该列，也就是说空值不占用任何存储空间）
- 无模式：每一行都有一个可以排序的主键和任意多的列，列可以根据需要动态增加，同一张表中不同的行可以有截然不同的列。
- 数据多版本：每个单元中的数据可以有多个版本，默认情况下，版本号自动分配，版本号就是单元格插入时的时间戳。
- 数据类型单一：HBase中的数据都是字符串，没有类型。

### 特点

- 强一致性：同一行数据的读写只能同时在一台Region Server上进行。
- 水平伸缩：只用增加Data node机器即可增加容量；只用增加Region Server机器即可增加读写吞吐量。
- 行事务：同一行数据的读写是原子操作的。
- 按列存储：数据按列存储，三维有序。
- 支持有限查询方式和一级索引：获得一行的所有数据；获得某行，某列族的所有数据；获得某行，某列族，某列的所有数据。
- 高性能随机读写。
- 与MapReduce无缝连接：MapReduce计算后的结果可直接写入HBase；存放在HBase的数据可直接通过MapReduce来进行分析。

### 基本概念

如果用编程语言风格来表示HBase数据存储模式，可以这样表示：`Map<RowKey, List<Map<Column, List<Value, Timestamp>>>>`。

**RowKey**

Byte Array，是表中每条记录的“主键”，方便快速查找，Rowkey的设计非常重要。与其他NoSQL数据库们一样，RowKey是用来检索记录的主键。

访问HBase Table中的行，只有三种方式：

1. 通过单个RowKey访问
2. 通过RowKey的range
3. 全表扫描

表中行的键是字节数组，最大长度64kb。

任何字符串都可以作为键。

表中的行根据行的键值进行排序，数据按照RowKey的字典序(byte order)排序存储。

所有对表的访问都要通过键。

**Column Family**

列族，拥有一个名称(string)，包含一个或者多个相关列。HBase每个列族存储为一个Store。

HBase表中的每个列都归属于某个列族，列族必须作为表模式(schema)定义的一部分预先定义。如 create 'info', 'realtime'。

列名以列族作为前缀，每个“列族”都可以有多个列成员(column)。如info:name, realtime:price。

新的列族成员可以随后按需、动态加入。

权限控制、存储以及调优都是在列族层面进行的。

HBase把同一列族里面的数据存储在同一目录下，由几个文件保存。

**Column**

属于某一个columnfamily。通过列族:单元格修饰符访问，可以具体到某个列。可以把单元格修饰符认为是实际的列名。

只要有列族存在，客户端随时可以把列添加到列族。

**Version Number**

类型为Long（64位整型），默认值是系统时间戳，可由用户自定义。

在HBase每个cell存储单元对同一份数据有多个版本，根据唯一的时间戳来区分每个版本之间的差异，不同版本的数据按照时间倒序排序，最新的数据版本排在最前面。

时间戳可以由HBase(在数据写入时自动)赋值，此时时间戳是精确到毫秒的当前系统时间。

时间戳也可以由客户显式赋值，如果应用程序要避免数据版本冲突，就必须自己生成具有唯一性的时间戳。

**Value(Cell)**

由行和列的坐标交叉决定。单元格是有版本的。单元格的内容是未解析的字节数组。由`{RowKey, column(=<family>+<label>), version}` 唯一确定的单元。cell中的数据是没有类型的，全部是字节码(Byte Array)形式存储。

**Region**

HBase自动把表水平划分成多个区域(region)，每个region会保存一个表里面某段连续的数据。

每个表一开始只有一个region，随着数据不断插入表，region不断增大，当增大到一个阀值的时候，region就会等分会两个新的region。

当table中的行不断增多，就会有越来越多的region。这样一张完整的表被保存在多个Region 上。

Region是Hbase中“分布式存储”和“负载均衡”的最小单元。最小单元就表示不同的region可以分布在不同的Region server上。但一个Region是不会拆分到多个server上的。

**锁**

HBase的写操作是锁行的，每一行都是一个原子元素，无论对行进行访问的事务涉及多少列，对行的更新都是原子的，也就是说要么成功要么失败不会存在成功一部分的情况。

### 数据模型

HBase以表的形式存储数据。表由行和列组成。行划分为若干个列族。

最基本的单位是列（column），一列或者多列组成一行（row），并且由唯一的行键（row key）来确定存储。一个表中有很多行，每一列可能有多个版本，在每一个单元格（Cell）中存储了不同的值。

HBase的行与行之间是有序的，按照row key的字典序进行排序，行键是唯一的，在一个表里只出现一次，否则就是在更新同一行，行键可以是任意的字节数组。一行由若干列组成，其中的某些列又可以构成一个列族（column family），一个列族的所有列存储在同一个底层的存储文件里，这个文件称之为HFile。

列族需要在创建表的时候就定义好，数量也不宜过多。列族名必须由可打印字符组成，创建表的时候不需要定义好列。对列的引用格式通常为family：qualifier，qualifier也可以是任意的字节数组。同一个列族里qualifier的名称应该唯一，否则就是在更新同一列，列的数量没有限制，可以有数百万个。列值也没有类型和长度限定。HBase会对row key的长度做检查，默认应该小于65536。

HBase的存取模式如下（表，行键，列族，列，时间戳）-> 值。即一个表中的某一行键的某一列族的某一列的某一个版本的值唯一。

行数据的存取操作是原子的，可以读取任意数目的列。目前还不支持跨行事务和跨表事务。

同一列族下的数据压缩在一起，访问控制磁盘和内存都在列族层面进行。

### 架构

HBase包含3个重要组件：Zookeeper、HMaster和HRegionServer。

**Zookeeper**

Zookeeper为整个HBase提供协助服务，包括：

1. 存放整个HBase集群的元数据以及集群的状态信息。
2. 实现HMaster主从节点的failover。

ZooKeeper为HBase集群提供协调服务，它管理着HMaster和HRegionServer的状态(available/alive等)，并且会在它们宕机时通知给HMaster，从而HMaster可以实现HMaster之间的failover，或对宕机的HRegionServer中的HRegion集合的修复(将它们分配给其他的HRegionServer)。ZooKeeper集群本身使用一致性协议(PAXOS协议)保证每个节点状态的一致性。

**HMaster**

HMaster主要用于监控和操作集群中的所有HRegionServer。HMaster没有单点问题，HBase中可以启动多个HMaster，通过Zookeeper的MasterElection机制保证总有一个Master在运行。

管理所有的HRegionServer，告诉其需要维护哪些HRegion，并监控所有HRegionServer的运行状态。当一个新的HRegionServer登录到HMaster时，HMaster会告诉它等待分配数据；而当某个HRegion死机时，HMaster会把它负责的所有HRegion标记为未分配，然后再把它们分配到其他HRegionServer中。HMaster没有单点问题，HBase可以启动多个HMaster，通过Zookeeper的选举机制保证集群中总有一个HMaster运行，从而提高了集群的可用性。

HMaster主要负责Table和Region的管理工作：

1. 管理用户对表的增删改查操作。
2. 管理HRegionServer的负载均衡，调整Region分布。
3. Region Split后，负责新Region的分布。
4. 在HRegionServer停机后，负责失效HRegionServer上Region迁移。

**HRegionServer**

HRegionServer是HBase中最核心的模块，主要负责响应用户I/O请求，向HDFS文件系统读写数据。HBase中的所有数据从底层来说一般都是保存在HDFS中的，用户通过一系列HRegionServer获取这些数据。集群一个节点上一般只运行一个HRegionServer，且每一个区段的HRegion只会被一个HRegionServer维护。HRegionServer主要负责响应用户I/O请求，向HDFS文件系统读写数据，是HBase中最核心的模块。HRegionServer内部管理了一系列HRegion对象，每个HRegion对应了逻辑表中的一个连续数据段。HRegion由多个HStore组成，每个HStore对应了逻辑表中的一个列族的存储，可以看出每个列族其实就是一个集中的存储单元。因此，为了提高操作效率，最好将具备共同I/O特性的列放在一个列族中。

1. 存放和管理本地HRegion。
2. 读写HDFS，管理Table中的数据。
3. Client直接通过HRegionServer读写数据（从HMaster中获取元数据，找到RowKey所在的HRegion/HRegionServer后）。

另外，Hbase架构中还包括以下几个部分：

**HRegion**

当表的大小超过预设值的时候，HBase会自动将表划分为不同的区域，每个区域包含表中所有行的一个子集。对用户来说，每个表是一堆数据的集合，靠主键（RowKey）来区分。从物理上来说，一张表被拆分成了多块，每一块就是一个HRegion。我们用表名+开始/结束主键，来区分每一个HRegion，一个HRegion会保存一个表中某段连续的数据，一张完整的表数据是保存在多个HRegion中的。

**HStore**

它是HBase存储的核心，由MemStore和StoreFiles两部分组成。MemStore是内存缓冲区，用户写入的数据首先会放入MemStore，当MemStore满了以后会Flush成一个StoreFile（底层实现是HFile），当StoreFile的文件数量增长到一定阈值后，会触发Compact合并操作，将多个StoreFiles合并成一个StoreFile，合并过程中会进行版本合并和数据删除操作。因此，可以看出HBase其实只有增加数据，所有的更新和删除操作都是在后续的Compact过程中进行的，这样使得用户的写操作只要进入内存就可以立即返回，保证了HBaseI/O的高性能。当StoreFiles Compact后，会逐步形成越来越大的StoreFile，当单个StoreFile大小超过一定阈值后，会触发Split操作，同时把当前的HRegion Split成2个HRegion，父HRegion会下线，新分出的2个子HRegion会被HMaster分配到相应的HRegionServer，使得原先1个HRegion的负载压力分流到2个HRegion上。

**HLog**

每个HRegionServer中都有一个HLog对象，它是一个实现了Write Ahead Log的预写日志类。在每次用户操作将数据写入MemStore的时候，也会写一份数据到HLog文件中，HLog文件会定期滚动刷新，并删除旧的文件（已持久化到StoreFile中的数据）。当HMaster通过Zookeeper感知到某个HRegionServer意外终止时，HMaster首先会处理遗留的 HLog文件，将其中不同HRegion的HLog数据进行拆分，分别放到相应HRegion的目录下，然后再将失效的HRegion重新分配，领取到这些HRegion的HRegionServer在加载 HRegion的过程中，会发现有历史HLog需要处理，因此会Replay HLog中的数据到MemStore中，然后Flush到StoreFiles，完成数据恢复。

### 物理存储

- 每个column family存储在HDFS上的一个单独文件中，空值不会被保存；
- Key和Version number在每个column family中均有一份；
- HBase为每个值维护了多级索引，即：< key, column family, column name, timestamp >。

物理存储:

1. Table中所有行都按照RowKey的字典序排列；
2. Table在行的方向上分割为多个Region；
3. Region按大小分割的，每个表开始只有一个region，随着数据增多，region不断增大，当增大到一个阀值的时候，region就会等分会两个新的region，之后会有越来越多的region；
4. Region是Hbase中分布式存储和负载均衡的最小单元，不同Region分布到不同RegionServer上。
5. Region虽然是分布式存储的最小单元，但并不是存储的最小单元。事实上，Region由一个或者多个Store组成，每个store保存一个column family。每个Strore又由一个memStore和0至多个StoreFile组成。
6. StoreFile以HFile格式保存在HDFS上，Hfile的格式如下：
    1. Data Block段：保存表中的数据，这部分可以被压缩，hbase I/O的基本单元；
    2. Meta Block段 (可选的)：保存用户自定义的kv对，可以被压缩。
    3. File Info段：HFile的元信息，不被压缩，用户也可以在这一部分添加自己的元信息。
    4.  Data Block Index段：Data Block的索引。每条索引的key是被索引的block的第一条记录的key。采用LRU机制淘汰，该blockcache的大小直接影响读性能。
    5.  Meta Block Index段 (可选的)：Meta Block的索引。
    6.  Trailer段：这一段是定长的。保存了每一段的偏移量，读取一个HFile时，会首先读取Trailer，Trailer保存了每个段的起始位置(段的Magic Number用来做安全check)，然后，DataBlock Index会被读取到内存中，这样，当检索某个key时，不需要扫描整个HFile，而只需从内存中找到key所在的block，通过一次磁盘io将整个block读取到内存中，再找到需要的key。
7.  HLog文件：一个普通的Hadoop Sequence File，Sequence File 的Key是HLogKey对象，HLogKey中记录了写入数据的归属信息，除了table和region名字外，同时还包括 sequence number和timestamp，timestamp是”写入时间”，sequence number的起始值为0，或者是最近一次存入文件系统中sequence number。HLog Sequece File的Value是HBase的KeyValue对象，即对应HFile中的KeyValue。类似mysql中的binlog，用来做灾难恢复用，Hlog记录数据的所有变更，一旦数据修改，就可以从log中进行恢复。
8.  WAL(Write-Ahead-Log)预写日志：是Hbase的RegionServer在处理数据插入和删除的过程中用来记录操作内容的一种日志。在每次Put、Delete等一条记录时，首先将其数据写入到RegionServer对应的HLog文件的过程。客户端往RegionServer端提交数据的时候，会先写WAL日志，只有当WAL日志写成功以后，客户端才会被告诉提交数据成功，如果写WAL失败会告知客户端提交失败，换句话说这其实是一个数据落地的过程。在一个RegionServer上的所有的Region都共享一个HLog，一次数据的提交是先写WAL，写入成功后，再写memstore。当memstore值到达一定是，就会形成一个个StoreFile（理解为HFile格式的封装，本质上还是以HFile的形式存储的）。

## 工作机制

### Region分配

任何时刻，一个region只能分配给一个region server。master记录了当前有哪些可用的region server。以及当前哪些region分配给了哪些region server，哪些region还没有分配。当存在未分配的region，并且有一个region server上有可用空间时，master就给这个region server发送一个装载请求，把region分配给这个region server。region server得到请求后，就开始对此region提供服务。 

### Region Server上线与下线

**上线**

master使用zookeeper来跟踪region server状态。当某个region server启动时，会首先在zookeeper上的server目录下建立代表自己的文件，并获得该文件的独占锁。由于master订阅了server目录上的变更消息，当server目录下的文件出现新增或删除操作时，master可以得到来自zookeeper的实时通知。因此一旦region server上线，master能马上得到消息。 

**下线**

当region server下线时，它和zookeeper的会话断开，zookeeper而自动释放代表这台server的文件上的独占锁。而master不断轮询server目录下文件的锁状态。如果master发现某个region server丢失了它自己的独占锁，(或者master连续几次和region server通信都无法成功)，master就是尝试去获取代表这个region server的读写锁，一旦获取成功，就可以确定：

1. region server和zookeeper之间的网络断开了。
2. region server挂了。

无论哪种情况，region server都无法继续为它的region提供服务了，此时master会删除server目录下代表这台region server的文件，并将这台region server的region分配给其它还活着的server。

如果网络短暂出现问题导致region server丢失了它的锁，那么region server重新连接到zookeeper之后，只要代表它的文件还在，它就会不断尝试获取这个文件上的锁，一旦获取到了，就可以继续提供服务。

### HMaster上线与下线

**上线**

1. 从zookeeper上获取唯一一个代表master的锁，用来阻止其它master成为master。
2. 扫描zookeeper上的server目录，获得当前可用的region server列表。
3. 和2中的每个region server通信，获得当前已分配的region和region server的对应关系。
4. 扫描.META.region的集合，计算得到当前还未分配的region，将他们放入待分配region列表。

**下线**

由于master只维护表和region的元数据，而不参与表数据I/O的过程，master下线仅导致所有元数据的修改被冻结(无法创建删除表，无法修改表的schema，无法进行region的负载均衡，无法处理region上下线，无法进行region的合并，唯一例外的是region的split可以正常进行，因为只有region server参与)，表的数据读写还可以正常进行。因此master下线短时间内对整个HBase集群没有影响。从上线过程可以看到，master保存的信息全是可以冗余信息（都可以从系统其它地方收集到或者计算出来），因此，一般HBase集群中总是有一个master在提供服务，还有一个以上的"master"在等待时机抢占它的位置。

### Region定位

hbase 使用三层类似B+树的结构来保存region位置：

- 第一层是保存zookeeper里面的文件，它持有root region的位置。
- 第二层root region是.META.表的第一个region，其中保存了.META.的位置。通过root region，我们就可以访问.META.表的数据。
- 第三层.META.表，它是一个特殊的表，保存了hbase中所有数据表的region位置信息。

假设，.META.表的一行在内存中大约占用1KB。并且每个region限制为128MB。
那么上面的三层结构可以保存的region数目为：(128MB/1KB) * (128MB/1KB) = 2^34个。

注意：

1. root region永远不会被split，保证了只需要三次跳转，就能定位到任意region 。
2. .META.表每行保存一个region的位置信息，RowKey 采用表名+表的最后一样编码而成。
3. 为了加快访问，.META.表的全部region都保存在内存中。
4. client会将查询过的位置信息保存缓存起来，缓存不会主动失效

### 读写流程

之前提到，hbase使用MemStore和StoreFile存储对表的更新。

数据在更新时首先写入Log(WAL log)和内存(MemStore)中，MemStore中的数据是排序的，当MemStore累计到一定阈值时，就会创建一个新的MemStore，并且将老的MemStore添加到flush队列，由单独的线程flush到磁盘上，成为一个StoreFile。于此同时，系统会在zookeeper中记录一个read point，表示这个时刻之前的变更已经持久化(minor compact)了。

当系统出现意外时，可能导致内存(MemStore)中的数据丢失，此时使用Log(WAL log)来恢复checkpoint之后的数据。

前面提到过StoreFile是只读的，一旦创建后就不可以再修改。因此Hbase的更新其实是不断追加的操作。当一个Store中的StoreFile数目达到一定的阈值后，就会进行一次合并(major compact)，将对同一个key的修改合并到一起，形成一个大的StoreFile，当StoreFile的大小达到一定阈值后，又会对StoreFile进行split，等分为两个StoreFile。

由于对表的更新是不断追加的，处理读请求时，需要访问Store中全部的StoreFile和MemStore，将他们的按照row key进行合并，由于StoreFile和MemStore都是经过排序的，并且StoreFile带有内存中索引，合并的过程还是比较快。

**写请求处理过程**

1. client向region server提交写请求；
2. region server找到目标region；
3. region检查数据是否与schema一致；
4. 如果客户端没有指定版本，则获取当前系统时间作为数据版本；
5. 将更新写入WAL log；
6. 将更新写入Memstore；
7. 判断Memstore的是否需要flush为Store文件。

**读请求处理过程**

1. client向region server提交读请求；
2. region server找到目标region；
3. region检查数据是否与schema一致；
4. 如果客户端没有指定版本，则获取当前系统时间作为数据版本；
5. 查找Memstore是否有数据；
6. 查找blockcache是否有数据。

**Hbase为什么能实现快速的查询？**

如果快速查询（从磁盘读数据），hbase是根据rowkey查询的，只要能快速的定位rowkey，就能实现快速的查询，主要是以下因素：

1. hbase是可划分成多个region，可以简单的理解为关系型数据库的多个分区。
2. 键是排好序了的。
3. 按列存储的。

- 首先，能快速找到行所在的region(分区)，假设表有10亿条记录，占空间1TB，分裂成了500个region，1个region占2个G。最多读取2G的记录，就能找到对应记录；
- 其次，是按列存储的，其实是列族，假设分为3个列族，每个列族就是666M，如果要查询的东西在其中1个列族上，1个列族包含1个或者多个HStoreFile，假设一个HStoreFile是128M，该列族包含5个HStoreFile在磁盘上. 剩下的在内存中。
- 再次，是排好序了的，要的记录有可能在最前面，也有可能在最后面，假设在中间，我们只需遍历2.5个HStoreFile共300M。
- 最后，每个HStoreFile(HFile的封装)，是以键值对（key-value）方式存储，只要遍历一个个数据块中的key的位置，并判断符合条件可以了。一般key是有限的长度，假设跟value是1:19（忽略HFile上其它块），最终只需要15M就可获取的对应的记录，按照磁盘的访问100M/S，只需0.15秒。加上块缓存机制（LRU原则），会取得更高的效率。

**HBase的实时查询**

- 实时查询，可以认为是从内存中查询，一般响应时间在1秒内。HBase的机制是数据先写入到内存中，当数据量达到一定的量（如128M），再写入磁盘中，在内存中，是不进行数据的更新或合并操作的，只增加数据，这使得用户的写操作只要进入内存中就可以立即返回，保证了HBase I/O的高性能。
- 实时查询即反映当前时间的数据，可以认为这些数据始终是在内存的，保证了数据的实时响应。

## 安装与配置

详见[Ubuntu16.04环境下安装配置HBase2.1.0集群](./installing-hbase2.1.0-on-ubuntu.md)。

## Shell操作

详见[HBase的Shell基本操作](./shell-command-of-hbase.md)。

## Java Api

详见[HBase的Java Api](./java-api-of-hbase.md)
