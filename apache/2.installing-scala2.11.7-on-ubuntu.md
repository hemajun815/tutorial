# Ubuntu16.04环境下Scala-2.11.7的安装与配置
## 前言
Spark提供了对Scala语言的支持。在此，选择scala-2.11.7版本进行安装与配置。在此之前，已经完成了[Ubuntu16.04环境下安装配置Hadoop2.8.1集群](https://github.com/hemajun815/tutorial/blob/master/apache/installing-hadoop2.8.1-on-ubuntu.md)。
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
1. scala-2.11.7.tgz [百度网盘](https://pan.baidu.com/s/1gfzmIT9)
## 安装
1. 下载[scala-2.11.7.tgz](https://pan.baidu.com/s/1gfzmIT9)到/root/tmp/文件夹下；
2. 解压文件：`tar -zxvf scala-2.11.7.tgz -C /usr/local/`；
3. 重命名文件：`mv /usr/local/scala-2.11.7 /usr/local/scala`；
4. 配置环境变量：`vim ~/.bashrc`，添加内容如下：
	```text
	export SCALA_HOME=/usr/local/scala
	export PATH=$PATH:$SCALA_HOME/bin
	```
5. 使变量生效：`source ~/.bashrc`；
6. 查看版本号，以验证安装成功：`scala -version`；
7. 同步文件到Slave1：`rsync -av /usr/local/scala/ Slave1:/usr/local/scala/`；
8. 同步文件到Slave2：`rsync -av /usr/local/scala/ Slave2:/usr/local/scala/`；
## 说明
在两台linux主机之间复制文件的方式有很多种，之前我们使用的是`scp`命令，这次我们使用的是`rsync`命令，都能达到复制的目的。
