# Hadoop分布式文件系统：HDFS

## 概述

### 设计理念

HDFS的设计理念是可以运行在普通机器上，以流式数据方式存取文件，一次写入，多次查询。

1. 可构建在廉价机器上：HDFS设计理念之一就是让它能运行在普通的硬件之上，即便硬件出现故障，也可以通过容错策略来保证数据的高可用。
2. 高容错性：HDFS将数据自动保存多个副本，副本丢失后，自动恢复，实现数据高容错性。
3. 适合批处理：也称为流式数据访问。HDFS适合一次写入、多次查询（读取）的情况。在数据集生成后，长时间在此数据集上进行各种分析。
4. 适合存储大文件：单个文件大，文件数量规模大。

### 局限

1. 实时性不好：要求低时间延迟的访问的应用，不适合在HDFS上运行。记住，HDFS是为高数据吞吐量应用优化的，这可能会以高时间延迟为代价。目前，对于低延迟的访问需求，HBase是更好的选择。
2. 小文件问题：由于NameNode将文件系统的元数据存储在内存中，因此该文件系统所能存储的文件总量受限于NameNode的内存总容量。根据经验，每个文件、目录和数据块的存储信息大约占150字节。过多的小文件存储会大量消耗NameNode的存储量。
3. 文件修改问题：HDFS中的文件只有一个Writer，它不支持具有多个写入者的操作，也不支持在文件的任意位置进行修改。

### 适用场景

1. 存取并管理PB级数据；
2. 处理非结构化数据；
3. 注重数据处理的吞吐量，对延时不敏感；
4. 应用模式为一次写入多次读取的存取模式。

### 架构

HDFS是一个主从结构，一个HDFS集群由一个（用于管理文件命名空间和调节客户端访问文件的主服务器）节点NameNode和一些（用于存储具体文件数据的服务器）节点DataNodes组成。

- NameNode：负责管理文件目录、文件和Block的对应关系以及Block和DataNode的对应关系。NameNode节点也成为元数据节点，用来管理文件系统的命名空间，维护目录树，接管用户的请求。 
  - 将文件的元数据保存在一个文件目录树中。
  - 在磁盘上保存为：fsimage和edits。
  - 保存DataNode的数据信息的文件，在系统启动时读入内存。
- DataNode：Datanode是文件系统的工作节点。它们根据需要存储并检索数据块（受客户端或NameNode调度），并且定期向NameNode发送他们所存储的块的列表。
- Client：客户端代表用户通过与NameNode和DataNode交互来访问整个文件系统。客户端提供了一个类似于POSIX（可移植操作系统界面）的文件系统接口，因此用户在编程时无需知道NameNode和DataNode也可以实现其功能。HDFS对外开放文件命名空间并允许用户数据以文件形式存储。用户通过客户端（Client）与HDFS进行通讯交互。
- SecondaryNameNode：文件系统客户端在执行写操作时，这些操作首先被记录到编辑日志中，然后更新NameNode内存中维护的文件系统的元数据；命名空间镜像文件（fsimage）是文件系统元数据的一个永久检查点，因为fsimage文件是一个大型文件，如果频繁的执行写操作，会使系统运行极为缓慢。但是如果不进行操作，editlog文件会无限增长，一旦NameNode需要恢复，则需要花费非常长的时间，所以HDFS引入了辅助NameNode，为主NameNode内存中的文件系统元数据创建检查点。实现将主NameNode的编辑日志和命名空间镜像文件合并。形成一个更小的editlog文件和最新的fsimage文件。客户端执行写操作的时候，首先更新编辑日志（edits），然后修改内存中的文件系统映射。随着时间的推移，编辑日志会非常大。这个时候，就需要将编辑日志和文件系统映射持久检查点（fsimage）进行合并，以减少编辑日志的大小。这个工作就是由辅助NameNode完成的。SecondaryNameNode工作原理：
  1. 辅助NameNode请求主NameNode停止使用edits文件，暂时将新的写操作记录到一个新的文件中；
  2. 辅助NameNode通过http get请求从主NameNode获取fsimage和edits文件；
  3. 辅助NameNode将fsimage文件载入内存，逐一执行edits文件中的操作，创建新的fsimage文件；
  4. 辅助NameNode通过使用http post将新的fsimage文件发送回主NameNode；
  5. 主NameNode用从辅助NameNode接收的fsimage文件替换旧的fsimage文件；用步骤1所产生的edits文件替换旧的edits文件。同时，还更新fstime文件来记录检查点执行时间。

