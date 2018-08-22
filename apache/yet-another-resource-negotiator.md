# Hadoop2.x资源调度平台：YARN

## 概述

在Hadoop 1.x版本中，MapReduce既要负责资源管理又要负责作业处理。MapReduce运行时环境由（一个）JobTracker和（若干个）TaskTracker两类服务组成。其中，JobTracker负责资源管理和所有作业的控制，而TaskTracker负责接收来自JobTracker的命令并执行它。

YARN是Hadoop 2.x中的资源管理系统，它是一个通用的资源管理模块，可为各类应用程序进行资源管理和调度。YARN不仅限于MapReduce一种框架使用，也可以供其他框架使用，比如Hive、Spark、Storm等。由于YARN的通用性，下一代MapReduce的核心已经从简单的支持单一应用的计算框架MapReduce转移到通用的资源管理系统YARN。

## 核心思想

将JobTracker和TaskTracker进行分离，它由下面几大构成组件：
1. 一个全局的资源管理器 ResourceManager；
2. ResourceManager的每个节点代理 NodeManager；
3. 表示每个应用的 ApplicationMaster；
4. 每一个ApplicationMaster拥有多个Container在NodeManager上运行。

## 主要优点

1. 大大减小了JobTracker（也就是现在的ResourceManager）的资源消耗，并且让监测每一个Job子任务(tasks)状态的程序分布式化了，更安全、更优美。
2. 在新的Yarn中，ApplicationMaster是一个可变更的部分，用户可以对不同的编程模型写自己的AppMst，让更多类型的编程模型能够跑在Hadoop集群中，可以参考hadoopYarn官方配置模板中的mapred-site.xml配置。
3. 对于资源的表示以内存为单位(在目前版本的Yarn中，没有考虑cpu的占用)，比之前以剩余slot数目更合理。
4. 老的框架中，JobTracker一个很大的负担就是监控job下的tasks的运行状况，现在，这个部分就扔给ApplicationMaster做了，而ResourceManager中有一个模块叫做ApplicationsMasters(注意不是ApplicationMaster)，它是监测ApplicationMaster的运行状况，如果出问题，会将其在其他机器上重启。
5. Container是Yarn为了将来作资源隔离而提出的一个框架。这一点应该借鉴了Mesos的工作，目前是一个框架，仅仅提供java虚拟机内存的隔离，hadoop团队的设计思路应该后续能支持更多的资源调度和控制，既然资源表示成内存量，那就没有了之前的map slot/reduce slot分开造成集群资源闲置的尴尬情况。

## 主要架构

### ResourceManager（RM）

RM是一个全局的资源管理器，负责整个系统的资源管理和分配。它主要由两个组件构成：调度器（Scheduler）和应用程序管理器（Applications Manager，ASM）。

- 调度器（Scheduler）：调度器根据容量、队列等限制条件（如每个队列分配一定的资源，最多执行一定数量的作业等），将系统中的资源分配给各个正在运行的应用程序。需要注意的是，该调度器是一个“纯调度器”，调度器仅根据各个应用程序的资源需求进行资源分配，而资源分配单位用一个抽象概念“资源容器”（Resource Container，简称Container）表示，Container是一个动态资源分配单位，它将内存、CPU、磁盘、网络等资源封装在一起，从而限定每个任务使用的资源量。此外，该调度器是一个可插拔的组件，用户可根据自己的需要设计新的调度器。
- 应用程序管理器（Applications Manager，ASM）：应用程序管理器负责管理整个系统中所有应用程序，包括应用程序提交、与调度器协商资源以启动ApplicationMaster、监控ApplicationMaster运行状态并在失败时重新启动它等。

### ApplicationMaster（AM）

用户提交的每个应用程序均包含一个AM，它的主要功能包括：

- 与RM调度器协商以获取资源（用Container表示）；
- 将得到的任务进一步分配给内部的任务(资源的二次分配)；
- 与NM通信以启动/停止任务；
- 监控所有任务运行状态，并在任务运行失败时重新为任务申请资源以重启任务。

