# Hive的Shell基本操作

## 前言

在此之前，已经完成了[Ubuntu16.04环境下安装配置Hadoop2.8.1集群](./installing-hadoop2.8.1-on-ubuntu.md)、[Ubuntu16.04环境下安装配置HBase2.1.0集群](./installing-hbase2.1.0-on-ubuntu.md)和[Ubuntu16.04环境下安装配置Hive2.3.3](./installing-hive2.3.3-on-ubuntu.md)。

## 设备配置

- 系统：Ubuntu
- 版本：16.04
- 处理器：Intel(R) Core(TM) i7-7700K CPU @ 4.20GHz
- 内存：2.0GB
- 类型：64位操作系统 64位处理器

## Hadoop环境

- 版本：Hadoop-2.8.1、HBase-2.1.0
- Master：192.168.73.130
- Slave1：192.168.73.132
- Slave2：192.168.73.133

## 连接Hive Shell

查看版本信息

```shell
root@Master:/usr/local/hive/bin# ./hive --version
Hive 2.3.3
Git git://daijymacpro-2.local/Users/daijy/commit/hive -r 8a511e3f79b43d4be41cd231cf5c99e43b248383
Compiled by daijy on Wed Mar 28 16:58:33 PDT 2018
From source with checksum 8873bba6c55a058614e74c0e628ab022
```

输入命令 `/usr/local/hive/bin/hive` 进入Hive的shell控制台。

```shell
root@Master:/usr/local/hive/bin# ./hive
SLF4J: Class path contains multiple SLF4J bindings.
SLF4J: Found binding in [jar:file:/usr/local/hive/lib/log4j-slf4j-impl-2.6.2.jar!/org/slf4j/impl/StaticLoggerBinder.class]
SLF4J: Found binding in [jar:file:/usr/local/hadoop/share/hadoop/common/lib/slf4j-log4j12-1.7.10.jar!/org/slf4j/impl/StaticLoggerBinder.class]
SLF4J: See http://www.slf4j.org/codes.html#multiple_bindings for an explanation.
SLF4J: Actual binding is of type [org.apache.logging.slf4j.Log4jLoggerFactory]

Logging initialized using configuration in jar:file:/usr/local/hive/lib/hive-common-2.3.3.jar!/hive-log4j2.properties Async: true
```

## 基本操作

### 数据库

显示数据库：

```shell
hive> show databases;
OK
default
Time taken: 3.863 seconds, Fetched: 1 row(s)
```

创建数据库：

```shell
hive> create database test;
OK
Time taken: 0.172 seconds
```

查看数据库描述

```shell
hive> desc database test;
OK
test		hdfs://Master:9000/user/hive/warehouse/test.db	root	USER
Time taken: 0.015 seconds, Fetched: 1 row(s)
```

切换当前数据库

```shell
hive> use test;
OK
Time taken: 0.015 seconds
```

### 创建表

在test数据库下创建表tb1：

```shell
hive> create table test.tb1(
    > id int,
    > name string,
    > sex tinyint);
OK
Time taken: 0.52 seconds
```

### 查看表

查看当前数据库中的表：

```shell
hive> show tables;
OK
tb1
Time taken: 0.033 seconds, Fetched: 1 row(s)
```

查看指定数据库中的表：

```shell
hive> show tables in default;
OK
Time taken: 0.027 seconds
```

查看表结构：

```shell
hive> desc test.tb1;
OK
id                  	int
name                	string
sex                 	tinyint
Time taken: 0.228 seconds, Fetched: 3 row(s)
```

### 插入数据

显示插入数据：

```shell
hive> insert into tb1(id, name, sex) values(1, 'hmj', 1);
Query ID = root_20180824192203_e0fda5a8-43e0-4220-8638-1f87cebff71f
Total jobs = 3
Launching Job 1 out of 3
Number of reduce tasks is set to 0 since there's no reduce operator
Starting Job = job_1535097816443_0001, Tracking URL = http://Master:8088/proxy/application_1535097816443_0001/
Kill Command = /usr/local/hadoop/bin/hadoop job  -kill job_1535097816443_0001
Hadoop job information for Stage-1: number of mappers: 1; number of reducers: 0
2018-08-24 19:22:14,230 Stage-1 map = 0%,  reduce = 0%
2018-08-24 19:22:21,797 Stage-1 map = 100%,  reduce = 0%, Cumulative CPU 1.02 sec
MapReduce Total cumulative CPU time: 1 seconds 20 msec
Ended Job = job_1535097816443_0001
Stage-4 is selected by condition resolver.
Stage-3 is filtered out by condition resolver.
Stage-5 is filtered out by condition resolver.
Moving data to directory hdfs://Master:9000/user/hive/warehouse/test.db/tb1/.hive-staging_hive_2018-08-24_19-22-03_954_561109500998164964-1/-ext-10000
Loading data to table test.tb1
MapReduce Jobs Launched:
Stage-Stage-1: Map: 1   Cumulative CPU: 1.02 sec   HDFS Read: 4476 HDFS Write: 72 SUCCESS
Total MapReduce CPU Time Spent: 1 seconds 20 msec
OK
Time taken: 20.546 seconds
```

隐式插入数据：