最终，主NameNode拥有最新的fsimage文件和一个更小的edits文件。当NameNode处在安全模式时，管理员也可调用hadoop dfsadmin –saveNameSpace命令来创建检查点。

从上面的过程中我们清晰的看到辅助NameNode和主NameNode拥有相近内存需求的原因（因为辅助NameNode也把fsimage文件载入内存）。因此，在大型集群中，辅助NameNode需要运行在一台专用机器上。

创建检查点的触发条件受两个配置参数控制。通常情况下，辅助NameNode每隔一小时（有fs.checkpoint.period属性设置）创建检查点；此外，当编辑日志的大小达到64MB（有fs.checkpoint.size属性设置）时，也会创建检查点。系统每隔五分钟检查一次编辑日志的大小。

### 块

HDFS内部机制是将一个文件分割成一个或多个块，这些块被存储在一组数据节点中。NameNode节点用来操作文件命名空间的文件或目录操作，如打开，关闭，重命名等等。它同时确定块与数据节点DataNode的映射。数据节点DataNode负责来自文件系统客户的读写请求。数据节点DataNode同时还要执行块的创建，删除，和来自NameNode的块复制指令。

dfs.blocksize是一个文件块的大小，默认64M。 
每一个block会在多个DataNode上存储多份副本，默认是3份。 
太大的话会有较少map同时计算，太小的话也浪费可用map个数资源，而且文件太小NameNode就浪费内存多。所以应该根据需要进行设置。

## 文件读写

### 权限控制

对于文件和目录，HDFS提供三种权限模式：读（r）、写（w）、执行（x）。读取文件或列出目录内容时需要只读权限。写入一个文件，或是在一个目录上创建及删除文件或目录，需要写入权限。对于文件而言，可执行权限可以忽略，因为你不能在HDFS中执行文件，但在访问一个目录的子项时需要该权限。

每个文件和目录都有所属用户（owner）、所属组别（group）及模式（mode）。这个模式是由所属用户的权限、组内成员的权限及其他用户的权限组成的。 

默认情况下，可以通过正在运行进程的用户名和组名来唯一确定客户端的标示。但由于客户端是远程的，任何用户都可以简单的在远程系统上以他的名义创建一个账户来进行访问。因此，作为共享文件系统资源和防止数据意外损失的一种机制，权限只能供合作团体中的用户使用，而不能在一个不友好的环境中保护资源。注意，最新的hadoop系统支持kerberos用户认证，该认证去除了这些限制。但是，除了上述限制之外，为防止用户或者自动工具及程序意外修改或删除文件系统的重要部分，启用权限控制还是很重要的。 

注意：这里有一个超级用户的概念，超级用户是NameNode进程的标识。对于超级用户，系统不会执行任何权限检查。

### 读文件

1. 客户端通过调用FileSystem对象的open()方法来打开希望读取的文件，对于HDFS来说，这个对象是分布式文件系统的一个实例。 
2. DistributedFileSystem通过使用RPC来调用NameNode，以确定文件起始块的位置。对于每一个块，NameNode返回存有该块复本的DataNode地址。此外，这些DataNode根据他们与客户端的距离来排序。如果客户端本身就是一个DataNode，并保存有相应数据块的一个复本时，该节点将从本地DataNode中读取数据。 
3. DistributedFileSystem类返回一个FSDataInputStream对象给客户端读取数据。FSDataInputStream类转而封装DFSInputStream对象，该对象管理着DataNode和NameNode的I/O。 
4. 接着，客户端对这个输入流调用read()方法。存储着文件起始块的DataNode地址的DFSInputStream随即连接距离最近的DataNode。通过对数据流反复调用read()方法，可以将数据从DataNode传输到客户端。到达块的末端时，DFSInputStream会关闭与该DataNode的连接，然后寻找下一个块的最佳DataNode。客户端只需要读取连续的流，并且对于客户端都是透明的。 
5. 客户端从流中读取数据时，块是按照打开DFSInputStream与DataNode新建连接的顺序读取的。它也需要询问NameNode来检索下一批所需块的DataNode的位置。一旦客户端完成读取，就对FSDataInputStream调用close()方法。 

