## MapReduce读写parquet格式文件的几种方式

### 摘要

本文将介绍常用parquet文件读写的几种方式

- 1.用spark的hadoopFile api读取hive中的parquet格式文件
- 2.用sparkSql读写hive中的parquet格式
- 3.用新旧MapReduce读写parquet格式文件

### 读parquet文件

读parquet文件
```
create table test(name string,age int) 
 row format delimited
 fields terminated by '\t';
```

加载数据

```
load data local inpath '/home/work/test/ddd.txt' into table test;
```
数据样例格式：
```
hive> select * from test limit 5; 
OK
leo 27
jim 38
leo 15
jack    22
jay 7
Time taken: 0.101 seconds, Fetched: 5 row(s)
```
创建parquet格式表

> create table test_parquet(name string,age int) stored as parquet

查看表结构

```
hive> show create table test_parquet;
OK
CREATE TABLE `test_parquet`(
  `name` string, 
  `age` int)
ROW FORMAT SERDE 
  'org.apache.hadoop.hive.ql.io.parquet.serde.ParquetHiveSerDe'
STORED AS INPUTFORMAT 
  'org.apache.hadoop.hive.ql.io.parquet.MapredParquetInputFormat'
OUTPUTFORMAT 
  'org.apache.hadoop.hive.ql.io.parquet.MapredParquetOutputFormat'
LOCATION
  'hdfs://localhost:9000/user/hive/warehouse/test_parquet'
TBLPROPERTIES (
  'transient_lastDdlTime'='1495038003')

```

可以看到数据的inputFormat是MapredParquetInputFormat，之后我们将用这个类来解析数据文件
往parquet格式表中插入数据

> insert into table test_parquet select * from test;

### a.用spark中hadoopFile api解析hive中parquet格式文件

如果是用spark-shell中方式读取文件一定要将hive-exec-0.14.0.jar加入到启动命令行中（MapredParquetInputFormat在这个jar中），还有就是要指定序列化的类，启动命令行如下

```
spark-shell \
--master spark://xiaobin:7077 \
--jars /home/xiaobin/soft/apache-hive-0.14.0-bin/lib/hive-exec-0.14.0.jar \
--conf spark.serializer=org.apache.spark.serializer.KryoSerializer
```

具体读取代码如下

```
import org.apache.hadoop.hive.ql.io.parquet.MapredParquetInputFormat
import org.apache.hadoop.hive.ql.io.parquet.MapredParquetInputFormat
import org.apache.hadoop.io.{ArrayWritable, NullWritable, Text}
import org.apache.hadoop.io.{ArrayWritable, NullWritable, Text}

val file =sc.hadoopFile("hdfs://localhost:9000/user/hive/warehouse/test_parquet/000000_0",
     classOf[MapredParquetInputFormat],classOf[Void],classOf[ArrayWritable])

file.take(10).foreach{case(k,v)=>
      val writables = v.get()
      val name = writables(0)
      val age = writables(1)
      println(writables.length+"    "+name+"   "+age)
    }

```

用MapredParquetInputFormat解析hive中parquet格式文件，每行数据将会解析成一个key和value，这里的key是空值，value是一个ArrayWritable，value的长度和表的列个数一样，value各个元素对应hive表中行各个字段的值

### b.用spark DataFrame 解析parquet文件

```
val conf = new SparkConf().setAppName("test").setMaster("local")
val sc = new SparkContext(conf)
val sqlContext = new org.apache.spark.sql.SQLContext(sc)
val parquet: DataFrame =
     sqlContext.read.parquet("hdfs://192.168.1.115:9000/user/hive/warehouse/test_parquet")
parquet.printSchema()
parquet.select(parquet("name"), parquet("age") + 1).show()
 
root
 |-- name: string (nullable = true)
 |-- age: integer (nullable = true)
 
+----+---------+
|name|(age + 1)|
+----+---------+
| leo|       28|
| jim|       39|
| leo|       16|
|jack|       23|
| jay|        8|
| jim|       38|
|jack|       37|
| jay|       12|

```