```shell
hive> insert into tb1 values(2, 'Leo', 2);
Query ID = root_20180824192339_fa27b70d-4869-4698-8ba3-cb7e4e5c2840
Total jobs = 3
Launching Job 1 out of 3
Number of reduce tasks is set to 0 since there's no reduce operator
Starting Job = job_1535097816443_0002, Tracking URL = http://Master:8088/proxy/application_1535097816443_0002/
Kill Command = /usr/local/hadoop/bin/hadoop job  -kill job_1535097816443_0002
Hadoop job information for Stage-1: number of mappers: 1; number of reducers: 0
2018-08-24 19:23:48,843 Stage-1 map = 0%,  reduce = 0%
2018-08-24 19:23:54,250 Stage-1 map = 100%,  reduce = 0%, Cumulative CPU 0.97 sec
MapReduce Total cumulative CPU time: 970 msec
Ended Job = job_1535097816443_0002
Stage-4 is selected by condition resolver.
Stage-3 is filtered out by condition resolver.
Stage-5 is filtered out by condition resolver.
Moving data to directory hdfs://Master:9000/user/hive/warehouse/test.db/tb1/.hive-staging_hive_2018-08-24_19-23-39_116_2386772226148603446-1/-ext-10000
Loading data to table test.tb1
MapReduce Jobs Launched:
Stage-Stage-1: Map: 1   Cumulative CPU: 0.97 sec   HDFS Read: 4481 HDFS Write: 72 SUCCESS
Total MapReduce CPU Time Spent: 970 msec
OK
Time taken: 17.759 seconds
```

插入多条数据

```shell
hive> insert into tb1 values(3, 'ccc', 1),(4, 'ddd', 1),(5, 'eee', 2);
Query ID = root_20180824192530_2ba35a13-89f7-4e04-8a5a-513316bb458e
Total jobs = 3
Launching Job 1 out of 3
Number of reduce tasks is set to 0 since there's no reduce operator
Starting Job = job_1535097816443_0003, Tracking URL = http://Master:8088/proxy/application_1535097816443_0003/
Kill Command = /usr/local/hadoop/bin/hadoop job  -kill job_1535097816443_0003
Hadoop job information for Stage-1: number of mappers: 1; number of reducers: 0
2018-08-24 19:25:39,603 Stage-1 map = 0%,  reduce = 0%
2018-08-24 19:25:44,989 Stage-1 map = 100%,  reduce = 0%, Cumulative CPU 1.06 sec
MapReduce Total cumulative CPU time: 1 seconds 60 msec
Ended Job = job_1535097816443_0003
Stage-4 is selected by condition resolver.
Stage-3 is filtered out by condition resolver.
Stage-5 is filtered out by condition resolver.
Moving data to directory hdfs://Master:9000/user/hive/warehouse/test.db/tb1/.hive-staging_hive_2018-08-24_19-25-30_045_8743069087074347498-1/-ext-10000
Loading data to table test.tb1
MapReduce Jobs Launched:
Stage-Stage-1: Map: 1   Cumulative CPU: 1.06 sec   HDFS Read: 4499 HDFS Write: 88 SUCCESS
Total MapReduce CPU Time Spent: 1 seconds 60 msec
OK
Time taken: 17.548 seconds
```

可见，Hive每次向表中插入一条记录，都会转换为MapReduce程序，效率低下。

### 简单查询

字段查询：

```shell
hive> select id, name from tb1;
OK
1	hmj
2	Leo
3	ccc
4	ddd
5	eee
Time taken: 0.166 seconds, Fetched: 5 row(s)
```

条件查询：

```shell
hive> select * from tb1 where sex=2;
OK
2	Leo	2
5	eee	2
Time taken: 0.369 seconds, Fetched: 2 row(s)
```

统计查询：

```shell
hive> select max(id) from tb1;
Query ID = root_20180824193115_37f65de2-b102-4d16-b38a-f62c2f8dff65
Total jobs = 1
Launching Job 1 out of 1
Number of reduce tasks determined at compile time: 1
In order to change the average load for a reducer (in bytes):
  set hive.exec.reducers.bytes.per.reducer=<number>
In order to limit the maximum number of reducers:
  set hive.exec.reducers.max=<number>
In order to set a constant number of reducers:
  set mapreduce.job.reduces=<number>
Starting Job = job_1535097816443_0004, Tracking URL = http://Master:8088/proxy/application_1535097816443_0004/
Kill Command = /usr/local/hadoop/bin/hadoop job  -kill job_1535097816443_0004
Hadoop job information for Stage-1: number of mappers: 1; number of reducers: 1
2018-08-24 19:31:23,079 Stage-1 map = 0%,  reduce = 0%
2018-08-24 19:31:28,269 Stage-1 map = 100%,  reduce = 0%, Cumulative CPU 0.67 sec
2018-08-24 19:31:34,593 Stage-1 map = 100%,  reduce = 100%, Cumulative CPU 1.56 sec
MapReduce Total cumulative CPU time: 1 seconds 560 msec
Ended Job = job_1535097816443_0004
MapReduce Jobs Launched:
Stage-Stage-1: Map: 1  Reduce: 1   Cumulative CPU: 1.56 sec   HDFS Read: 8195 HDFS Write: 101 SUCCESS
Total MapReduce CPU Time Spent: 1 seconds 560 msec
OK
5
Time taken: 20.314 seconds, Fetched: 1 row(s)
```

**说明**

Hive简单查询不启用MapReduce Job 而启用Fetch task，在YARN Applications看不到ApplicationID。

启用MapReduce Job是会消耗系统开销的。对于这个问题，从Hive0.10.0版本开始，对于简单的不需要聚合的类似SELECT <col> from <table> LIMIT n语句，不需要启动MapReduce job，直接通过Fetch task获取数据。
