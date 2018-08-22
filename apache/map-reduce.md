# Hadoop分布式运算框架：MapReduce

## 概述

MapReduce是一种编程模型，用于大规模数据集（大于1TB）的并行运算。概念Map（映射）和Reduce（归约），是它们的主要思想。它极大地方便了编程人员在不会分布式并行编程的情况下，将自己的程序运行在分布式系统上。当前的软件实现是指定一个Map（映射）函数，用来把一组键值对映射成一组新的键值对，指定并发的Reduce（归约）函数，用来保证所有映射的键值对中的每一个共享相同的键组。

MapReduce是面向大数据并行处理的计算模型、框架和平台，它隐含了以下三层含义：

- MapReduce是一个基于集群的高性能并行计算平台（Cluster Infrastructure）。它允许用市场上普通的商用服务器构成一个包含数十、数百至数千个节点的分布和并行计算集群。
- MapReduce是一个并行计算与运行软件框架（Software Framework）。它提供了一个庞大但设计精良的并行计算软件框架，能自动完成计算任务的并行化处理，自动划分计算数据和计算任务，在集群节点上自动分配和执行任务以及收集计算结果，将数据分布存储、数据通信、容错处理等并行计算涉及到的很多系统底层的复杂细节交由系统负责处理，大大减少了软件开发人员的负担。
- MapReduce是一个并行程序设计模型与方法（Programming Model & Methodology）。它借助于函数式程序设计语言Lisp的设计思想，提供了一种简便的并行程序设计方法，用Map和Reduce两个函数编程实现基本的并行计算任务，提供了抽象的操作和并行编程接口，以简单方便地完成大规模数据的编程和计算处理。

## 执行过程

### Input

一般而言，MapReduce的输入来源是HDFS。已知HDFS中block的大小是128M（Hadoop2.x默认是128M，Hadoop1.x默认是64M）。MapReduce计算框架首先会用InputFormat的子类FileInputFormat类对输入文件进行切分，形成输入分片（InputSplit）。每个InputSplit分片将作为一个Map任务的输入，输入分片存储的并非数据本身，而是一个分片长度和一个记录数据位置的数组。也就是说，InputSplit只是对输入数据进行逻辑上的划分，并不会对物理文件切分成片进行存储。那如何确定InputSplit的大小呢？

1. 如果mapred-site.xml中没有设置分片大小的范围，InputSplit分片大小就由block块决定，即是InputSplit分片大小等于block块大小。
2. 如果mapred-site.xml中设置了`mapred.min.split.size`和`mapred.max.split.size`参数，就可以控制InputSplit的大小。如下所示：

  ```java
  /**
    * 参数mapred.min.split.size的默认值为1个字节；
    * minSplitSize随着FileFormat的不同而不同；
    * mapred.max.split.size默认为Long.MAX_VALUE = 9223372036854775807；
    * */
  minSize = max{minSplitSize, mapred.min.split.size} 
  maxSize = mapred.max.split.size
  splitSize = max{minSize,min{maxSize,blockSize}}
  ```

  InputSplit分片大小的下限是max{mapred.min.split.size, minSplitSize}，上限是mapred.max.split.size。

  由此可知：一个InputSplit分片可以大于一个block块，也可以小于一个block块。对于Map任务来说，处理的单位是一个InputSplit。请注意InputSplit是一个逻辑概念，InputSplit所包含的数据仍然存储在HDFS的block块中。

  在进行任务调度时，会优先考虑本节点的数据。如果本节点没有可处理的数据或者还需要其他节点的数据，Map任务就必须从其他节点将数据通过网络传递到本节点，性能受到影响。如果InputSplit的大小大于block块大小时，Map任务就必须从其他节点读取数据，这样就不能很好实现数据本地性。所以，InputSplit的大小尽量等于block块大小，以提高Map任务的数据本地性。

### Map

InputSplit将键值数据传给map方法处理后，输出中间结果到本地磁盘。在这个过程中，为了保证IO效率，采用了先写入内存的环形缓冲区，并做一次预排序。