### c.用hivesql直接读取hive表

在local模式下没有测试成功，打包用spark-submit测试，代码如下

```

val conf = new SparkConf().setAppName("test")
val sc = new SparkContext(conf)
val hiveContext = new HiveContext(sc)
val sql: DataFrame = hiveContext.sql("select * from test_parquet limit 10")
sql.take(10).foreach(println)
 
[leo,27]                                                                        
[jim,38]
[leo,15]
[jack,22]
[jay,7]
[jim,37]
[jack,36]
[jay,11]
[leo,35]

```

提交任务命令行

> spark-submit --class quickspark.QuickSpark02 --master spark://192.168.1.115:7077 sparkcore-1.0-SNAPSHOT.jar

## 写parquet文件

### a.用spark写parquet文件

```

val conf = new SparkConf().setAppName("test").setMaster("local")
val sc = new SparkContext(conf)
val sqlContext = new org.apache.spark.sql.SQLContext(sc)
 
//    读取文件生成RDD
val file = sc.textFile("hdfs://192.168.1.115:9000/test/user.txt")
 
 //定义parquet的schema，数据字段和数据类型需要和hive表中的字段和数据类型相同，否则hive表无法解析
val schema = (new StructType)
      .add("name", StringType, true)
      .add("age", IntegerType, false)
 
val rowRDD = file.map(_.split("\t")).map(p => Row(p(0), Integer.valueOf(p(1).trim)))
//    将RDD装换成DataFrame
val peopleDataFrame = sqlContext.createDataFrame(rowRDD, schema)
peopleDataFrame.registerTempTable("people")
    peopleDataFrame.write.parquet("hdfs://192.168.1.115:9000/user/hive/warehouse/test_parquet/")
```
用hivesql读取用spark DataFrame生成的parquet文件

```
hive> select * from test_parquet limit 10;
OK
SLF4J: Failed to load class "org.slf4j.impl.StaticLoggerBinder".
SLF4J: Defaulting to no-operation (NOP) logger implementation
SLF4J: See http://www.slf4j.org/codes.html#StaticLoggerBinder for further details.
leo 27
jim 38
leo 15
jack    22
jay 7
jim 37
jack    36
jay 11
leo 35
leo 33

```
### b.用MapReduce写parquet文件

用MR读写parquet文件，刚开始打算使用hive中指定的org.apache.hadoop.hive.ql.io.parquet.MapredParquetOutputFormat这个类，但是这个类的getRecordWriter方法没实现，直接抛出异常

```
Override
public RecordWriter<Void, ArrayWritable> getRecordWriter(
    final FileSystem ignored,
    final JobConf job,
    final String name,
    final Progressable progress
    ) throws IOException {
  throw new RuntimeException("Should never be used");
}
```

所以使用官方提供的parquet解析方式，github地址：https://github.com/apache/parquet-mr/，导入依赖

```
 <dependency>
      <groupId>org.apache.parquet</groupId>
      <artifactId>parquet-common</artifactId>
      <version>1.8.1</version>
  </dependency>
  <dependency>
      <groupId>org.apache.parquet</groupId>
      <artifactId>parquet-encoding</artifactId>
      <version>1.8.1</version>
  </dependency>
  <dependency>
      <groupId>org.apache.parquet</groupId>
      <artifactId>parquet-column</artifactId>
      <version>1.8.1</version>
  </dependency>
  <dependency>
      <groupId>org.apache.parquet</groupId>
      <artifactId>parquet-hadoop</artifactId>
      <version>1.8.1</version>
  </dependency>
```

arquet读写有新旧两个版本，主要是新旧MR api之分，我们用新旧老版本的MR实现下parquet文件的读写

### 旧版本如下

