# Ubuntu16.04环境下Spark-2.2.0的安装和部署
## 前言
在此之前，已经完成了[Ubuntu16.04环境下安装配置Hadoop2.8.1集群](https://github.com/hemajun815/tutorial/blob/master/apache/installing-hadoop2.8.1-on-ubuntu.md)和[Ubuntu16.04环境下Scala-2.11.7的安装与配置](https://github.com/hemajun815/tutorial/blob/master/apache/installing-scala2.11.7-on-ubuntu.md)。
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
## 注意
1. 安装与配置过程中的所有操作均在Master节点上完成。
## 软件下载
1. spark-2.2.0-bin-hadoop2.7.tgz [百度网盘](https://pan.baidu.com/s/1kUDb6ar)
## 安装
1. 下载[spark-2.2.0-bin-hadoop2.7.tgz](https://pan.baidu.com/s/1kUDb6ar)到/root/tmp/文件夹下；
2. 解压文件：`tar -zxvf ./spark-2.2.0-bin-hadoop2.7.tgz -C /usr/local/`；
3. 重命名文件：`mv /usr/local/spark-2.2.0-bin-hadoop2.7/ /usr/local/spark/`；
## 部署
1. 进入spark根目录：`cd $SPARK_HOME`；
2. 使用命令`cp conf/spark-env.sh.template conf/spark-env.sh`复制文件，并配置新复制的文件，`vim conf/spark-env.sh`，内容如下：
	```text
	export JAVA_HOME=/usr/local/jdk				#Java根目录
	export SCALA_HOME=/usr/local/scala			#Scala根目录
	export HADOOP_HOME=/usr/local/hadoop			#Hadoop根目录
	export HADOOP_CONF_DIR=/usr/local/hadoop/etc/hadoop	#Hadoop集群配置文件的目录 
	export SPARK_MASTER_IP=Master				#Spark集群的Master节点的地址
	export SPARK_WORKER_MEMORY=2g				#每个Worker节点能够最大分配给exectors的内存大小 
	export SPARK_WORKER_CORES=2				#每个Worker节点所占有的CPU核数目
	export SPARK_WORKER_INSTANCES=1				#每台机器上开启的Worker节点的数目
	```
3. 使用命令`cp conf/slaves.template conf/slaves`复制文件，并配置新复制的文件，`vim conf/slaves`，内容如下：
	```text
	Slave1
	Slave2
	```
4. 同步文件到Slave1：`rsync -av /usr/local/spark/ Slave1:/usr/local/spark/`；
5. 同步文件到Slave2：`rsync -av /usr/local/spark/ Slave2:/usr/local/spark/`；
## 启动
```console
$ /usr/local/hadoop/sbin/start-dfs.sh
$ /usr/local/spark/sbin/start-all.sh
```
## 停止
```console
$ /usr/local/spark/sbin/stop-all.sh
$ /usr/local/hadoop/sbin/stop-dfs.sh
```
