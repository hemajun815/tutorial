# Ubuntu16.04环境下安装配置Hive2.3.3

## 前言

在此之前，已经完成了[Ubuntu16.04环境下安装配置Hadoop2.8.1集群](./installing-hadoop2.8.1-on-ubuntu.md)和[Ubuntu16.04环境下安装配置HBase2.1.0集群](./installing-hbase2.1.0-on-ubuntu.md)。

## 设备配置

- 系统：Ubuntu
- 版本：16.04
- 处理器：Intel(R) Core(TM) i7-7700K CPU @ 4.20GHz
- 内存：2.0GB
- 类型：64位操作系统 64位处理器

## 软件下载

- Hive-2.3.3：【[百度网盘](https://pan.baidu.com/s/150WsPwzfyK8LRVHTvbuDIQ)】

## Hadoop环境

- 版本：Hadoop-2.8.1、HBase-2.1.0
- Master：192.168.73.130
- Slave1：192.168.73.132
- Slave2：192.168.73.133

## 安装与配置

1. 将下载好的Hive安装包上传到Master节点；
2. 解压文件：`tar -zxf apache-hive-2.3.3-bin.tar.gz -C /usr/local/`；
3. 跳转目录并重命名文件夹：`cd /usr/local/ && mv apache-hive-2.3.3-bin hive && cd hive`；
4. 拷贝文件：`cp ./conf/hive-env.sh.template ./conf/hive-env.sh`；
5. 编辑文件：`vim ./conf/hive-env.sh`；

    ```shell
    # Hadoop所在文件夹
    HADOOP_HOME=/usr/local/hadoop
    ```
6. 初始化：`./bin/schematool -initSchema -dbType derby`；

## 测试

- 命令`/usr/local/hive/bin/hive --version`可查看hive版本号。
- 命令`/usr/local/hive/bin/hive`可进入hive shell。
