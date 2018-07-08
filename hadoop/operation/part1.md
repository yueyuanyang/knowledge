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









