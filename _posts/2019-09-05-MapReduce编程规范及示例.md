---
layout:     post
title:      MapReduce编程规范及示例
subtitle:   MapReduce的开发分为八个步骤，其中map阶段分为2个步骤，shuffle阶段分为4个步骤，reduce阶段分为2个步骤
date:       2019-09-05
author:     caosw
header-img: img/001.jpg
catalog: true
tags:
    - Hadoop
    - MapReduce
---
# MapReduce编程规范及示例
***
### 规范

`Map阶段2个步骤`
<br>第一步、读取文件，解析成key，value对，这里的key，value对指代的是k1，v1。
<br>第二步、自定义map逻辑，接收第一步读取的k1，v1，转换成新的 k2，v2输出。

`shuffle阶段4个步骤（非重点）`
<br>第三步、分区   相同key的数据发送到同一个reduce里面去，形成一个集合，这里的key指代的是k2。
<br>第四步、排序   对数据进行字典顺序的排序。
<br>第五步、规约   主要是在map端对数据做一次聚合，减少输出的 k2的数据量。
<br>第六步、分组   将相同的数据发送到同一组里面去调用一次reduce逻辑。

`reduce阶段2个步骤`
<br>第七步、自定义reduce逻辑，接收k2，v2转换成新的k3，v3进行输出。
<br>第八步、输出，将reduce处理完成之后的数据进行输出。


### WordCount示例

需求在给定的文件文件中统计输出每一个单词出现的总次数

##### 1、准备

    [root@hadoop /root]# vi wordcount.txt
    hello,world,hadoop
    hive,sqoop,flume,hello
    kitty,tom,jerry,world
    hadoop,caosw,JLBlob
    [root@hadoop /root]# hdfs dfs -mkdir /wordcount
    [root@hadoop /root]# hdfs dfs -put wordcount.txt /wordcount

##### 2、构建maven工程

pom.xml文件如下

    <dependencies>
        <dependency>
            <groupId>org.apache.hadoop</groupId>
            <artifactId>hadoop-hdfs</artifactId>
            <version>2.7.7</version>
        </dependency>
        <dependency>
            <groupId>org.apache.hadoop</groupId>
            <artifactId>hadoop-client</artifactId>
            <version>2.7.7</version>
        </dependency>
        <dependency>
            <groupId>org.apache.hadoop</groupId>
            <artifactId>hadoop-common</artifactId>
            <version>2.7.7</version>
        </dependency>
    </dependencies>
    <build>
        <plugins>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-compiler-plugin</artifactId>
                <version>3.7.0</version>
                <configuration>
                    <source>1.8</source>
                    <target>1.8</target>
                    <encoding>UTF-8</encoding>
                </configuration>
            </plugin>
        </plugins>
    </build>

##### 3、定义一个mapper类

    package com.epoint.mapper;

    import java.io.IOException;

    import org.apache.hadoop.io.LongWritable;
    import org.apache.hadoop.io.Text;
    import org.apache.hadoop.mapreduce.Mapper;

    public class WordCountMapper extends Mapper<LongWritable,  Text, Text, LongWritable>{
        @Override
        public void map(LongWritable key, Text value,  Mapper<LongWritable, Text, Text, LongWritable>.Context  context) 
         throws IOException, InterruptedException {
            String line = value.toString();
            String[] split = line.split(",");
            for (String word : split) {
                context.write(new Text(word), new  LongWritable(1));
            }
        }
    }

##### 4、定义reducer类

    package com.epoint.reduce;

    import java.io.IOException;

    import org.apache.hadoop.io.LongWritable;
    import org.apache.hadoop.io.Text;
    import org.apache.hadoop.mapreduce.Reducer;

    public class WordCountReducer extends Reducer<Text,  LongWritable, Text, LongWritable>{
        /**
         * 自定义reduce逻辑
         * 所有的key代表单词，所有的values表示单词出现的次数
         */
        @Override
        public  void reduce(Text key, Iterable<LongWritable>  values,
              Reducer<Text, LongWritable, Text,  LongWritable>.Context context) throws IOException,  InterruptedException {
            long count = 0;
            for (LongWritable value : values) {
                count += value.get();
            }
            context.write(key, new LongWritable(count));
        }
    }

