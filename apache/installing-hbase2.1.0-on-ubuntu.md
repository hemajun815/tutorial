# Ubuntu16.04环境下安装配置HBase2.1.0集群

## 前言

在此之前，已经完成了[Ubuntu16.04环境下安装配置Hadoop2.8.1集群](./installing-hadoop2.8.1-on-ubuntu.md)。

## 设备配置

- 系统：Ubuntu
- 版本：16.04
- 处理器：Intel(R) Core(TM) i7-7700K CPU @ 4.20GHz
- 内存：2.0GB
- 类型：64位操作系统 64位处理器

## 软件下载

- HBase-2.1.0：【[百度网盘](https://pan.baidu.com/s/1AnOy-w5AUIJ9i1qZLs7F3A)】

## Hadoop环境

- 版本：Hadoop-2.8.1
- Master：192.168.73.130
- Slave1：192.168.73.132
- Slave2：192.168.73.133

## 安装与配置

1. 将下载好的HBase安装包上传到Master节点；
2. 解压文件：`tar -zxvf hbase-2.1.0-bin.tar.gz -C /usr/local/`；
3. 跳转目录并重命名文件夹：`cd /usr/local/ && mv hbase-2.1.0 hbase && cd hbase`；
4. 编辑文件：`vim ./conf/hbase-env.sh`:
   
  ```shell
  # 引入Java环境变量：
  export JAVA_HOME=/usr/local/jdk1.8.0_131
  # 使用自带Zookeeper：
  export HBASE_MANAGES_ZK=true
  ```

5. 配置文件：`vim ./conf/hbase-site.xml`:

  ```xml
  <configuration>
    <property> 
      <name>hbase.rootdir</name> <!-- hbase存放数据目录 -->
      <value>hdfs://Master:9000/hbase</value><!-- 端口要和Hadoop的fs.defaultFS端口一致-->
    </property> 
    <property> 
      <name>hbase.cluster.distributed</name> <!-- 是否分布式部署 -->
      <value>true</value> 
    </property>
    <property> 
      <name>hbase.zookeeper.quorum</name> <!-- list of zookooper -->
      <value>Master,Slave1,Slave2</value> 
    </property> 　
    <property>
      <name>hbase.zookeeper.property.dataDir</name> <!--zookooper配置、日志等的存储位置 -->
      <value>/usr/local/hbase/zookeeper</value>
    </property>
  </configuration>
  ```

6. 配置文件：`vim ./conf/regionservers`；

  ```shell
  Slave1
  Slave2
  ```

7. 将Master上配置好HBase文件拷贝到Slave1和Slave2上：
   
  ```shell
  cd ..
  scp -r ./hbase/ Slave1:/usr/local/
  scp -r ./hbase/ Slave2:/usr/local/
  ```

## 启动

1. 启动HDFS：`/usr/local/hadoop/sbin/start-dfs.sh`；
2. 启动YARN：`/usr/local/hadoop/sbin/start-yarn.sh`；
3. 启动HBase：`/usr/local/hbase/bin/start-hbase.sh`；

## 测试

- 使用`jps`查看进程，Master上多出`HMaster`和`HQuorumPeer`进程，Slave1和Slave2上多出`HRegionServer`和`HQuorumPeer`表明HBase启动完成。
- 浏览器访问 http://Master:16010 可查看HBase运行状态。
- 输入命令 `/usr/local/hbase/bin/hbase shell` 可进入HBase的shell接口。

## 停止

1. 停止HBase：`/usr/local/hbase/bin/stop-hbase.sh`；
2. 停止YARN：`/usr/local/hadoop/sbin/stop-yarn.sh`；
3. 停止HDFS：`/usr/local/hadoop/sbin/stop-dfs.sh`；
