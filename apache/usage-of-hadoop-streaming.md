# Hadoop Streaming使用入门
## 环境
- 操作系统：Ubuntu
- 系统版本：16.04.3 LTS
- Hadoop版本：2.8.1（安装请参考[Ubuntu16.04环境下安装配置Hadoop2.8.1集群](./installing-hadoop2.8.1-on-ubuntu.md)）
## 简介
Hadoop Streaming提供了一个便于进行MapReduce编程的工具包，使用它可以基于一些可执行命令、脚本语言或其他编程语言来实现Mapper和Reducer，从而充分利用Hadoop并行计算框架的优势和能力，来处理大数据。它提供了一种非常灵活的编程接口，是一种常用的非Java API编写MapReduce的工具。
## 原理
- Mapper和Reducer会从标准输入中读取用户数据，一行一行处理后发送给标准输出。Streaming工具会创建MapReduce作业，发送给各个tasktracker，同时监控整个作业的执行过程。
- 如果一个文件（可执行或者脚本）作为Mapper，在Mapper初始化时，每一个Mapper任务会把该文件作为一个单独进程启动，Mapper任务运行时，它把输入切分成行并把每一行提供给可执行文件进程的标准输入。 同时，Mapper收集可执行文件进程标准输出的内容，并把收到的每一行内容转化成key/value对，作为mapper的输出。 默认情况下，一行中第一个tab（制表符）之前的部分作为key，之后的（不包括tab）作为value。如果没有tab，整行作为key值，value值为null。
- 对于reducer，类似。
## 用法
```console
Usage: $HADOOP_PREFIX/bin/hadoop jar hadoop-streaming.jar [options]
```
1. 不同版本hadoop中hadoop-streaming.jar文件所在路径不同，hadoop2.8.1中所在的路径为`$HADOOP_PREFIX/share/hadoop/tools/lib/hadoop-streaming-2.8.1.jar`
2. options支持的常用选项包括：
  ```text
  -input		<path> Map阶段DFS上的输入路径，可以是多个输入路径。
  -output		<path> Reduce阶段DFS上的输出路径。
  -mapper		<cmd|JavaClassName> 可选的，要执行的Mapper。
  -combiner	<cmd|JavaClassName> 可选的，要执行的Combiner。
  -reducer	<cmd|JavaClassName> 可选的，要执行的Reducer。
  -file		<file> 可选的，打包分发在mapper或reducer中要用的文件，如配置文件、字典等。已淘汰，使用"-files"替换。
  -inputformat    <TextInputFormat(default)|SequenceFileAsTextInputFormat|JavaClassName> 可选的，输入的格式。
  -outputformat   <TextOutputFormat(default)|JavaClassName> 可选的，输出的格式。
  -partitioner    <JavaClassName>  可选的，用于分区的类。
  -numReduceTasks <num> 可选的，reduce的数量。
  -cmdenv         <n>=<v> 可选的，传递一个或多个环境变量。
  -mapdebug       <cmd> 可选的，当某个map任务失败时执行的脚本。
  -reducedebug    <cmd> 可选的，当某个reduce任务失败时执行的脚本。
  -lazyOutput     可选的，延迟创建输出。
  -background     可选的，提交任务并且不等待完成。
  -verbose        可选的，打印详细输出。
  -info           可选的，打印详细的使用说明。
  -help           可选的，打印帮助信息。
  ```
3. option支持的通用选项包括：
  ```text
  -conf		<configuration file> 应用配置文件。
  -D		<property=value> 作业的属性配置。
  -fs		<file:///|hdfs://namenode:port> 重写配置文件中的"fs.defaultFS"属性。
  -jt		<local|resourcemanager:port> 指定一个ResourceManager。
  -files		<comma separated list of files> 打包分发在mapper或reducer中要用的文件，如配置文件、字典等。
  -libjars	<comma separated list of jars> 打包分发在classpath中包含的jar文件。
  -archives	<comma separated list of archives> 提供自动解压的jar包。
  ```
