# MapReduce示例：Reduce端多表级联

## 前言

- 集群环境：[Ubuntu16.04环境下安装配置Hadoop2.8.1集群](./installing-hadoop2.8.1-on-ubuntu.md)；
- 开发环境：[Windows环境下使用IDEA开发MapReduce程序](./developing-mapreduce-programs-using-idea-in-windows-environment.md)；

## 需求

现有两个文件，文件一存储人名和学校编号，文件二存储学校编号和学校名称，要求编写MapReduce程序，实现在Reduce端对两个文件进行链接，输出人名和对应学校名。

## 数据

- hdfs://Master:9000/ReduceMultiTableJoin/input/user.txt部分内容
  ```text
  uname,sid
  Tom,10001
  Nancy,10001
  Jim,10002
  Leo,10003
  ```

- hdfs://Master:9000/ReduceMultiTableJoin/input/school.txt部分内容
  ```text
  10001,school_001
  10002,school_002
  10003,school_003
  ```