### NodeManager（NM）

NM是每个节点上的资源和任务管理器，一方面，它会定时地向RM汇报本节点上的资源使用情况和各个Container的运行状态；另一方面，它接收并处理来自AM的Container启动/停止等各种请求。

### Container

Container是YARN中的资源抽象，它封装了某个节点上的多维度资源，如内存、CPU、磁盘、网络等，当AM向RM申请资源时，RM为AM返回的资源便是用Container表示。YARN会为每个任务分配一个Container，且该任务只能使用该Container中描述的资源。

## 工作流程

当用户向YARN中提交一个应用程序后，YARN将分两个阶段运行该应用程序：第一个阶段是启动ApplicationMaster；第二个阶段是由ApplicationMaster创建应用程序，为它申请资源，并监控它的整个运行过程，直到运行完成。具体过程如下：

1. 客户端程序向ResourceManager提交应用并请求一个ApplicationMaster实例；
2. ResourceManager找到可以运行一个Container的NodeManager，并在这个Container中启动ApplicationMaster实例；
3. ApplicationMaster向ResourceManager进行注册，注册之后客户端就可以查询ResourceManager获得自己ApplicationMaster的详细信息，以后就可以和自己的ApplicationMaster直接交互了；
4. 在平常的操作过程中，ApplicationMaster根据resource-request协议向ResourceManager发送resource-request请求
5. 当Container被成功分配之后，ApplicationMaster通过向NodeManager发送container-launch-specification信息来启动Container， container-launch-specification信息包含了能够让Container和ApplicationMaster交流所需要的资料。
6. 应用程序的代码在启动的Container中运行，并把运行的进度、状态等信息通过application-specific协议发送给ApplicationMaster。
7. 在应用程序运行期间，提交应用的客户端主动和ApplicationMaster交流获得应用的运行状态、进度更新等信息，交流的协议也是application-specific协议；
8. 一旦应用程序执行完成并且所有相关工作也已经完成，ApplicationMaster向ResourceManager取消注册然后关闭，用到所有的Container也归还给系统。

## 调度器

Yarn的资源调度目前支持内存和CPU两种资源。Yarn支持三种调度方式：FIFO、FAIR和DRF分别是指先来先服务、公平调度和主资源公平调度。

### FIFO调度器

先按照优先级高低调度，如果优先级相同，则按照提交时间先后顺序调度，如果提交时间相同，则按照（队列或者应用程序）名称大小（字符串比较）调度；不支持有子队列的情况。

在进行资源分配的时候，先给队列中最头上的应用进行分配资源，待最头上的应用需求满足后再给下一个分配，以此类推。 

FIFO Scheduler是最简单也是最容易理解的调度器，也不需要任何配置，但它并不适用于共享集群。 

在FIFO 调度器中，小任务会被大任务阻塞。大的应用可能会占用所有集群资源，这就导致其它应用被阻塞。

### Fair调度器

按照内存资源使用量比率调度，即按照used_memory/minShare大小调度（核心思想是按照该调度算法决定调度顺序，但还需考虑一些边界情况）；

在Fair调度器中，我们不需要预先占用一定的系统资源，Fair调度器会为所有运行的job动态的调整系统资源。当第一个大job提交时，只有这一个job在运行，此时它获得了所有集群资源；当第二个小任务提交后，Fair调度器会分配一半资源给这个小任务，让这两个任务公平的共享集群资源。从第二个任务提交到获得资源会有一定的延迟，因为它需要等待第一个任务释放占用的Container。小任务执行完成之后也会释放自己占用的资源，大任务又获得了全部的系统资源。最终的效果就是Fair调度器即得到了高的资源利用率又能保证小任务及时完成。 

### Capacity调度器

对于Capacity调度器，有一个专门的队列用来运行小任务，但是为小任务专门设置一个队列会预先占用一定的集群资源，这就导致大任务的执行时间会落后于使用FIFO调度器时的时间。

Apache Hadoop Yarn默认使用Capacity调度器。