4. 常见的作业属性配置：
  ```xml
  <configuration>

  <property>
  <name>mapred.job.name</name>
  <value></value>
  <description>The name of the job.</description>
  </property>

  <property>
  <name>mapred.mapper.class</name>
  <value>org.apache.hadoop.mapred.lib.IdentityMapper</value>
  <description>The full class name of the mapper.</description>
  </property>

  <property>
  <name>mapred.combiner.class</name>
  <value></value>
  <description>The full class name of the combiner.</description>
  </property>

  <property>
  <name>mapred.reducer.class</name>
  <value>org.apache.hadoop.mapred.lib.IdentityReducer</value>
  <description>The full class name of the reducer.</description>
  </property>

  <property>
  <name>mapred.jar</name>
  <value>No default.</value>
  <description>The full path to the jarfile containing all the needed classes.</description>
  </property>

  <property>
  <name>mapred.map.tasks</name>
  <value>1</value>
  <description>The default number of map tasks per job. Typically set to a prime several times greater than number of available hosts. Ignored when mapred.job.tracker is "local". </description>
  </property>

  <property>
  <name>mapred.reduce.tasks</name>
  <value>1</value>
  <description>The default number of reduce tasks per job. Typically set to a prime close to the number of available hosts. Ignored when mapred.job.tracker is "local". </description>
  </property>

  <property>
  <name>mapred.input.dir</name>
  <value></value>
  <description>A comma separated list of input directories.</description>
  </property>

  <property>
  <name>mapred.output.dir</name>
  <value></value>
  <description>A comma separated list of output directories.</description>
  </property>

  <property>
  <name>mapred.input.format.class</name>
  <value>org.apache.hadoop.mapred.TextInputFormat</value>
  <description>The full class name of the InputFormat class to be used for obtaining the input to the mapper.</description>
  </property>

  <property>
  <name>mapred.output.format.class</name>
  <value>org.apache.hadoop.mapred.TextOutputFormat</value>
  <description>The full class name of the OutputFormat class to be used for saving the output of the reducer.</description>
  </property>

  <property>
  <name>mapred.input.key.class</name>
  <value>org.apache.hadoop.io.LongWritable</value>
  <description>The full classname of the input key.</description>
  </property>

  <property>
  <name>mapred.input.value.class</name>
  <value>org.apache.hadoop.io.UTF8</value>
  <description>The full classname of the input value.</description>
  </property>

  <property>
  <name>mapred.output.key.class</name>
  <value>org.apache.hadoop.io.LongWritable</value>
  <description>The full classname of the output key.</description>
  </property>

  <property>
  <name>mapred.output.value.class</name>
  <value>org.apache.hadoop.io.UTF8</value>
  <description>The full classname of the output value.</description>
  </property>

  <property>
  <name>mapred.partitioner.class</name>
  <value>org.apache.hadoop.mapred.lib.HashPartitioner</value>
  <description>The full classname of the partitioner class.</description>
  </property>

  <property>
  <name>user.name</name>
  <value>Dr. Who</value>
  <description>The name of the user running the job.</description>
  </property>

  <property>
  <name>mapred.combine.buffer.size</name>
  <value>100000</value>
  <description>The number of entries the combining collector caches before combining them and writing to disk.</description>
  </property>

  <property>
  <name>mapred.speculative.execution</name>
  <value>true</value>
  <description>If true, then multiple instances of some map tasks may be executed in parallel.</description>
  </property>

  <property>
  <name>mapred.min.split.size</name>
  <value>0</value>
  <description>The minimum size chunk that map input should be split into. Note that some file formats may have minimum split sizes that take priority over this setting.</description>
  </property>

  </configuration>
  ```
## 示例
1. mapper.py
  ```python
  import sys

  def read_from_input(file):  
      for line in file:  
  	yield line.split(' ')

  def main(separator = ' '):  
      data = read_from_input(sys.stdin)  
      for words in data:  
  	for word in words: 
  	    print(word + "\t1")

  if __name__ == '__main__':  
      main()
  ```
2. reducer.py
  ```python
  import sys  
  from itertools import groupby  
  from operator import itemgetter  

  def read_from_mapper(file, separator):  
      for line in file:  
  	yield line.strip().split(separator, 2)  

  def main(separator = '\t'):  
      data = read_from_mapper(sys.stdin, separator)  
      for current_word, group in groupby(data, itemgetter(0)):  
  	try:  
  	    total_count = sum(int(count) for current_word, count in group)  
  	    print "%s%s%d" % (current_word, separator, total_count)  
  	except ValueError:  
  	    pass  

  if __name__ == '__main__':  
      main()  
  ```
3. 位于dfs上`/wordcount/words`的输入文件：
  ```text
  Hello Hadoop Hello Spark Hello Python Hello Pyspark
  ```
4. 执行命令：
  ```console
  hadoop jar $HADOOP_PREFIX/share/hadoop/tools/lib/hadoop-streaming-2.8.1.jar -input /wordcount/words -output /wordcount/output -file mapper.py -file reducer.py -mapper "python mapper.py" -reducer "python reducer.py" -jobconf mapreduce.reduce.tasks=1 -jobconf mapreduce.job.name="hs-test"
  ```
5. 位于dfs上`/wordcount/output/part-00000`的输出文件：
  ```text
  Hadoop	1
  Hello	4
  Python	1
  Spark	1
  ```
