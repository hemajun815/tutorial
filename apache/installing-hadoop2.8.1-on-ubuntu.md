# Ubuntu16.04环境下安装配置Hadoop2.8.1集群
## 网络环境
本次安装使用了三台设备进行集群的搭建，IP地址如下：
- 192.168.73.130
- 192.168.73.132
- 192.168.73.133
## 设备配置
- 系统：Ubuntu
- 版本：16.04
- 处理器：Intel(R) Core(TM) i7-7700K CPU @ 4.20GHz
- 内存：2.0GB
- 类型：64位操作系统 64位处理器
## 安装说明
- 本次安装皆是在root用户下进行操作。可参照[opening-root-on-ubuntu](https://github.com/hemajun815/tutorial/blob/master/ubuntu/opening-root-on-ubuntu.md)启用root用户。
## 软件下载
1. jdk-8u131-linux-x64.tar.gz [百度网盘](https://pan.baidu.com/s/1c25ywcK)
2. hadoop-2.8.1.tar.gz [百度网盘](https://pan.baidu.com/s/1c1C9MgC)
## 安装
### 配置主机名
#### Master
在192.168.73.130主机上，输入命令`vim /etc/hostname`编辑/etc/hostname文件，清空文件中的内容，输入Master，保存，退出，重启系统。
#### Salve1
在192.168.73.132主机上，输入命令`vim /etc/hostname`编辑/etc/hostname文件，清空文件中的内容，输入Slave1，保存，退出，重启系统。
#### Salve2
在192.168.73.133主机上，输入命令`vim /etc/hostname`编辑/etc/hostname文件，清空文件中的内容，输入Slave2，保存，退出，重启系统。
#### Matser、Salve1、Salve2
在三台主机上，输入命令`vim /etc/hosts`编辑/etc/hosts文件，在文件最前面加上以下内容：
```text
# Cluster
192.168.73.130 Master
192.168.73.132 Slave1
192.168.73.133 Slave2
```
保存，退出。
### 安装ssh服务
#### ssh说明
SSH之所以能够保证安全，原因在于它采用了公钥加密。过程如下：
1. 远程主机收到用户的登录请求，把自己的公钥发给用户；
2. 用户使用这个公钥，将登录密码加密后，发送回来；
3. 远程主机用自己的私钥，解密登录密码，如果密码正确，就同意用户登录。
#### 安装
在三台主机上，均输入`apt-get install openssh-server`命令安装ssh服务。安装完成后，由于ssh默认不允许root用户登录，可参考[ssh-with-root](https://github.com/hemajun815/tutorial/blob/master/ubuntu/ssh-with-root.md)允许root用户连接。
### 配置ssh服务
#### 说明
Hadoop运行过程中需要管理远端Hadoop守护进程，在Hadoop启动以后，NameNode是通过SSH（Secure Shell）来启动和停止各个DataNode上的各种守护进程的。这就必须在节点之间执行指令的时候是不需要输入密码的形式，故我们需要配置SSH运用无密码公钥认证的形式，这样NameNode使用SSH无密码登录并启动DataName进程，同样原理，DataNode上也能使用SSH无密码登录到 NameNode。
#### 配置
1. 【三台主机】首次运行命令`ssh localhost`产生/root/.ssh/目录；
2. 【Master】进入目录：`cd /root/.ssh/`；
3. 【Master】产生无密公钥：`ssh-keygen -t rsa`；
4. 【Master】将id_rsa.pub追加到authorized_keys：`cat ./id_rsa.pub >> ./authorized_keys`；
5. 【Master】将id_rsa.pub文件发送到其他主机`scp ./id_rsa.pub Slave1:/root/tmp`、`scp ./id_rsa.pub Slave2:/root/tmp`;
6. 【Slave1】将id_rsa.pub追加到authorized_keys：`cat /root/tmp/id_rsa.pub >> /root/.ssh/authorized_keys`；
7. 【Slave2】将id_rsa.pub追加到authorized_keys：`cat /root/tmp/id_rsa.pub >> /root/.ssh/authorized_keys`；
8. 此时Master已经能够无密ssh连接Slave1和Slave2了，然后再在Slave1和Slave2上重复步骤2～7，使得各主机之间均能相互ssh无密码连接。
### 安装与配置jdk
1. 【三台主机】[下载jdk-8u131-linux-x64.tar.gz](https://pan.baidu.com/s/1c25ywcK)到/root/tmp/文件夹中；
2. 【三台主机】解压文件：`tar -zxvf /root/tmp/jdk-8u131-linux-x64.tar.gz -C /usr/local/`；
3. 【三台主机】编辑环境变量文件：`vim /root/.bashrc`；
4. 【三台主机】在.bashrc文件最顶部输入以下内容，保存文件；
	```text
	export JAVA_HOME=/usr/local/jdk1.8.0_131
	export JRE_HOME=${JAVA_HOME}/jre
	export CLASSPATH=.:${JAVA_HOME}/lib:${JRE_HOME}/lib
	export PATH=.:${JAVA_HOME}/bin:$PATH
	```
5. 【三台主机】让配置生效：`source /root/.bashrc`；
6. 【三台主机】查看jdk版本信息以验证安装：`java -version`。
### 安装与配置Hadoop
1. 【Master】[下载hadoop-2.8.1.tar.gz](https://pan.baidu.com/s/1c1C9MgC)到/root/tmp/文件夹中；
2. 【Master】解压文件：`tar -zxvf /root/tmp/hadoop-2.8.1.tar.gz -C /usr/local/`；
3. 【Master】重命名文件夹：`mv /usr/local/hadoop-2.8.1 /usr/local/hadoop`；
4. 【Master】进入Hadoop文件夹：`cd /usr/local/hadoop/`；
5. 【Master】配置slaves文件，`vim etc/hadoop/slaves`，内容如下：
	```text
	Slave1
	Slave2
	```
6. 【Master】配置core-site.xml文件，`vim etc/hadoop/core-site.xml`，内容如下：
	```xml
	<configuration>
		<property>
		    <name>fs.defaultFS</name>
		    <value>hdfs://Master:9000</value>
		</property>
		<property>
		    <name>hadoop.tmp.dir</name>
		    <value>file:/usr/local/hadoop/tmp</value>
		    <description>Abase for other temporary directories.</description>
		</property>
	</configuration>
	```
7. 【Master】配置hdfs-site.xml文件，`vim etc/hadoop/hdfs-site.xml`，内容如下：
	```xml
	<configuration>
		<property>
		    <name>dfs.namenode.secondary.http-address</name>
		    <value>Master:50090</value>
		</property>
		<property>
		    <name>dfs.namenode.name.dir</name>
		    <value>file:/usr/local/hadoop/tmp/dfs/name</value>
		</property>
		<property>
		    <name>dfs.datanode.data.dir</name>
		    <value>file:/usr/local/hadoop/tmp/dfs/data</value>
		</property>
		<property>
		    <name>dfs.replication</name>
		    <value>2</value>
		</property>
	</configuration>
	```
8. 【Master】使用命令`cp etc/hadoop/mapred-site.xml.template etc/hadoop/mapred-site.xml`复制文件，并配置新复制的文件，`vim etc/hadoop/mapred-site.xml`，内容如下：
	```xml
	<configuration>
		<property>
		    <name>mapreduce.framework.name</name>
		    <value>yarn</value>
		</property>
	</configuration>
	```
9. 【Master】配置yarn-site.xml文件，`vim etc/hadoop/yarn-site.xml`，内容如下：
	```xml
	<configuration>
		<!-- Site specific YARN configuration properties -->
		<property>
		    <name>yarn.resourcemanager.hostname</name>
		    <value>Master</value>
		</property>
		<property>
		    <name>yarn.nodemanager.aux-services</name>
		    <value>mapreduce_shuffle</value>
		</property>
	</configuration>
	```
10. 配置完成，将Hadoop文件夹打包，复制到其他节点上，解压：
	```console
	# Master
	$ cd /usr/local/
	$ tar -zcvf ./hadoop.tar.gz ./hadoop 
	$ scp ./hadoop.tar.gz Slave1:/root/tmp
	$ scp ./hadoop.tar.gz Slave2:/root/tmp
	# Slave1 & Slave2
	$ tar -zcvf /root/tmp/hadoop.tar.gz /usr/local
	```
11. 首次启动前，格式化hdfs：
	```console
	# Master
	$ cd /usr/local
	$ ./hadoop/bin/hdfs namenode -format
	```
## 验证安装
1. 【Master】启动Hadoop：
	```console
	$ cd /usr/local/hadoop
	$ sbin/start-dfs.sh
	$ sbin/start-yarn.sh
	```
2. 【Master】查看运行状态：
	```console
	$ /usr/local/hadoop/bin/hdfs dfsadmin -report
	```
3. 【Master】停止Hadoop：
	```console
	$ cd /usr/local/hadoop
	$ sbin/stop-yarn.sh
	$ sbin/stop-dfs.sh
	```
