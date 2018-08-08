# Windows环境下使用IDEA开发MapReduce程序

## 前言

在此之前，已经完成了[Ubuntu16.04环境下安装配置Hadoop2.8.1集群](./installing-hadoop2.8.1-on-ubuntu.md)。本教程则是要在Windows环境下搭建MapReduce开发环境，远程调用建立好的集群执行MR程序。

## 环境

- 系统：Windows 10 Enterprise
- 版本：1803
- 处理器：Intel(R) Core(TM) i7-7700K CPU @ 4.20GHz
- 内存：16.0GB
- 类型：64位操作系统 64位处理器

## 软件下载

- JDK【[下载地址](http://www.oracle.com/technetwork/java/javase/downloads/jdk8-downloads-2133151.html)】（不一定与集群中的JDK版本一致，集群中安装的是jdk-8u131，而本次教程时下载的是jdk-8u181。）
- Hadoop【[百度网盘](https://pan.baidu.com/s/1c1C9MgC)】（需要与集群中Hadoop版本一致，但无需配置。解压即可。）
- winutils【[下载地址](https://github.com/steveloughran/winutils)】（选择与Hadoop版本对应的文件下载。）
- IDEA【[下载地址](https://www.jetbrains.com/idea/download/)】（社区版与专业版皆可。）

## 配置开发环境

**安装JDK**

1. 运行下载的JDK安装文件，选择安装路径，执行安装；
2. 打开系统高级设置，设置环境变量；
3. 添加环境变量`JAVA_HOME`路径设置到JDK安装目录；
4. 将`%JAVA_HOME%\bin\`添加到Path变量中；
5. 打开命令行窗口，执行`java -version`命令，命令行显示java的版本信息则表示安装成功。

**安装Hadoop**

1. 解压下载好的Hadoop文件；
2. 打开系统高级设置，设置环境变量；
3. 添加环境变量`HADOOP_HOME`路径设置到Hadoop根目录；
4. 将`%HADOOP_HOME%\bin\`添加到Path变量中。

**安装winutils**

1. 解压下载好的winutils文件；
2. 将其中与Hadoop版本（本教程中是Hadoop2.8.1）对应的文件拷贝到`%HADOOP_HOME%\bin\`下。

**安装IDEA**

1. 运行下载的IDEA安装程序，选择安装路径，执行安装。

## 开发MapReduce程序

1. 启动IDEA；
2. 新建项目，选择JAVA项目，选择JDK，设置项目名称和项目目录，完成；
3. 在Project视图中，鼠标右击项目，选择`Open Module Settings`；
4. 选择`Libraries`，点击`New Project Library`，选择Java；
5. 在弹出框中选择`%HADOOP_HOME%\share\hadoop\`下的所有文件夹，点击`OK`；
6. 点击`Add`，选择`%HADOOP_HOME%\share\hadoop\common\`下的`lib`文件夹，点击`OK`；
7. 修改Name为`Hadoop`，点击`OK`；
8. 在项目中src文件夹上右击选择`New Package`，设置包名（如hemajun.mapred.example），点击`OK`；
9. 在新建的包上右击选择`New Class`，设置类名（如WordCount），点击`OK`；
10. 编辑代码如下：
    ```java
    package hemajun.mapred.example;

    import org.apache.hadoop.conf.Configuration;
    import org.apache.hadoop.fs.Path;
    import org.apache.hadoop.io.LongWritable;
    import org.apache.hadoop.io.Text;
    import org.apache.hadoop.mapreduce.Job;
    import org.apache.hadoop.mapreduce.Mapper;
    import org.apache.hadoop.mapreduce.Reducer;
    import org.apache.hadoop.mapreduce.lib.input.FileInputFormat;
    import org.apache.hadoop.mapreduce.lib.input.TextInputFormat;
    import org.apache.hadoop.mapreduce.lib.output.FileOutputFormat;
    import org.apache.hadoop.mapreduce.lib.output.TextOutputFormat;

    import java.io.IOException;

    public final class WordCount {
        public static class WordCountMapper extends Mapper<LongWritable, Text, Text, LongWritable> {
            @Override
            protected void map(LongWritable key, Text value, Mapper<LongWritable, Text, Text, LongWritable>.Context context)
                    throws IOException, InterruptedException {
                String[] words = value.toString().split("[,. ]");
                for (String word : words) {
                    if (!"".equals(word)) {
                        context.write(new Text(word), new LongWritable(1));
                    }
                }
            }
        }

        public static class WordCountReducer extends Reducer<Text, LongWritable, Text, LongWritable> {
            @Override
            protected void reduce(Text key, Iterable<LongWritable> values,
                                Reducer<Text, LongWritable, Text, LongWritable>.Context context) throws IOException, InterruptedException {
                long sum = 0;
                for (LongWritable value : values) {
                    sum += value.get();
                }
                context.write(key, new LongWritable(sum));
            }
        }

        public static void main(String[] args) throws Exception {
            if (args.length != 3) {
                System.err.println("Usage : WordCount <in> <out>");
                System.exit(2);
            }
            System.setProperty("HADOOP_USER_NAME", "root");
            Configuration configuration = new Configuration();
            configuration.set("fs.defaultFS", "hdfs://192.168.73.130:9000");
            Job job = Job.getInstance(configuration, "WordCount");
            job.setJarByClass(WordCount.class);
            job.setMapperClass(WordCountMapper.class);
            job.setCombinerClass(WordCountReducer.class);
            job.setReducerClass(WordCountReducer.class);
            job.setNumReduceTasks(2);
            job.setOutputKeyClass(Text.class);
            job.setOutputValueClass(LongWritable.class);
            job.setInputFormatClass(TextInputFormat.class);
            job.setOutputFormatClass(TextOutputFormat.class);
            FileInputFormat.addInputPath(job, new Path(args[1]));
            FileOutputFormat.setOutputPath(job, new Path(args[2]));
            System.exit(job.waitForCompletion(true) ? 0 : 1);
        }
    }
    ```
11. 菜单栏点击`Run`，选择`Edit Configurations...`；
12. 在弹出窗口中点击`Add New Configuration`，选择`Application`；
13. 设置名称为"Hadoop"，选择主类为刚刚定义的主类（如hemajun.mapred.example.WordCount），设置程序参数为"WorCount hdfs://192.168.73.130:9000/WordCount/input hdfs://192.168.73.130:9000/WordCount/output"；
14. 集群上启动Hadoop：`./sbin/start-dfs.sh && ./sbin/start-yarn.sh`；
15. 新建输入文件夹：`./bin/hadoop fs -mkdir /WordCount/ && ./bin/hadoop fs -mkdir /WordCount/input/`（更多HDFS操作可参考[HDFS的Shell基本操作](./shell-command-of-hdfs.md)）；
16. 新建输入文件：`echo "Hello Hadoop" > text.txt`；
17. 将输入文件上传到HDFS：`./bin/hadoop fs -put ./text.txt /WordCount/input/`（更多HDFS操作可参考[HDFS的Shell基本操作](./shell-command-of-hdfs.md)）；
18. 回到IDEA，菜单栏点击`Run`，选择`Run 'Hadoop'`，可看到程序正在编译执行，Run视图中可看到运行日志；
19. 程序运行完成后，可在HDFS上查看相应的运行日志和运行结果。
