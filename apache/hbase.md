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

| 优缺点 | 行式存储 | 列式存储 |
| --- | --- | --- |
| 优点 | 数据被保存在一起，INSERT/UPDATE容易。 | 查询时只有涉及到的列会被读取；投影(projection)很高效；任何列都能作为索引。 |
| 缺点 | 选择(Selection)时即使只涉及某几列，所有数据也都会被读取。 | 选择完成时，被选择的列要重新组装。INSERT/UPDATE比较麻烦。 |

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

- RowKey：Byte Array，是表中每条记录的“主键”，方便快速查找，Rowkey的设计非常重要。
- Column Family：列族，拥有一个名称(string)，包含一个或者多个相关列。
- Column：属于某一个columnfamily，familyName:columnName，每条记录可动态添加。
- Version Number：类型为Long，默认值是系统时间戳，可由用户自定义。
- Value(Cell)：Byte Array。

如果用编程语言风格来表示HBase数据存储模式，可以这样表示：`Map<RowKey, List<Map<Column, List<Value, Timestamp>>>>`。

### 架构

HBase包含3个重要组件：Zookeeper、HMaster和HRegionServer。

**Zookeeper**

Zookeeper为整个HBase提供协助服务，包括：

1. 存放整个HBase集群的元数据以及集群的状态信息。
2. 实现HMaster主从节点的failover。

ZooKeeper为HBase集群提供协调服务，它管理着HMaster和HRegionServer的状态(available/alive等)，并且会在它们宕机时通知给HMaster，从而HMaster可以实现HMaster之间的failover，或对宕机的HRegionServer中的HRegion集合的修复(将它们分配给其他的HRegionServer)。ZooKeeper集群本身使用一致性协议(PAXOS协议)保证每个节点状态的一致性。

**HMaster**

HMaster主要用于监控和操作集群中的所有HRegionServer。HMaster没有单点问题，HBase中可以启动多个HMaster，通过Zookeeper的MasterElection机制保证总有一个Master在运行 

HMaster主要负责Table和Region的管理工作：

1. 管理用户对表的增删改查操作。
2. 管理HRegionServer的负载均衡，调整Region分布。
3. Region Split后，负责新Region的分布。
4. 在HRegionServer停机后，负责失效HRegionServer上Region迁移。

**HRegionServer**

HRegionServer是HBase中最核心的模块，主要负责响应用户I/O请求，向HDFS文件系统读写数据。

1. 存放和管理本地HRegion。
2. 读写HDFS，管理Table中的数据。
3. Client直接通过HRegionServer读写数据（从HMaster中获取元数据，找到RowKey所在的HRegion/HRegionServer后）。

### 物理存储

- 每个column family存储在HDFS上的一个单独文件中，空值不会被保存；
- Key和Version number在每个column family中均有一份； 
- HBase为每个值维护了多级索引，即：< key, column family, column name, timestamp >。 

物理存储: 

1. Table中所有行都按照RowKey的字典序排列； 
2. Table在行的方向上分割为多个Region； 
3. Region按大小分割的，每个表开始只有一个region，随着数据增多，region不断增大，当增大到一个阀值的时候，region就会等分会两个新的region，之后会有越来越多的region； 
4. Region是Hbase中分布式存储和负载均衡的最小单元，不同Region分布到不同RegionServer上。