注意：在读取数据的时候，如果DFSInputStream在与DataNode通讯时遇到错误，它便会尝试从这个块的另外一个临近DataNode读取数据。他也会记住那个故障DataNode，以保证以后不会反复读取该节点上后续的块。DFSInputStream也会通过校验和确认从DataNode发送来的数据是否完整。如果发现一个损坏的块， DFSInputStream就会在试图从其他DataNode读取一个块的副本之前通知NameNode。 

总结：在这个设计中，NameNode会告知客户端每个块中最佳的DataNode，并让客户端直接联系该DataNode去检索数据。由于数据流分散在该集群中的所有DataNode，所以这种设计会使HDFS可扩展到大量的并发客户端。同时，NameNode仅需要响应位置的请求（这些信息存储在内存中，非常高效），而无需响应数据请求，否则随着客户端数量的增长，NameNode很快会成为一个瓶颈。

### 写文件

1. 首先客户端通过DistributedFileSystem上的create()方法指明一个预创建的文件的文件名。
2. DistributedFileSystem再通过RPC调用向NameNode申请创建一个新文件（这时该文件还没有分配相应的block）。NameNode检查是否有同名文件存在以及用户是否有相应的创建权限，如果检查通过，NameNode会为该文件创建一个新的记录，否则的话文件创建失败，客户端得到一个IOException异常。DistributedFileSystem返回一个FSDataOutputStream以供客户端写入数据，与FSDataInputStream类似，FSDataOutputStream封装了一个DFSOutputStream用于处理NameNode与datanode之间的通信。 
3. 当客户端开始写数据时，DFSOutputStream把写入的数据分成包（packet）, 放入一个中间队列——数据队列（data queue）中去。DataStreamer从数据队列中取数据，同时向NameNode申请一个新的block来存放它已经取得的数据。NameNode选择一系列合适的DataNode（个数由文件的replica数决定）构成一个管道线（pipeline），这里我们假设replica为3，所以管道线中就有三个DataNode。
4. DataSteamer把数据流式的写入到管道线中的第一个DataNode中，第一个DataNode再把接收到的数据转到第二个DataNode中，以此类推。 
5. DFSOutputStream同时也维护着另一个中间队列——确认队列（ack queue），确认队列中的包只有在得到管道线中所有的DataNode的确认以后才会被移出确认队列。 
6. 当客户端完成写数据后，它会调用close()方法。这个操作会冲洗（flush）所有剩下的package到pipeline中。
7. 等待这些package确认成功，然后通知NameNode写入文件成功。这时候NameNode就知道该文件由哪些block组成（因为DataStreamer向NameNode请求分配新block，NameNode当然会知道它分配过哪些blcok给给定文件），它会等待最少的replica数被创建，然后成功返回。

如果某个DataNode在写数据的时候当掉了，下面这些对用户透明的步骤会被执行： 
1. 管道线关闭，所有确认队列上的数据会被挪到数据队列的首部重新发送，这样可以确保管道线中当掉的DataNode下流的DataNode不会因为当掉的DataNode而丢失数据包。 
2. 在还在正常运行的DataNode上的当前block上做一个标志，这样当当掉的DataNode重新启动以后NameNode就会知道该DataNode上哪个block是刚才当机时残留下的局部损坏block，从而可以把它删掉。 
3. 已经当掉的DataNode从管道线中被移除，未写完的block的其他数据继续被写入到其他两个还在正常运行的DataNode中去，NameNode知道这个block还处在under-replicated状态（也即备份数不足的状态）下，然后他会安排一个新的replica从而达到要求的备份数，后续的block写入方法同前面正常时候一样。

有可能管道线中的多个DataNode当掉（虽然不太经常发生），但只要dfs.replication.min（默认为1）个replica被创建，我们就认为该创建成功了。剩余的replica会在以后异步创建以达到指定的replica数。 

注意：
1. hdfs在写入的过程中，有一点与hdfs读取的时候非常相似，就是：DataStreamer在写入数据的时候，每写完一个DataNode的数据块（默认64M）,都会重新向NameNode申请合适的DataNode列表。这是为了保证系统中DataNode数据存储的均衡性。
2. hdfs写入过程中，DataNode管线的确认应答包并不是每写完一个DataNode，就返回一个确认应答，而是一直写入，直到最后一个DataNode写入完毕后，统一返回应答包。如果中间的一个DataNode出现故障，那么返回的应答就是前面完好的DataNode确认应答，和故障DataNode的故障异常。这样我们也就可以理解，在写入数据的过程中，为什么数据包的校验是在最后一个DataNode完成
