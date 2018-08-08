# MapReduce实例：单表链接

## 前言

- 集群环境：[Ubuntu16.04环境下安装配置Hadoop2.8.1集群](./installing-hadoop2.8.1-on-ubuntu.md)；
- 开发环境：[Windows环境下使用IDEA开发MapReduce程序](./developing-mapreduce-programs-using-idea-in-windows-environment.md)；

## 需求

输入为两列数据，第一列Parent，第二列Child，要求编写MapReduce程序输出两列数据：GrandParent name和GrandChild name。

## 数据

- hdfs://Master:9000/SingleTableJoin/input/table.txt
  ```text
  parent name,child name
  Tom,Nancy
  Tony,Mike
  Nancy,Tony
  Tony,Jim
  ```
