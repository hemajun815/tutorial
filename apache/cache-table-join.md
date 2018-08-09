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

## 程序

- CacheTableJoin.java

```java
package hemajun.mapred.example;

import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.fs.Path;
import org.apache.hadoop.io.LongWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Job;
import org.apache.hadoop.mapreduce.lib.input.FileInputFormat;
import org.apache.hadoop.mapreduce.lib.output.FileOutputFormat;

import java.io.IOException;
import java.net.URI;
import java.net.URISyntaxException;

public class CacheTableJoin {
    public static void main(String[] args) throws IOException, ClassNotFoundException, InterruptedException, URISyntaxException {
        if (args.length != 3) {
            System.err.println("Usage: CacheTableJoin <in> <out>");
            System.exit(2);
        }
        System.setProperty("HADOOP_USER_NAME", "root");
        Configuration configuration = new Configuration();
        configuration.set("fs.defaultFS", "hdfs://Master:9000");
        Job job = Job.getInstance(configuration);
        job.setJobName("CacheTableJoin");
        job.setJarByClass(CacheTableJoin.class);
        job.setMapperClass(Mapper.class);
        job.setMapOutputKeyClass(Text.class);
        job.setMapOutputValueClass(LongWritable.class);
        job.setCombinerClass(Reducer.class);
        job.setReducerClass(Reducer.class);
        job.setOutputKeyClass(Text.class);
        job.setOutputValueClass(LongWritable.class);
        job.addCacheFile(new URI(String.format("%s/user.txt", args[1])));
        job.addCacheFile(new URI(String.format("%s/web.txt", args[1])));
        FileInputFormat.setInputPaths(job, new Path(String.format("%s/record.txt", args[1])));
        FileOutputFormat.setOutputPath(job, new Path(args[2]));
        System.exit(job.waitForCompletion(true) ? 0 : 1);
    }
}
```

- Mapper.java

```java
package hemajun.mapred.example;

import org.apache.hadoop.fs.Path;
import org.apache.hadoop.io.LongWritable;
import org.apache.hadoop.io.Text;

import java.io.BufferedReader;
import java.io.File;
import java.io.FileReader;
import java.io.IOException;
import java.text.SimpleDateFormat;
import java.util.Date;
import java.util.HashMap;
import java.util.Map;
import java.util.regex.Pattern;

public class Mapper extends org.apache.hadoop.mapreduce.Mapper<LongWritable, Text, Text, LongWritable> {
    private Map<String, String> users = new HashMap<>();
    private Map<String, String> webs = new HashMap<>();
    private final Pattern pattern = Pattern.compile("^[0-9]+$");
    private final SimpleDateFormat simpleDateFormat = new SimpleDateFormat("yyyy-M-d");
    private final LongWritable one = new LongWritable(1);

    @Override
    protected void setup(Context context) throws IOException {
        Path[] paths = context.getLocalCacheFiles();
        String line;
        for (Path path : paths) {
            if (path.toString().contains("user.txt")) {
                BufferedReader bufferedReader = new BufferedReader(new FileReader(new File(path.toString())));
                while ((line = bufferedReader.readLine()) != null) {
                    String[] elements = line.split("[,]");
                    if (elements.length == 3 && pattern.matcher(elements[0]).matches())
                        users.put(elements[0], elements[1]);
                }
                bufferedReader.close();
            }
            else if (path.toString().contains("web.txt")) {
                BufferedReader bufferedReader = new BufferedReader(new FileReader(new File(path.toString())));
                while ((line = bufferedReader.readLine()) != null) {
                    String[] elements = line.split("[,]");
                    if (elements.length == 3 && pattern.matcher(elements[0]).matches())
                        webs.put(elements[0], elements[1]);
                }
                bufferedReader.close();
            }
        }
    }

    @Override
    protected void map(LongWritable key, Text value, Context context) {
        String[] elements = value.toString().split("[,]");
        if (elements.length == 4 && pattern.matcher(elements[0]).matches()) try {
            Date dateMin = simpleDateFormat.parse("2017-6-24");
            Date dateMax = simpleDateFormat.parse("2017-6-30");
            Date date = simpleDateFormat.parse(elements[3]);
            if (date.getTime() >= dateMin.getTime() && date.getTime() <= dateMax.getTime()) {
                String strKey = String.format("%s\t%s", webs.get(elements[2]), users.get(elements[1]));
                context.write(new Text(strKey), one);
            }
        } catch (Exception e) {
            e.printStackTrace();
        }
    }

    @Override
    protected void cleanup(Context context) {
        users.clear();
        webs.clear();
    }
}
```

- Reducer.java

```java
package hemajun.mapred.example;

import org.apache.hadoop.io.LongWritable;
import org.apache.hadoop.io.Text;

import java.io.IOException;

public class Reducer extends org.apache.hadoop.mapreduce.Reducer<Text, LongWritable, Text, LongWritable> {
    @Override
    protected void reduce(Text key, Iterable<LongWritable> values, Context context) throws IOException, InterruptedException {
        long sum = 0;
        for (LongWritable value : values)
            sum += value.get();
        context.write(key, new LongWritable(sum));
    }
}
```

## 执行结果

- hdfs://Master:9000/CacheTableJoin/output/part-r-00000部分数据

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

## 注

缓存机制是将文件缓存在Map工作节点本地，缓存路径是节点上的本地文件路径，故使用缓存机制时不能远程调试，须得将代码打包成Jar，然后发送到集群上面运行。本教程中代码在集群上的运行命令如下：

```shell
./bin/hadoop jar ./cache_table_join.jar CacheTableJoin hdfs://Master:9000/CacheTableJoin/input hdfs://Master:9000/CacheTableJoin/output
```
