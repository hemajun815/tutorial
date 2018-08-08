# MapReduce实例：Map端多表链接

## 前言

- 集群环境：[Ubuntu16.04环境下安装配置Hadoop2.8.1集群](./installing-hadoop2.8.1-on-ubuntu.md)；
- 开发环境：[Windows环境下使用IDEA开发MapReduce程序](./developing-mapreduce-programs-using-idea-in-windows-environment.md)；

## 需求

现有三个文件，文件一存储人员信息（人员编号、人员名称和性别），文件二存储网站信息（网站编号、网站名称和网站类别），文件三存储访问记录（记录编号、人员编号、网站编号和记录时间）。要求编写MapReduce程序，实现在Reduce端对三个文件进行链接，输出人员名称和网站名称以及指定时间段内的访问次数。

## 数据

- hdfs://Master:9000/CacheTableJoin/input/user.txt前10条数据
```text
id,name,sex
1,user_01,male
2,user_02,female
3,user_03,male
4,user_04,female
5,user_05,male
6,user_06,male
7,user_07,male
8,user_08,male
9,user_09,female
10,user_10,female
```

- hdfs://Master:9000/CacheTableJoin/input/web.txt前10条数据
```text
id,name,type
1,www.web001.com,E-commerce
2,www.web002.com,E-commerce
3,www.web003.com,Social Network
4,www.web004.com,Search
5,www.web005.com,E-commerce
6,www.web006.com,Search
7,www.web007.com,Search
8,www.web008.com,E-commerce
9,www.web009.com,Search
10,www.web010.com,Search
```

- hdfs://Master:9000/CacheTableJoin/input/record.txt前10条数据
```text
id,user_id,web_id,date
1,6,194,2017-6-11
2,12,107,2017-6-27
3,11,97,2017-6-8
4,2,52,2017-6-18
5,6,16,2017-6-14
6,12,28,2017-6-15
7,10,50,2017-6-5
8,11,21,2017-6-1
9,1,18,2017-6-15
10,4,61,2017-6-24
```

## 思路

将数据量较小的user.txt和web.txt文件缓存在工作节点本地，map阶段直接与record.txt文件的相应字段做连接。然后reduce阶段统计规定时间段内的访问次数。

