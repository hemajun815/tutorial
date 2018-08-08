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

## 程序

```java
package hemajun.mapred.example;

import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.fs.Path;
import org.apache.hadoop.io.LongWritable;
import org.apache.hadoop.io.NullWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Job;
import org.apache.hadoop.mapreduce.Mapper;
import org.apache.hadoop.mapreduce.Reducer;
import org.apache.hadoop.mapreduce.lib.input.FileInputFormat;
import org.apache.hadoop.mapreduce.lib.input.TextInputFormat;
import org.apache.hadoop.mapreduce.lib.output.FileOutputFormat;
import org.apache.hadoop.mapreduce.lib.output.TextOutputFormat;

import java.io.IOException;
import java.util.ArrayList;

public class SingleTableJoin {
    public static class SingleTableJoinMapper extends Mapper<LongWritable, Text, Text, Text> {
        @Override
        protected void map(LongWritable key, Text value, Context context) throws IOException, InterruptedException {
            String[] elements = value.toString().split("[, ]");
            // 同一行数据交替输出两次。
            if (elements.length == 2) {
                context.write(new Text(elements[0]), new Text(elements[1] + ",c"));
                context.write(new Text(elements[1]), new Text(elements[0] + ",p"));
            }
        }
    }

    public static class SingleTableJoinReducer extends Reducer<Text, Text, Text, NullWritable> {
        @Override
        protected void setup(Context context) throws IOException, InterruptedException {
            // 输出表头。
            context.write(new Text("grandparent name,grandchild name"), null);
        }

        @Override
        protected void reduce(Text key, Iterable<Text> values, Context context) throws IOException, InterruptedException {
            ArrayList<String> grandparents = new ArrayList<>();
            ArrayList<String> grandchildren = new ArrayList<>();
            for (Text text : values) {
                String[] elements = text.toString().split("[,]");
                if (elements.length == 2) {
                    if ("p".equals(elements[1])) {
                        grandparents.add(elements[0]);
                    }
                    else {
                        grandchildren.add(elements[0]);
                    }
                }
            }
            for (String grandparent : grandparents) {
                for (String grandchild : grandchildren) {
                    context.write(new Text(grandparent + "," + grandchild), null);
                }
            }
        }
    }

    public static void main(String[] args) throws IOException, ClassNotFoundException, InterruptedException {
        if (args.length != 3) {
            System.err.println("Usage: SingleTableJoin <in> <out>");
            System.exit(2);
        }
        System.setProperty("HADOOP_USER_NAME", "root");
        Configuration configuration = new Configuration();
        configuration.set("fs.defaultFS", "hdfs://Master:9000");
        Job job = Job.getInstance(configuration);
        job.setJobName("SingleTableJoin");
        job.setJarByClass(SingleTableJoin.class);
        job.setMapperClass(SingleTableJoinMapper.class);
        job.setReducerClass(SingleTableJoinReducer.class);
        // 设置Map输出格式。
        job.setMapOutputKeyClass(Text.class);
        job.setMapOutputValueClass(Text.class);
        // 设置Reduce输出格式。
        job.setOutputKeyClass(Text.class);
        job.setOutputValueClass(NullWritable.class);
        // 设置输入格式。
        job.setInputFormatClass(TextInputFormat.class);
        job.setOutputFormatClass(TextOutputFormat.class);
        // 设置输入输出路径
        FileInputFormat.addInputPath(job, new Path(args[1]));
        FileOutputFormat.setOutputPath(job, new Path(args[2]));
        System.exit(job.waitForCompletion(true) ? 0 : 1);
    }
}
```

## 执行结果

- hdfs://Master:9000/SingleTableJoin/output/part-r-00000.txt
  ```text
  grandparent name,grandchild name
  Tom,Tony
  Nancy,Jim
  Nancy,Mike
  ```