```

package com.fan.hadoop.parquet;
 
import java.io.IOException;
import java.util.*;
import org.apache.hadoop.fs.Path;
import org.apache.hadoop.io.*;
import org.apache.hadoop.mapred.*;
import org.apache.parquet.hadoop.example.GroupWriteSupport;
import org.apache.parquet.example.data.Group;
import org.apache.parquet.example.data.simple.SimpleGroupFactory;
import org.apache.parquet.hadoop.mapred.DeprecatedParquetOutputFormat;
import org.apache.parquet.schema.MessageTypeParser;
/**
 * Created by fanlegefan.com on 17-7-17.
 */
public class ParquetMR {
 
    public static class Map extends MapReduceBase implements
            Mapper<LongWritable, Text, Text, IntWritable> {
 
        private final static IntWritable one = new IntWritable(1);
        private Text word = new Text();
 
        public void map(LongWritable key, Text value,
                        OutputCollector<Text, IntWritable> output,
                            Reporter reporter) throws IOException {
            String line = value.toString();
            StringTokenizer tokenizer = new StringTokenizer(line);
            while (tokenizer.hasMoreTokens()) {
                word.set(tokenizer.nextToken());
                output.collect(word, one);
            }
        }
    }
 
    public static class Reduce extends MapReduceBase implements
            Reducer<Text, IntWritable, Void, Group> {
        private SimpleGroupFactory factory;
        public void reduce(Text key, Iterator<IntWritable> values,
                           OutputCollector<Void, Group> output,
                           Reporter reporter) throws IOException {
            int sum = 0;
            while (values.hasNext()) {
                sum += values.next().get();
            }
 
            Group group = factory.newGroup()
                    .append("name",  key.toString())
                    .append("age", sum);
            output.collect(null,group);
        }
 
        @Override
        public void configure(JobConf job) {
            factory = new SimpleGroupFactory(GroupWriteSupport.getSchema(job));
        }
    }
 
    public static void main(String[] args) throws Exception {
        JobConf conf = new JobConf(ParquetMR.class);
        conf.setJobName("wordcount");
 
        String in = "hdfs://localhost:9000/test/wordcount.txt";
        String out = "hdfs://localhost:9000/test/wd";
 
 
        String writeSchema = "message example {\n" +
                "required binary name;\n" +
                "required int32 age;\n" +
                "}";
 
        conf.setMapOutputKeyClass(Text.class);
        conf.setMapOutputValueClass(IntWritable.class);
 
        conf.setOutputKeyClass(NullWritable.class);
        conf.setOutputValueClass(Group.class);
 
        conf.setMapperClass(Map.class);
        conf.setReducerClass(Reduce.class);
 
        conf.setInputFormat(TextInputFormat.class);
        conf.setOutputFormat(DeprecatedParquetOutputFormat.class);
 
        FileInputFormat.setInputPaths(conf, new Path(in));
        DeprecatedParquetOutputFormat.setWriteSupportClass(conf, GroupWriteSupport.class);
        GroupWriteSupport.setSchema(MessageTypeParser.parseMessageType(writeSchema), conf);
 
        DeprecatedParquetOutputFormat.setOutputPath(conf, new Path(out));
 
        JobClient.runJob(conf);
    }
 
}

```

生成的文件：

```
hadoop dfs -ls /test/wd
Found 2 items
-rw-r--r--   3 work supergroup          0 2017-07-18 17:41 /test/wd/_SUCCESS
-rw-r--r--   3 work supergroup        392 2017-07-18 17:41 /test/wd/part-00000-r-00000.parquet
```
将生成的文件复制到hive表test_parquet的路径下：

```
hadoop dfs -cp /test/wd/part-00000-r-00000.parquet /user/work/warehouse/test_parquet/
```
测试hive表读取parquet文件
```
hive> select * from test_parquet limit 10;
OK
action  2
hadoop  2
hello   3
in  2
presto  1
spark   1
world   1
Time taken: 0.056 seconds, Fetched: 7 row(s)
```

### 新版本如下

