# MapReduce实例：二次排序

## 前言

- 集群环境：[Ubuntu16.04环境下安装配置Hadoop2.8.1集群](./installing-hadoop2.8.1-on-ubuntu.md)；
- 开发环境：[Windows环境下使用IDEA开发MapReduce程序](./developing-mapreduce-programs-using-idea-in-windows-environment.md)；

## 需求

现有一个文件，存储内容为网站域名、用户名、访问次数。现要求编写MapReduce程序，实现对文件内容的排序输出，先按照用户名称升序排列，用户名称相同的按照访问次数降序排列。

## 数据

- hdfs://Master:9000/SecondarySort/input/record.txt部分数据

```text
www.web161.com	user_06	28
www.web161.com	user_07	21
www.web161.com	user_08	24
www.web161.com	user_09	24
www.web161.com	user_10	20
www.web161.com	user_11	20
www.web161.com	user_12	14
www.web161.com	user_13	35
www.web161.com	user_14	26
www.web161.com	user_15	18
www.web161.com	user_16	27
www.web161.com	user_17	24
```

## 思路

通过设置自定义Key完成二次排序。