1. 首先map输出到内存中的一个环状的内存缓冲区，缓冲区的大小默认为100MB（可通过修改配置项`mpareduce.task.io.sort.mb`进行修改）。
2. 然后，当写入内存缓冲区的大小到达一定比例时，默认为80%（可通过`mapreduce.map.sort.spill.percent`配置项修改），将启动一个溢写线程将内存缓冲区的内容溢写到磁盘（spill to disk）。这个溢写线程是独立的，不影响map向缓冲区写结果的线程。当溢写线程启动后，需要对80M的空间内的数据按照key进行排序。溢写线程会在磁盘中新建一个溢出写文件，溢写线程默认根据数据键值对溢写出文件进行分区（patition），接着后台线程将根据数据最终要传送到的Reduce把内存缓冲区中的数据写入溢出写文件对应分区。在每个patition分区，后台线程按键值进行内排序，此时如果有一个Combiner，则会在排序后的输出上运行。默认的patition分区算法是将每个键值对的键的Hash值与reducer数量进行模运算得到patition值。
3. 随着map处理，map输出数据增多，磁盘中溢写文件文件的数据也在增加。这就需要将磁盘中的多个小的溢写文件合并成一个大文件。注意，合并后的大文件已经进行了分区，每个分区内进行了排序，该大文件就是Map任务的输出结果。
4. 为了提高磁盘IO性能，可以将Map输出进行压缩，这样磁盘书写速度提高，可以减少磁盘空间，减少传递给Reducer的数据量。注意，默认情况下，map输出是不压缩的，可以在mapred-site.xml文件中配置`mapreduce.output.fileoutputformat.commpress`值为`true`，即可开启压缩功能。

map过程的输出是写入本地磁盘而不是HDFS，但是一开始数据并不是直接写入磁盘而是缓冲在内存中，缓存的好处就是减少磁盘I/O的开销，提高合并和排序的速度。又因为默认的内存缓冲大小是100M（当然这个是可以配置的），所以在编写map函数的时候要尽量减少内存的使用，为shuffle过程预留更多的内存，因为该过程是最耗时的过程。

### Reduce

MapReduce确保每个reducer的输入都是按照key排序的。Shuffler就是mapper和reducer中间的一个步骤，也就是将map的输出转换为reduce的输入的过程。在shuffle过程中，可以把mapper的输出按照某种key值重新切分和组合成n份，把key值符合某种范围的输送到特定的reducer端处理。shuffle是MapReduce运行的的核心。

Map输出结果时进行了Partitioner分区操作。其实Partitioner分区操作和Map阶段的输入分片（Input Split）很像，一个Partitioner对应一个Reduce作业，如果我们MapReduce只有一个Reduce操作，那么Partitioner就只有一个，如果我们有多个Reduce操作，那么Partitioner对应的就会有多个，Partitioner因此就是Reduce的输入分片，主要是根据实际key和value的值，和实际业务类型或者为了更好的Reduce负载均衡要求而进行，这是提高Reduce效率的一个关键所在。

一个Map任务的输出，可能被多个Reduce任务抓取。每个Reduce任务可能需要多个Map任务的输出作为其特殊的输入文件，而每个Map任务的完成时间可能不同，当Map任务完成时，Reduce任务就开始运行。Reduce任务根据分区号在多个Map输出中抓取（Fetch）对应分区的数据，这个过程也就是Shuffle的copy过程。复制操作时Reduce会开启几个复制线程，这些线程默认个数是5个。这个复制过程和Map写入磁盘过程类似，也有阀值和内存大小，阀值一样可以在配置文件里配置，而内存大小是直接使用Reduce的TaskTracker的内存大小，复制时候Reduce还会进行排序操作和合并文件操作。

如果map输出很小，则会被复制到Reducer所在节点的内存缓冲区，缓冲区的大小可以通过mapred-site.xml文件中的`mapreduce.reduce.shuffle.input.buffer.percent`指定。一旦Reducer所在节点的内存缓冲区达到阀值，或者缓冲区中的文件数达到阀值，则合并溢写到磁盘。 如果map输出较大，则直接被复制到Reducer所在节点的磁盘中。 

随着Reducer所在节点的磁盘中溢写文件增多，后台线程会将它们合并为更大且有序的文件。 

当完成复制Map输出，进入Sort阶段。这个阶段通过**归并排序**逐步将多个Map输出小文件合并成大文件。最后几个通过归并合并成的大文件作为Reduce的输出。

当Reducer的输入文件确定后，整个Shuffle操作才最终结束。之后就是Reducer的执行了，最后Reducer会把结果存到HDFS上。

### 特别说明

#### 排序

排序贯穿Map任务和Reduce任务，在MapReduce计算框架中，主要用到两种排序算法：快速排序和归并排序。在Map任务发生了2次排序，Reduce任务发生一次排序。 