新版本的MR读写Parquet和老版本有点区别，schema必须用在conf中设置，其他的区别不大

>conf.set("parquet.example.schema",writeSchema);

```
package com.fan.hadoop.parquet;
 
import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.fs.Path;
import org.apache.hadoop.io.IntWritable;
import org.apache.hadoop.io.LongWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Job;
import org.apache.hadoop.mapreduce.Mapper;
import org.apache.hadoop.mapreduce.Reducer;
import org.apache.hadoop.mapreduce.lib.input.FileInputFormat;
import org.apache.hadoop.mapreduce.lib.input.TextInputFormat;
import org.apache.parquet.example.data.Group;
import org.apache.parquet.example.data.simple.SimpleGroupFactory;
import org.apache.parquet.hadoop.ParquetOutputFormat;
import org.apache.parquet.hadoop.example.GroupWriteSupport;
import java.io.IOException;
import java.util.StringTokenizer;
 
/**
 * Created by fanglegefan.com on 17-7-18.
 */
public class ParquetNewMR {
 
    public static class WordCountMap extends
            Mapper<LongWritable, Text, Text, IntWritable> {
 
        private final IntWritable one = new IntWritable(1);
        private Text word = new Text();
 
        public void map(LongWritable key, Text value, Context context)
                throws IOException, InterruptedException {
            String line = value.toString();
            StringTokenizer token = new StringTokenizer(line);
            while (token.hasMoreTokens()) {
                word.set(token.nextToken());
                context.write(word, one);
            }
        }
    }
 
    public static class WordCountReduce extends
            Reducer<Text, IntWritable, Void, Group> {
        private SimpleGroupFactory factory;
 
        public void reduce(Text key, Iterable<IntWritable> values,
                           Context context) throws IOException, InterruptedException {
            int sum = 0;
            for (IntWritable val : values) {
                sum += val.get();
            }
            Group group = factory.newGroup()
                    .append("name",  key.toString())
                    .append("age", sum);
            context.write(null,group);
        }
 
        @Override
        protected void setup(Context context) throws IOException, InterruptedException {
            super.setup(context);
            factory = new SimpleGroupFactory(GroupWriteSupport.getSchema(context.getConfiguration()));
 
        }
    }
 
    public static void main(String[] args) throws Exception {
        Configuration conf = new Configuration();
        String writeSchema = "message example {\n" +
                "required binary name;\n" +
                "required int32 age;\n" +
                "}";
        conf.set("parquet.example.schema",writeSchema);
 
        Job job = new Job(conf);
        job.setJarByClass(ParquetNewMR.class);
        job.setJobName("parquet");
 
        String in = "hdfs://localhost:9000/test/wordcount.txt";
        String out = "hdfs://localhost:9000/test/wd1";
 
        job.setMapOutputKeyClass(Text.class);
        job.setMapOutputValueClass(IntWritable.class);
 
        job.setOutputValueClass(Group.class);
 
        job.setMapperClass(WordCountMap.class);
        job.setReducerClass(WordCountReduce.class);
 
        job.setInputFormatClass(TextInputFormat.class);
        job.setOutputFormatClass(ParquetOutputFormat.class);
 
        FileInputFormat.addInputPath(job, new Path(in));
        ParquetOutputFormat.setOutputPath(job, new Path(out));
        ParquetOutputFormat.setWriteSupportClass(job, GroupWriteSupport.class);
 
        job.waitForCompletion(true);
    }
}
```
查看生成的文件

```
hadoop dfs -ls /user/work/warehouse/test_parquet
 
Found 4 items
-rw-r--r--   1 work work          0 2017-07-18 18:27 /user/work/warehouse/test_parquet/_SUCCESS
-rw-r--r--   1 work work        129 2017-07-18 18:27 /user/work/warehouse/test_parquet/_common_metadata
-rw-r--r--   1 work work        275 2017-07-18 18:27 /user/work/warehouse/test_parquet/_metadata
-rw-r--r--   1 work work        392 2017-07-18 18:27 /user/work/warehouse/test_parquet/part-r-00000.parquet
```
将生成的文件复制到hive表test_parquet的路径下：
>hadoop dfs -cp /test/wd/part-00000-r-00000.parquet /user/work/warehouse/test_parquet/


