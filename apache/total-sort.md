# MapReduce实例：全排序

## 前言

- 集群环境：[Ubuntu16.04环境下安装配置Hadoop2.8.1集群](./installing-hadoop2.8.1-on-ubuntu.md)；
- 开发环境：[Windows环境下使用IDEA开发MapReduce程序](./developing-mapreduce-programs-using-idea-in-windows-environment.md)；

## 需求

对一批文本文档中数字进行排序。并将结果存放于不同的输出文件中，输出文件之间依然保证有序。

## 数据

存放于HDFS上的一批文本文档。每个文档有10000行，文档的每一行是一个数字，每个数字都来自于区间[0, 100000)。

## 思路

### 读取数据并排序

Map端读取文件中的数字，输出为中间结果：

```java
static class TotalSortMapper extends Mapper<LongWritable, Text, LongWritable, LongWritable> {
    @Override
    protected void map(LongWritable key, Text value, Context context) throws IOException, InterruptedException {
        String line = value.toString().trim();
        if (!"".equals(line))
            context.write(new LongWritable(Long.parseLong(line)), new LongWritable(1));
    }
}
```

Reduce端输出排序结果：

```java
static class TotalSortReducer extends Reducer<LongWritable, LongWritable, LongWritable, LongWritable> {
    static Long idx = 0L;
    @Override
    protected void reduce(LongWritable key, Iterable<LongWritable> values, Context context) throws IOException, InterruptedException {
        for (LongWritable value : values) {
            idx += value.get();
            context.write(new LongWritable(idx), key);
        }
    }
}
```

主函数中，创建相应任务：

```java    
public static void main(String[] args) throws IOException, ClassNotFoundException, InterruptedException {
    if (args.length != 3) {
        System.err.println("Usage: TotalSort <in> <out>");
        System.exit(2);
    }
    System.setProperty("HADOOP_USER_NAME", "root");
    Configuration configuration = new Configuration();
    configuration.set("fs.defaultFS", "hdfs://Master:9000");
    Job job = Job.getInstance(configuration);
    job.setJobName("TotalSort");
    job.setJarByClass(TotalSort.class);
    job.setMapperClass(TotalSortMapper.class);
    job.setMapOutputKeyClass(LongWritable.class);
    job.setMapOutputValueClass(LongWritable.class);
    job.setReducerClass(TotalSortReducer.class);
    job.setNumReduceTasks(2);
    job.setOutputKeyClass(LongWritable.class);
    job.setOutputValueClass(LongWritable.class);
    FileInputFormat.addInputPath(job, new Path(args[1]));
    FileOutputFormat.setOutputPath(job, new Path(args[2]));
    System.exit(job.waitForCompletion(true) ? 0 : 1);
}
```

执行任务后，由于设置的Reduce个数为2，所以在输出目录中出现两个文件。查看这两个文件的内容可以看到，文件内部的确是按照数字的升序进行排列，但是文件与文件间却不是有序的。

我们知道，MapReduce过程中，键值对被分配到哪一个分区上是由Partitioner决定，默认的Partitioner是按照Key的hashcode对Reduce个数取模得到。那如何使输出文件间也保持有序呢？这里提供两个方案：
1. 方案一：人工指定分区。
2. 方案二：使用TotalOrderPartitioner完成分区。

### 人工指定分区

人工指定分区相对比较简单，即集成Partitioner类，完成自定义分区：

```java
    static class TotalSortPartitioner extends Partitioner<LongWritable, LongWritable> {
        @Override
        public int getPartition(LongWritable longWritable, LongWritable longWritable2, int i) {
            if (i == 2)
                return longWritable.get() > 10000 ? 0 : 1;
            return 0;
        }
    }
```

这样就可以将大于10000的数字分到分区0，将小于10000的数字分到分区1。但是这样会出现一个严重的问题：数据倾斜。各个分区分得的数据量不一致，每个Reduce执行时间长度不一。

### 使用TotalOrderPartitioner完成分区

1. 在开始Map之前，Mapreduce首先执行InputSampler对样本抽样，并生成partition file写入HDFS。InputSampler对输入split进行抽样，并使用sortComparator对抽样结果进行排序。常用抽样方法有：
    - RandomSampler：按照给定频次，进行随机抽样。
    - IntervalSampler：按照给定间隔，进行定间隔抽样。
    - SplitSampler：取每个split的前n个样本进行抽样。
2. InputSampler在HDFS上写入一个partition file（sequence file），决定不同分区的key边界。对于n个Reducer，partition file有n-1个边界数据。Map的output按照partition file的边界不同，分别写入对应的分区。
3. Mapper使用TotalOrderPartitioner类读取partition file，获得每个Mapper使用TotalOrderPartitioner类。这个类读取partition file，确定每个分区的边界。
4. 在shuffle阶段，每个Reducer会拉取对应分区中已排序的(key, value)。由于每个分区已按照partition file设置边界，这样分区1中的数据都比分区2小，分区2数据都比分区3小（假设升序排列）。
5. Reducer处理对应分区数据并写入HDFS后，输出数据也保持全局有序。

```java
public static void main(String[] args) throws IOException, ClassNotFoundException, InterruptedException {
    if (args.length != 4) {
        System.err.println("Usage: TotalSort <in> <partition.file> <out>");
        System.exit(2);
    }
    System.setProperty("HADOOP_USER_NAME", "root");
    Configuration configuration = new Configuration();
    configuration.set("fs.defaultFS", "hdfs://Master:9000");
    Job job = Job.getInstance(configuration);
    job.setJobName("TotalSort");
    job.setJarByClass(TotalSort.class);
    job.setMapperClass(TotalSortMapper.class);
    job.setMapOutputKeyClass(LongWritable.class);
    job.setMapOutputValueClass(LongWritable.class);
    job.setPartitionerClass(TotalOrderPartitioner.class);
    job.setReducerClass(TotalSortReducer.class);
    job.setNumReduceTasks(3);
    job.setOutputKeyClass(LongWritable.class);
    job.setOutputValueClass(LongWritable.class);
    FileInputFormat.addInputPath(job, new Path(args[1]));
    FileOutputFormat.setOutputPath(job, new Path(args[3]));
    TotalOrderPartitioner.setPartitionFile(job.getConfiguration(), new Path(args[2]));
    InputSampler.RandomSampler<Object, Text> sampler = new InputSampler.RandomSampler<>(0.1, 1000, 10);
    InputSampler.writePartitionFile(job, sampler);
    System.exit(job.waitForCompletion(true) ? 0 : 1);
}
```