1. 第1次排序发生在Map输出的内存环形缓冲区，使用快速排序。当缓冲区达到阀值时，在溢写到磁盘之前，后台线程会将缓冲区的数据划分成相应分区，在每个分区中按照键值进行内排序。 
2. 第2次排序是在Map任务输出的磁盘空间上将多个溢写文件归并成一个已分区且有序的输出文件。由于溢写文件已经经过一次排序，所以合并溢写文件时只需一次归并排序即可使输出文件整体有序。 
3. 第3次排序发生在Shuffle阶段，将多个复制过来的Map输出文件进行归并，同样经过一次归并排序即可得到有序文件

#### 其他说明

1. 各个Map函数对所划分的数据并行处理，从不同的输入数据产生不同的中间结果输出；
2. 各个Reduce各自并行计算，各自负责处理不同的中间结果数据集合;
3. 进行Reduce处理之前，须等到所有的Map函数做完，并且在进入Reduce前会对Map的中间结果数据进行整理(Shuffle)，保证将Map的结果发送给对应的Reduce；
4. 最终汇总所有Reduce的输出结果即可获得最终结果；

## 工作机制

### 基础模式

1. MapReduce函数库首先把输入文件分成M块，每块大概64MB到128MB。接着在集群的机器上执行处理程序；
2. MapReduce算法运行过程中有一个主控程序，称为Master。主控程序会产生很多作业程序，称为Worker。并且把M个Map任务和R个Reduce任务分配给这些Worker，让它们去完成；
3. 被分配了Map任务的Worker读取并处理相关的输入(这里的输入是指已经被切割的输入小块split)。它处理输入的数据，并且将分割出的键/值(key/value)对传递给用户定义的Map()函数。Map()函数产生的中间结果键/值(key/value)对暂时缓冲到内存；
4. Map()函数缓冲到内存的中间结果将被定时刷写到本地硬盘，这些数据通过分区函数分成R个区。这些中间结果在本地硬盘的位置信息将被发送回Master，然后这个Master负责把这些位置信息传送给Reduce()函数的Worker；
5. 当Master通知了Reduce()函数的Worker关于中间键/值(key/value)对的位置时，Worker调用远程方法从Map()函数的Worker机器的本地硬盘上读取缓冲的中间数据。当Reduce()函数的Worker读取到了所有的中间数据，它就使用这些中间数据的键(key)进行排序，这样可以使得相同键(key)的值都在一起；
6. Reduce()函数的Worker根据每一个中间结果的键(key)来遍历排序后的数据，并且把键(key)和相关的中间结果值(value)集合传递给Reduce()函数。Reduce()函数的Worker最终把输出结果存放在Master机器的一个输出文件中；
7. 当所有的Map任务和Reduce任务都已经完成后，Master激活用户程序。在这时，MapReduce返回用户程序的调用点；
8. 当以上步骤成功结束以后，MapReduce的执行数据存放在总计R个输出文件中(每个输出文件都是由Reduce任务产生的，这些文件名是用户指定的)。通常，用户不需要将这R个输出文件合并到一个文件，他们通常把这些文件作为输入传递给另一个MapReduce调用，或者用另一个分布式应用来处理这些文件，并且这些分布式应用把这些文件看成为输入文件由于分区(partition)成为的多个块文件；

### YARN模式

1. 客户端向集群提交作业，启动一个Job。 
2. Job从资源管理器ResourceManager获取新的作业应用程序ID。 
3. 客户端检查作业的输出情况，计算输入分片，并将作业jar包、配置、分片信息等作业资源复制到HDFS。 
4. Job通过调用资源管理器ResourceManager的`submitApplication()`方法提交作业。 
5. ResourceManager接收到作业后，将作业请求传递给调度器。ResourceManager分配一个container，然后ResourceManager在NodeManager的管理下，在container中启动一个ApplicationMaster进程。 
6. ApplicationMaster对作业进行初始化，并保持对作业的跟踪，判断作业是否完成。 
7. ApplicationMaster根据存储在HDFS中的分片信息确定Map和Reduce的数量，获取计算出的输入分片，为每个分片创建一个map任务。并创建reduce任务。 
8. ApplicationMaster为本次作业的Map和Reduce以轮询的方式向ResourceManager申请container。master为作业向资源管理器请求一个容器来运行任务。 
9. ApplicationMaster获取到container后，与NodeManager进行通信启动container。 
10. container从HDFS中获取作业的jar包、配置和分布式缓存文件等，将任务需要的资源本地化。 
11. container启动Map或Reduce任务。