测试hive

```

hive> select name,age from test_parquet limit 10;
OK
action  2
hadoop  2
hello   3
in  2
presto  1
spark   1
world   1
Time taken: 0.036 seconds, Fetched: 7 row(s)
```

### 用mapreduce读parquet文件

```
package com.fan.hadoop.parquet;
 
import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.fs.Path;
import org.apache.hadoop.io.LongWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Job;
import org.apache.hadoop.mapreduce.Mapper;
import org.apache.hadoop.mapreduce.Reducer;
import org.apache.hadoop.mapreduce.lib.output.FileOutputFormat;
import org.apache.hadoop.mapreduce.lib.output.TextOutputFormat;
import org.apache.parquet.example.data.Group;
import org.apache.parquet.hadoop.ParquetInputFormat;
import org.apache.parquet.hadoop.api.DelegatingReadSupport;
import org.apache.parquet.hadoop.api.InitContext;
import org.apache.parquet.hadoop.api.ReadSupport;
import org.apache.parquet.hadoop.example.GroupReadSupport;
 
import java.io.IOException;
import java.util.*;
 
/**
 * Created by fanglegefan.com on 17-7-18.
 */
public class ParquetNewMRReader {
 
    public static class WordCountMap1 extends
            Mapper<Void, Group, LongWritable, Text> {
 
        protected void map(Void key, Group value,
                           Mapper<Void, Group, LongWritable, Text>.Context context)
                throws IOException, InterruptedException {
 
            String name = value.getString("name",0);
            int  age = value.getInteger("age",0);
 
            context.write(new LongWritable(age),
                    new Text(name));
        }
    }
 
    public static class WordCountReduce1 extends
            Reducer<LongWritable, Text, LongWritable, Text> {
 
        public void reduce(LongWritable key, Iterable<Text> values,
                           Context context) throws IOException, InterruptedException {
            Iterator<Text> iterator = values.iterator();
            while(iterator.hasNext()){
                context.write(key,iterator.next());
            }
        }
 
    }
 
    public static final class MyReadSupport extends DelegatingReadSupport<Group> {
        public MyReadSupport() {
            super(new GroupReadSupport());
        }
 
        @Override
        public org.apache.parquet.hadoop.api.ReadSupport.ReadContext init(InitContext context) {
            return super.init(context);
        }
    }
 
    public static void main(String[] args) throws Exception {
        Configuration conf = new Configuration();
        String readSchema = "message example {\n" +
                "required binary name;\n" +
                "required int32 age;\n" +
                "}";
        conf.set(ReadSupport.PARQUET_READ_SCHEMA, readSchema);
 
        Job job = new Job(conf);
        job.setJarByClass(ParquetNewMRReader.class);
        job.setJobName("parquet");
 
        String in = "hdfs://localhost:9000/test/wd1";
        String  out = "hdfs://localhost:9000/test/wd2";
 
 
        job.setMapperClass(WordCountMap1.class);
        job.setReducerClass(WordCountReduce1.class);
 
        job.setInputFormatClass(ParquetInputFormat.class);
        ParquetInputFormat.setReadSupportClass(job, MyReadSupport.class);
        ParquetInputFormat.addInputPath(job, new Path(in));
 
        job.setOutputFormatClass(TextOutputFormat.class);
        FileOutputFormat.setOutputPath(job, new Path(out));
 
        job.waitForCompletion(true);
    }
}
```

查看生成的文件

```
hadoop dfs -cat /test/wd2/part-r-00000
 
1       world
1       spark
1       presto
2       in
2       hadoop
2       action

```


> https://blog.csdn.net/woloqun/article/details/76068147







