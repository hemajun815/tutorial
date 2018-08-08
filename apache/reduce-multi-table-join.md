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
  sid,sname
  10001,school_001
  10002,school_002
  10003,school_003
  ```

## 程序

- ReduceMultiTableJoin.java
```java
package hemajun.mapred.example;

import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.fs.Path;
import org.apache.hadoop.io.NullWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Job;
import org.apache.hadoop.mapreduce.lib.input.MultipleInputs;
import org.apache.hadoop.mapreduce.lib.input.TextInputFormat;
import org.apache.hadoop.mapreduce.lib.output.FileOutputFormat;

import java.io.IOException;

public class ReduceMultiTableJoin {
    public static void main(String[] args) throws IOException, ClassNotFoundException, InterruptedException {
        if (args.length != 3) {
            System.err.println("Usage: ReduceMultiTableJoin <in> <out>");
            System.exit(2);
        }
        System.setProperty("HADOOP_USER_NAME", "root");
        Configuration configuration = new Configuration();
        configuration.set("fs.defaultFS", "hdfs://Master:9000");
        Job job = Job.getInstance(configuration);
        job.setJobName("ReduceMultiTableJoin");
        job.setJarByClass(ReduceMultiTableJoin.class);
        job.setReducerClass(JoinReducer.class);
        job.setMapOutputKeyClass(Text.class);
        job.setMapOutputValueClass(Text.class);
        job.setOutputKeyClass(Text.class);
        job.setOutputValueClass(NullWritable.class);
        MultipleInputs.addInputPath(job, new Path(String.format("%s/user.txt", args[1])), TextInputFormat.class, UserMapper.class);
        MultipleInputs.addInputPath(job, new Path(String.format("%s/school.txt", args[1])), TextInputFormat.class, SchoolMapper.class);
        FileOutputFormat.setOutputPath(job, new Path(args[2]));
        System.exit(job.waitForCompletion(true) ? 0 : 1);
    }
}
```
- UserMapper.java
```java
package hemajun.mapred.example;

import org.apache.hadoop.io.LongWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Mapper;

import java.io.IOException;
import java.util.regex.Pattern;

public class UserMapper extends Mapper<LongWritable, Text, Text, Text> {
    @Override
    protected void map(LongWritable key, Text value, Context context) throws IOException, InterruptedException {
        String[] elements = value.toString().split("[,]");
        Pattern pattern = Pattern.compile("^[0-9]+$");
        if (elements.length == 2 && pattern.matcher(elements[1]).matches()) {
            context.write(new Text(elements[1]), new Text(elements[0] + ",u"));
        }
    }
}
```
- SchoolMapper.java
```java
package hemajun.mapred.example;

import org.apache.hadoop.io.LongWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Mapper;

import java.io.IOException;
import java.util.regex.Pattern;

public class SchoolMapper extends Mapper<LongWritable, Text, Text, Text> {
    @Override
    protected void map(LongWritable key, Text value, Context context) throws IOException, InterruptedException {
        String[] elements = value.toString().split("[,]");
        Pattern pattern = Pattern.compile("^[0-9]+$");
        if (elements.length == 2 && pattern.matcher(elements[0]).matches()) {
            context.write(new Text(elements[0]), new Text(elements[1] + ",s"));
        }
    }
}
```
- JoinReducer.java
```java
package hemajun.mapred.example;

import org.apache.hadoop.io.NullWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Reducer;

import java.io.IOException;
import java.util.ArrayList;

public class JoinReducer extends Reducer<Text, Text, Text, NullWritable> {
    @Override
    protected void setup(Context context) throws IOException, InterruptedException {
        context.write(new Text("uname,sname"), null);
    }

    @Override
    protected void reduce(Text key, Iterable<Text> values, Context context) throws IOException, InterruptedException {
        ArrayList<String> users = new ArrayList<>();
        ArrayList<String> schools = new ArrayList<>();
        for (Text text : values) {
            String[] elements = text.toString().split("[,]");
            if (elements.length == 2) {
                if ("u".equals(elements[1])) users.add(elements[0]);
                else schools.add(elements[0]);
            }
        }
        for (String user : users)
            for (String school : schools)
                context.write(new Text(String.format("%s,%s", user, school)), null);
    }
}
```

## 执行结果

- hdfs://Master:9000/ReduceMultiTableJoin/output/part-r-00000
  ```text
  uname,sname
  Nancy,school_001
  Tom,school_001
  Jim,school_002
  Leo,school_003
  ```
