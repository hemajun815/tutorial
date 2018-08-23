# HBase的Shell基本操作

## 前言

在此之前，已经完成了[Ubuntu16.04环境下安装配置Hadoop2.8.1集群](./installing-hadoop2.8.1-on-ubuntu.md)和[Ubuntu16.04环境下安装配置HBase2.1.0集群](./installing-hbase2.1.0-on-ubuntu.md)。

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

## 连接HBase Shell

输入命令 `/usr/local/hbase/bin/hbase shell` 进入HBase的shell控制台。

```shell
root@Master:~# /usr/local/hbase/bin/hbase shell
HBase Shell
Use "help" to get list of supported commands.
Use "exit" to quit this interactive shell.
Version 2.1.0, re1673bb0bbfea21d6e5dba73e013b09b8b49b89b, Tue Jul 10 17:26:48 CST 2018
Took 0.0117 seconds
hbase(main):001:0>
```

查看版本信息

```shell
hbase(main):001:0> version
2.1.0, re1673bb0bbfea21d6e5dba73e013b09b8b49b89b, Tue Jul 10 17:26:48 CST 2018
Took 0.0097 seconds
```

查看状态信息

```shell
hbase(main):002:0> status
1 active master, 0 backup masters, 2 servers, 0 dead, 1.0000 average load
Took 0.5284 seconds
```

## 基本操作

### 命名空间

关系数据库系统中，命名空间namespace是表的逻辑分组，同一组中的表有类似的用途。hbase的表也有命名空间的管理方式，命名空间的概念为即将到来的多租户特性打下基础：

- 配额管理（ Quota Management (HBASE-8410)）：限制一个namespace可以使用的资源，资源包括region和table等；
- 命名空间安全管理（ Namespace Security Administration (HBASE-9206)）：提供了另一个层面的多租户安全管理；
- Region服务器组（Region server groups (HBASE-6721)）：一个命名空间或一张表，可以被固定到一组 regionservers上，从而保证了数据隔离性。

HBase系统默认定义了两个缺省的namespace：

- hbase：系统内建表，包括namespace和meta表；
- default：用户建表时未指定namespace的表都创建在此；

可以通过`list_namespace`命令查看命名空间：

```shell
hbase(main):003:0> list_namespace
NAMESPACE
default
hbase
2 row(s)
Took 0.0588 seconds
```

创建命名空间：

```shell
hbase(main):005:0> create_namespace 'hmjdb'
Took 0.6101 seconds
hbase(main):006:0> list_namespace
NAMESPACE

default

hbase

hmjdb
3 row(s)
Took 0.0142 seconds
```

查看namespace结构：

```shell
hbase(main):001:0> describe_namespace 'hmjdb'
DESCRIPTION
{NAME => 'hmjdb'}
Took 0.4547 seconds
=> 1
```

删除命名空间

```shell
hbase(main):033:0> list_namespace
NAMESPACE
default
hbase
hmjdb
3 row(s)
Took 0.0202 seconds
hbase(main):034:0> drop_namespace 'hmjdb'
Took 0.4915 seconds
hbase(main):035:0> list_namespace
NAMESPACE
default
hbase
2 row(s)
Took 0.0165 seconds
```

### 数据表

列举表：

```shell
hbase(main):003:0> list
TABLE
0 row(s)
Took 0.0283 seconds
=> []
```

在指定命名空间中创建表：

```shell
hbase(main):004:0> create 'hmjdb:t1', 'cf1', 'cf2'
Created table hmjdb:t1
Took 2.4704 seconds
=> Hbase::Table - hmjdb:t1
hbase(main):005:0> list
TABLE
hmjdb:t1
1 row(s)
Took 0.0203 seconds
=> ["hmjdb:t1"]
```

查看表结构：

```shell
hbase(main):010:0> desc 'hmjdb:t1'
Table hmjdb:t1 is ENABLED
hmjdb:t1
COLUMN FAMILIES DESCRIPTION
{NAME => 'cf1', VERSIONS => '1', EVICT_BLOCKS_ON_CLOSE => 'false', NEW_VERSION_BEHAVIOR => 'false', KEEP_DELETED_CEL
LS => 'FALSE', CACHE_DATA_ON_WRITE => 'false', DATA_BLOCK_ENCODING => 'NONE', TTL => 'FOREVER', MIN_VERSIONS => '0',
 REPLICATION_SCOPE => '0', BLOOMFILTER => 'ROW', CACHE_INDEX_ON_WRITE => 'false', IN_MEMORY => 'false', CACHE_BLOOMS
_ON_WRITE => 'false', PREFETCH_BLOCKS_ON_OPEN => 'false', COMPRESSION => 'NONE', BLOCKCACHE => 'true', BLOCKSIZE =>
'65536'}
{NAME => 'cf2', VERSIONS => '1', EVICT_BLOCKS_ON_CLOSE => 'false', NEW_VERSION_BEHAVIOR => 'false', KEEP_DELETED_CEL
LS => 'FALSE', CACHE_DATA_ON_WRITE => 'false', DATA_BLOCK_ENCODING => 'NONE', TTL => 'FOREVER', MIN_VERSIONS => '0',
 REPLICATION_SCOPE => '0', BLOOMFILTER => 'ROW', CACHE_INDEX_ON_WRITE => 'false', IN_MEMORY => 'false', CACHE_BLOOMS
_ON_WRITE => 'false', PREFETCH_BLOCKS_ON_OPEN => 'false', COMPRESSION => 'NONE', BLOCKCACHE => 'true', BLOCKSIZE =>
'65536'}
2 row(s)
Took 0.2380 seconds
```