##### 5、定义一个主类，用来表述job并提交job

    package com.epoint.main;

    import org.apache.hadoop.conf.Configuration;
    import org.apache.hadoop.conf.Configured;
    import org.apache.hadoop.fs.Path;
    import org.apache.hadoop.io.LongWritable;
    import org.apache.hadoop.io.Text;
    import org.apache.hadoop.mapreduce.Job;
    import  org.apache.hadoop.mapreduce.lib.input.TextInputFormat;
    import  org.apache.hadoop.mapreduce.lib.output.TextOutputFormat;
    import org.apache.hadoop.util.Tool;
    import org.apache.hadoop.util.ToolRunner;

    import com.epoint.mapper.WordCountMapper;
    import com.epoint.reduce.WordCountReducer;
    public class JobMain extends Configured implements Tool{
        @Override
        public int run(String[] args) throws Exception {
            Job job = Job.getInstance(super.getConf(),  JobMain.class.getSimpleName());
            // 打包到集群上运行的时候必须添加如下配置，指定程序的main函数
            job.setJarByClass(JobMain.class);
            // 第一步：读取输入文件解析成key，value对
            job.setInputFormatClass(TextInputFormat.class);
            TextInputFormat.addInputPath(job, new  Path("hdfs://192.168.206.204:8020/wordcount"));
          
            //第二步：设置mapper类
            job.setMapperClass(WordCountMapper.class);
            //设置map阶段完成之后的输出类型
            job.setMapOutputKeyClass(Text.class);
            job.setMapOutputValueClass(LongWritable.class);
          
            //第三步至第六步省略
            //第七步：设置reduce类
            job.setReducerClass(WordCountReducer.class);
            //设置reduce阶段完成之后的输出类型
            job.setOutputKeyClass(Text.class);
            job.setOutputValueClass(LongWritable.class);
          
            //第八步：设置输出类以及输出路径
            job.setOutputFormatClass(TextOutputFormat.class);
            TextOutputFormat.setOutputPath(job, new  Path("hdfs://192.168.206.204:8020/wordcount_out"));
            boolean b = job.waitForCompletion(true);
            return b?0:1;
        }
     
        public static void main(String[] args) throws  Exception {
            Configuration configuration = new  Configuration();
            Tool tool = new JobMain();
            int run = ToolRunner.run(configuration, tool,  args);
            System.exit(run);
        }
    }

##### 6、运行程序，查看结果

    [root@hadoop /root]# hdfs dfs -ls /
    Found 7 items
    drwxr-xr-x   - root  supergroup          0 2019-09-04 15:51 /benchmarks
    drwxr-xr-x   - root  supergroup          0 2019-09-04 10:05 /caosw
    drwxr-xr-x   - caosw supergroup          0 2019-09-04 17:49 /hello
    drwx------   - root  supergroup          0 2019-09-04 09:56 /tmp
    drwxr-xr-x   - root  supergroup          0 2019-09-04 15:29 /user
    drwxr-xr-x   - root  supergroup          0 2019-09-05 10:35 /wordcount
    drwxr-xr-x   - caosw supergroup          0 2019-09-05 10:58 /wordcount_out
    [root@hadoop /root]# hdfs dfs -ls /wordcount_out
    Found 2 items
    -rw-r--r--   3 caosw supergroup          0 2019-09-05 10:58 /wordcount_out/_SUCCESS
    -rw-r--r--   3 caosw supergroup         90 2019-09-05 10:58 /wordcount_out/part-r-00000
    [root@hadoop /root]# hdfs dfs -get /wordcount_out/part-r-00000 /root
    [root@hadoop /root]# more part-r-00000
            1
    JLBlob  1
    caosw   1
    flume   1
    hadoop  2
    hello   2
    hive    1
    jerry   1
    kitty   1
    sqoop   1
    tom     1
    world   2
