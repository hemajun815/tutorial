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

## 程序

- SecondarySort.java

```java
package hemajun.mapred.example;

import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.fs.Path;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Job;
import org.apache.hadoop.mapreduce.lib.input.FileInputFormat;
import org.apache.hadoop.mapreduce.lib.output.FileOutputFormat;

import java.io.IOException;

public class SecondarySort {
    public static void main(String[] args) throws IOException, ClassNotFoundException, InterruptedException {
        if (args.length != 3) {
            System.err.println("Usage: SecondarySort <in> <out>");
            System.exit(2);
        }
        System.setProperty("HADOOP_USER_NAME", "root");
        Configuration configuration = new Configuration();
        configuration.set("fs.defaultFS", "hdfs://Master:9000");
        Job job = Job.getInstance(configuration);
        job.setJobName("SecondarySort");
        job.setJarByClass(SecondarySort.class);
        job.setMapperClass(SecondarySortMapper.class);
        job.setMapOutputKeyClass(KeyPair.class);
        job.setMapOutputValueClass(Text.class);
        job.setReducerClass(SecondarySortReducer.class);
        job.setOutputKeyClass(Text.class);
        job.setOutputValueClass(Text.class);
        FileInputFormat.addInputPath(job, new Path(args[1]));
        FileOutputFormat.setOutputPath(job, new Path(args[2]));
        System.exit(job.waitForCompletion(true) ? 0 : 1);
    }
}
```

- SecondarySortMapper.java

```java
package hemajun.mapred.example;

import org.apache.hadoop.io.LongWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Mapper;

import java.io.IOException;

public class SecondarySortMapper extends Mapper<LongWritable, Text, KeyPair, Text> {
    @Override
    protected void map(LongWritable key, Text value, Context context) throws IOException, InterruptedException {
        String[] elements = value.toString().split("[ \t]");
        if (elements.length == 3) {
            KeyPair keyPair = new KeyPair();
            keyPair.setUserName(elements[1]);
            keyPair.setCount(Integer.parseInt(elements[2]));
            context.write(keyPair, new Text(elements[0]));
        }
    }
}
```

- SecondarySortReducer.java

```java
package hemajun.mapred.example;

import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Reducer;

import java.io.IOException;

public class SecondarySortReducer extends Reducer<KeyPair, Text, Text, Text> {
    @Override
    protected void reduce(KeyPair key, Iterable<Text> values, Context context) throws IOException, InterruptedException {
        for (Text text : values) {
            context.write(new Text(key.getUserName()), new Text(String.format("%s\t%d", text.toString(), key.getCount())));
        }
    }
}
```

- KeyPair.java

```java
package hemajun.mapred.example;

import org.apache.hadoop.io.WritableComparable;

import java.io.DataInput;
import java.io.DataOutput;
import java.io.IOException;

public class KeyPair implements WritableComparable<KeyPair> {
    private String userName;
    private int count;

    public String getUserName() {
        return this.userName;
    }

    public void setUserName(String userName) {
        this.userName = userName;
    }

    public int getCount() {
        return this.count;
    }

    public void setCount(int count) {
        this.count = count;
    }

    public KeyPair()
    {
        this.setUserName("");
        this.setCount(0);
    }

    @Override
    public int compareTo(KeyPair keyPair) {
        int ret = this.getUserName().compareTo(keyPair.getUserName());
        if (ret == 0) {
            ret = keyPair.getCount() - this.getCount();
        }
        return ret;
    }

    @Override
    public void write(DataOutput dataOutput) throws IOException {
        dataOutput.writeUTF(this.getUserName());
        dataOutput.writeInt(this.getCount());
    }

    @Override
    public void readFields(DataInput dataInput) throws IOException {
        this.setUserName(dataInput.readUTF());
        this.setCount(dataInput.readInt());
    }

    @Override
    public boolean equals(Object o) {
        if (o == null)
            return false;
        if (this.getClass() != o.getClass())
            return false;
        KeyPair keyPair = (KeyPair)o;
        return this.getUserName().equals(keyPair.getUserName());
    }

    @Override
    public int hashCode() {
        return this.getUserName().hashCode() & Integer.MAX_VALUE;
    }
}
```


## 执行结果

- hdfs://Master:9000/SecondarySort/output/part-r-00000

```text
user_20	www.web044.com	15
user_20	www.web110.com	15
user_20	www.web068.com	14
user_20	www.web154.com	14
user_20	www.web016.com	14
user_20	www.web060.com	14
user_20	www.web105.com	14
user_20	www.web026.com	14
user_20	www.web058.com	13
user_20	www.web198.com	13
```