表是否存在：

```shell
hbase(main):011:0> exists 'hmjdb:t1'
Table hmjdb:t1 does exist
Took 0.0064 seconds
=> true
```

向表中添加数据：

```shell
hbase(main):012:0> put 'hmjdb:t1', 'rk1', 'cf1:name', 'hmj'
Took 0.1780 seconds
hbase(main):013:0> put 'hmjdb:t1', 'rk1', 'cf1:age', '24'
Took 0.0274 seconds
hbase(main):014:0> put 'hmjdb:t1', 'rk2', 'cf1:name', 'Leo'
Took 0.0307 seconds
hbase(main):015:0> put 'hmjdb:t1', 'rk2', 'cf1:sex', 'man'
Took 0.0146 seconds
```

查询数据：

```shell
hbase(main):016:0> get 'hmjdb:t1', 'rk1'
COLUMN                         CELL
 cf1:age                       timestamp=1535015201892, value=24
 cf1:name                      timestamp=1535015185368, value=hmj
1 row(s)
Took 0.0453 seconds
hbase(main):017:0> get 'hmjdb:t1', 'rk1', 'cf1:name'
COLUMN                         CELL
 cf1:name                      timestamp=1535015185368, value=hmj
1 row(s)
Took 0.0155 seconds
hbase(main):018:0> get 'hmjdb:t1', 'rk2', 'cf1:sex'
COLUMN                         CELL
 cf1:sex                       timestamp=1535015238935, value=man
1 row(s)
Took 0.0095 seconds
```

扫描表：

```shell
hbase(main):019:0> scan 'hmjdb:t1'
ROW                            COLUMN+CELL
 rk1                           column=cf1:age, timestamp=1535015201892, value=24
 rk1                           column=cf1:name, timestamp=1535015185368, value=hmj
 rk2                           column=cf1:name, timestamp=1535015228808, value=Leo
 rk2                           column=cf1:sex, timestamp=1535015238935, value=man
2 row(s)
Took 0.0233 seconds
```

删除整行数据：

```shell
hbase(main):020:0> deleteall 'hmjdb:t1', 'rk1'
Took 0.0225 seconds
hbase(main):021:0> scan 'hmjdb:t1'
ROW                            COLUMN+CELL
 rk2                           column=cf1:name, timestamp=1535015228808, value=Leo
 rk2                           column=cf1:sex, timestamp=1535015238935, value=man
1 row(s)
Took 0.0104 seconds
```

删除某列数据：

```shell
hbase(main):022:0> delete 'hmjdb:t1', 'rk2', 'cf1:sex'
Took 0.0218 seconds
hbase(main):023:0> scan 'hmjdb:t1'
ROW                            COLUMN+CELL
 rk2                           column=cf1:name, timestamp=1535015228808, value=Leo
1 row(s)
Took 0.0084 seconds
```

截断表：

```shell
hbase(main):024:0> count 'hmjdb:t1'
1 row(s)
Took 0.0500 seconds
=> 1
hbase(main):025:0> truncate 'hmjdb:t1'
Truncating 'hmjdb:t1' table (it may take a while):
Disabling table...
Truncating table...
Took 4.8113 seconds
hbase(main):026:0> count 'hmjdb:t1'
0 row(s)
Took 0.8707 seconds
=> 0
```

修改表结构：

```shell
hbase(main):027:0> disable 'hmjdb:t1'
Took 0.8133 seconds
hbase(main):028:0> alter 'hmjdb:t1', READONLY
Updating all regions with the new schema...
All regions updated.
Done.
Took 1.7364 seconds
hbase(main):029:0> enable 'hmjdb:t1'
Took 1.3371 seconds
```

删除表：

```shell
hbase(main):030:0> disable 'hmjdb:t1'
Took 1.3160 seconds
hbase(main):031:0> drop 'hmjdb:t1'
Took 0.5288 seconds
hbase(main):032:0> list
TABLE
0 row(s)
Took 0.0176 seconds
=> []
```
