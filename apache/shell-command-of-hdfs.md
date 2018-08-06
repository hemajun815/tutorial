# HDFS 的 Shell 基本操作

## 前言

在此之前，已经完成了[Ubuntu16.04环境下安装配置Hadoop2.8.1集群](./installing-hadoop2.8.1-on-ubuntu.md)。

## 设备配置

- 系统：Ubuntu
- 版本：16.04
- 处理器：Intel(R) Core(TM) i7-7700K CPU @ 4.20GHz
- 内存：2.0GB
- 类型：64位操作系统 64位处理器


## Hadoop环境

- 版本：Hadoop-2.8.1
- Master：192.168.73.130
- Slave1：192.168.73.132
- Slave2：192.168.73.133

## Shell 命令

HDFS 的 Shell 命令都是通过 `hadoop fs` 来进行执行的，在命令行输入此命令可以看到完整的 Usage 如下：

```console
root@Master:/usr/local/hadoop# ./bin/hadoop fs
Usage: hadoop fs [generic options]
	[-appendToFile <localsrc> ... <dst>]
	[-cat [-ignoreCrc] <src> ...]
	[-checksum <src> ...]
	[-chgrp [-R] GROUP PATH...]
	[-chmod [-R] <MODE[,MODE]... | OCTALMODE> PATH...]
	[-chown [-R] [OWNER][:[GROUP]] PATH...]
	[-copyFromLocal [-f] [-p] [-l] [-d] <localsrc> ... <dst>]
	[-copyToLocal [-f] [-p] [-ignoreCrc] [-crc] <src> ... <localdst>]
	[-count [-q] [-h] [-v] [-t [<storage type>]] [-u] [-x] <path> ...]
	[-cp [-f] [-p | -p[topax]] [-d] <src> ... <dst>]
	[-createSnapshot <snapshotDir> [<snapshotName>]]
	[-deleteSnapshot <snapshotDir> <snapshotName>]
	[-df [-h] [<path> ...]]
	[-du [-s] [-h] [-x] <path> ...]
	[-expunge]
	[-find <path> ... <expression> ...]
	[-get [-f] [-p] [-ignoreCrc] [-crc] <src> ... <localdst>]
	[-getfacl [-R] <path>]
	[-getfattr [-R] {-n name | -d} [-e en] <path>]
	[-getmerge [-nl] [-skip-empty-file] <src> <localdst>]
	[-help [cmd ...]]
	[-ls [-C] [-d] [-h] [-q] [-R] [-t] [-S] [-r] [-u] [<path> ...]]
	[-mkdir [-p] <path> ...]
	[-moveFromLocal <localsrc> ... <dst>]
	[-moveToLocal <src> <localdst>]
	[-mv <src> ... <dst>]
	[-put [-f] [-p] [-l] [-d] <localsrc> ... <dst>]
	[-renameSnapshot <snapshotDir> <oldName> <newName>]
	[-rm [-f] [-r|-R] [-skipTrash] [-safely] <src> ...]
	[-rmdir [--ignore-fail-on-non-empty] <dir> ...]
	[-setfacl [-R] [{-b|-k} {-m|-x <acl_spec>} <path>]|[--set <acl_spec> <path>]]
	[-setfattr {-n name [-v value] | -x name} <path>]
	[-setrep [-R] [-w] <rep> <path> ...]
	[-stat [format] <path> ...]
	[-tail [-f] <file>]
	[-test -[defsz] <path>]
	[-text [-ignoreCrc] <src> ...]
	[-touchz <path> ...]
	[-truncate [-w] <length> <path> ...]
	[-usage [cmd ...]]

Generic options supported are
-conf <configuration file>     specify an application configuration file
-D <property=value>            use value for given property
-fs <file:///|hdfs://namenode:port> specify default filesystem URL to use, overrides 'fs.defaultFS' property from configurations.
-jt <local|resourcemanager:port>    specify a ResourceManager
-files <comma separated list of files>    specify comma separated files to be copied to the map reduce cluster
-libjars <comma separated list of jars>    specify comma separated jar files to include in the classpath.
-archives <comma separated list of archives>    specify comma separated archives to be unarchived on the compute machines.

The general command line syntax is
command [genericOptions] [commandOptions]
```

## 基本操作示例

1. 在 HDFS 上新建文件夹 /input ：
   
   ```console
   root@Master:/usr/local/hadoop# ./bin/hadoop fs -mkdir /input
   ```

2. 上传本地文件到 HDFS ：
   
   ```console
   root@Master:/usr/local/hadoop# ./bin/hadoop fs -put ~/test.txt /input
   ```

3. 列出 HDFS 指定目录下的所有内容：
   
   ```console
   root@Master:/usr/local/hadoop# ./bin/hadoop fs -ls /input
   ```

4. 查看 HDFS 上文件内容：
   
   ```console
   root@Master:/usr/local/hadoop# ./bin/hadoop fs -cat /input/test.txt
   ```

5. 在 HDFS 上复制文件：
   
   ```console
   root@Master:/usr/local/hadoop# ./bin/hadoop fs -cp /input/test.txt /input/test-copy.txt
   ```

6. 下载 HDFS 上文件到本地：
   
   ```console
   root@Master:/usr/local/hadoop# ./bin/hadoop fs -get /input/test-copy.txt ~/
   ```

7. 合并 HDFS 上的多个文件到本地：
   
   ```console
   root@Master:/usr/local/hadoop# ./bin/hadoop fs -getmerge /input/* ~/test-merge.txt
   ```

8. 删除 HDFS 上的文件：
   
   ```console
   root@Master:/usr/local/hadoop# ./bin/hadoop fs -rm /input/test-copy.txt
   ```

9. 删除 HDFS 上的文件夹：
   
   ```console
   root@Master:/usr/local/hadoop# ./bin/hadoop fs -rm -r /input
   ```
