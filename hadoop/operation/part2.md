## mapreduce对温度数据进行自定义排序、分组、分区等

### 一、需求说明
---

#### 1、数据文件说明

hdfs中有一些存储温度的数据文件，以文本形式存储，示例如下：

日期和时间中间是空格，为整体，表示检测站点监测的时间，后面是检测的温度，中间通过制表符 t 相隔。

```
1949-10-01 14:21:02 34
1949-10-01 14:21:02 36
1949-10-01 14:21:02 32
1950-10-01 14:21:02 37
1951-10-01 14:21:02 41
1951-10-01 14:21:02 46
```
#### 2、需求

1. 计算在1949-1955年中,每年的温度降序排序且每年单独一个文件输出存储

需要进行自定义分区、自定义分组、自定义排序。

### 二、解决
---

**1、思路**

- 1.按照年份升序排序再按照每年的温度降序排序
- 2.按照年份进行分组,每一年份对应一个reduce task

**2、自定义mapper输出类型KeyPair**

可以看出，每一行温度姑且称为一个数据，每个数据中有两部分，一部分是时间，另一部分是温度。

因此map输出必须使用自定义的格式输出，并且输出之后需要自定义进行排序和分组等操作，默认的那些都不管用了。

**定义KeyPair**

自定义的输出类型因为要将map的输出放到reduce中去运行，因此需要实现hadoop的WritableComparable的接口，并且该接口的模板变量也得是KeyPair，就像是LongWritable一个意思（查看LongWritable的定义就可以知道）

**实现WritableComparable 的接口，就必须重写write/readFileds/compareTo三个方法，依次作用于序列化/反序列化/比较**

同时需要重写toString和hashCode避免equals的问题。

**KeyPair定义如下**

值得注意的是：**在进行序列化输出的时候也就是write，里面用了将标准格式的时间（文件中显示的格式时间）进行的时间的转换，用了DataInput和DataOutput**

```
import org.apache.hadoop.io.WritableComparable;
import java.io.DataInput;
import java.io.DataOutput;
import java.io.IOException;
 
/**
 * Project   : hadooptest2
 * Package   : com.mapreducetest.temp
 * User      : Postbird @ http://www.ptbird.cn
 * TIME      : 2017-01-19 21:53
 */
 
/**
 * 为温度和年份封装成对象
 * year表示年份 而temp为温度
 */
public class KeyPair implements WritableComparable<KeyPair>{
    //年份
    private int year;
    //温度
    private int temp;
 
    public void setYear(int year) {
        this.year = year;
    }
 
    public void setTemp(int temp) {
        this.temp = temp;
    }
 
    public int getYear() {
        return year;
    }
 
    public int getTemp() {
        return temp;
    }
    @Override
    public int compareTo(KeyPair o) {
        //传过来的对象和当前的year比较 相等为0 不相等为1
        int result=Integer.compare(year,o.getYear());
        if(result != 0){
            //两个year不相等
            return 0;
        }
        //如果年份相等 比较温度
        return Integer.compare(temp,o.getTemp());
    }
 
    @Override
    //序列化
    public void write(DataOutput dataOutput) throws IOException {
       dataOutput.writeInt(year);
       dataOutput.writeInt(temp);
    }
 
    @Override
    //反序列化
    public void readFields(DataInput dataInput) throws IOException {
        this.year=dataInput.readInt();
        this.temp=dataInput.readInt();
    }
 
    @Override
    public String toString() {
        return year+"\t"+temp;
    }
 
    @Override
    public int hashCode() {
        return new Integer(year+temp).hashCode();
    }
}
```

### 3、自定义分组
---

将同一年监测的温度放到一起，因此需要对年份进行比较。

因此比较输入的数据中的年份即可，注意此时比较的都是KeyPair的类型，Map出来的输出也是这个类型。

**因为继承了WritableComparator，因此需要重写compare方法，比较的是KeyPair（KeyPair实现了WritableComparable接口），实际比较的使他们的年份,年份相同则得到0**

```
/**
 * Project   : hadooptest2
 * Package   : com.mapreducetest.temp
 * User      : Postbird @ http://www.ptbird.cn
 * TIME      : 2017-01-19 22:08
 */
 
import org.apache.hadoop.io.WritableComparable;
import org.apache.hadoop.io.WritableComparator;
 
/**
 *  为温度分组 比较年份即可
 */
public class GroupTemp extends WritableComparator{
 
    public GroupTemp() {
        super(KeyPair.class,true);
    }
    @Override
    public int compare(WritableComparable a, WritableComparable b) {
        //年份相同返回的是0
        KeyPair o1=(KeyPair)a;
        KeyPair o2=(KeyPair)b;
        return Integer.compare(o1.getYear(),o2.getYear());
    }
}
```

### 4、自定义分区
---

自定义分区的目的是在根据年份分好了组之后，将不同的年份创建不同的reduce task任务，因此需要对年份处理。

```
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Partitioner;
 
/**
 * Project   : hadooptest2
 * Package   : com.mapreducetest.temp
 * User      : Postbird @ http://www.ptbird.cn
 * TIME      : 2017-01-19 22:17
 */
 
//自定义分区
//每一个年份生成一个reduce任务
public class FirstPartition extends Partitioner<KeyPair,Text>{
    @Override
    public int getPartition(KeyPair key, Text value, int num) {
        //按照年份进行分区 年份相同,返回的是同一个值
        return (key.getYear()*127)%num;
    }
}

```

### 5、自定义排序
---

最终还是比较的是温度的排序，因此这部分也是非常重要的。

根据上面的需求，需要对年份进行生序排序，而对温度进行降序排序，首选比较条件是年份.
```
/**
 * Project   : hadooptest2
 * Package   : com.mapreducetest.temp
 * User      : Postbird @ http://www.ptbird.cn
 * TIME      : 2017-01-19 22:08
 */
 
import org.apache.hadoop.io.WritableComparable;
import org.apache.hadoop.io.WritableComparator;
 
/**
 *  为温度排序的封装类
 */
public class SortTemp extends WritableComparator{
 
    public SortTemp() {
        super(KeyPair.class,true);
    }
    //自定义排序
    @Override
    public int compare(WritableComparable a, WritableComparable b) {
        //按照年份升序排序 按照温度降序排序
        KeyPair o1=(KeyPair)a;
        KeyPair o2=(KeyPair)b;
        int result=Integer.compare(o1.getYear(),o2.getYear());
        //比较年份 如果年份不相等
        if(result != 0){
            return result;
        }
        //两个年份相等 对温度进行降序排序，注意 - 号
        return -Integer.compare(o1.getTemp(),o2.getTemp());
    }
}

```

### 6、MapReduce程序的编写

几个值得注意的点：

- 1.数据文件中前面的时间是字符串，但是我们的KeyPair的set却不是字符串，因此需要进行字符串转日期的format操作，使用的是SimpleDateFormat，格式自然是"yyyy-MM-dd HH:mm:ss"了。
- 2.输入每行数据之后，通过正则匹配"t"的制表符，然后将温度和时间分开，将时间format并得到年份，将第二部分字符串去掉“℃”的符号得到数字，然后创建KeyPair类型的数据，在输出即可。

- 1.每个年份都生成一个reduce task依据就是自定义分区中对年份进行了比较处理，为了简单就把map的输出结果在reduce中再输出一次，三个reduce task，就会生成三个输出文件。
- 2.因为使用了自定义的排序，分组，分区，因此就需要进行指定相关的class，同时也需要执行reduce task的数量。
- 3.其实最后客户端还是八股文的固定形式而已，只不过多了自定义的指定，没有别的。

```

import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.fs.FileSystem;
import org.apache.hadoop.fs.Path;
import org.apache.hadoop.io.LongWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Job;
import org.apache.hadoop.mapreduce.Mapper;
import org.apache.hadoop.mapreduce.Reducer;
import org.apache.hadoop.mapreduce.lib.input.FileInputFormat;
import org.apache.hadoop.mapreduce.lib.output.FileOutputFormat;
import org.apache.hadoop.util.GenericOptionsParser;
 
import java.io.IOException;
import java.net.URI;
import java.text.SimpleDateFormat;
import java.util.Calendar;
import java.util.Date;
 
/**
 * Project   : hadooptest2
 * Package   : com.mapreducetest.temp
 * User      : Postbird @ http://www.ptbird.cn
 * TIME      : 2017-01-19 22:28
 */
public class RunTempJob {
    //字符串转日期format
    public static SimpleDateFormat SDF=new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
    /**
     * Mapper
     * 输出的Key是自定义的KeyPair
     */
    static class TempMapper extends Mapper<LongWritable,Text,KeyPair,Text>{
        protected void map(LongWritable key,Text value,Context context) 
            throws IOException,InterruptedException{
            String line=value.toString();
            //1949-10-01 14:21:02    34℃
            // 前面是空格 时间和温度通过\t分割
            String[] ss=line.split("\t");
//            System.err.println(ss.length);
            if(ss.length==2){
                try{
                    //获得日期
                    Date date=SDF.parse(ss[0]);
                    Calendar c=Calendar.getInstance();
                    c.setTime(date);
                    int year=c.get(1);//得到年份
                    //字符串截取得到温度，去掉℃
                    String temp = ss[1].substring(0,ss[1].indexOf("℃"));
                    //创建输出key 类型为KeyPair
                    KeyPair kp=new KeyPair();
                    kp.setYear(year);
                    kp.setTemp(Integer.parseInt(temp));
                    //输出
                    context.write(kp,value);
                }catch(Exception ex){
                    ex.printStackTrace();
                }
            }
        }
    }
    /**
     *  Reduce 区域
     *  Map的输出是Reduce的输出
     */
    static class TempReducer extends Reducer<KeyPair,Text,KeyPair,Text> {
        @Override
        protected void reduce(KeyPair kp, Iterable<Text> values, Context context) 
             throws IOException, InterruptedException {
            for (Text value:values){
                context.write(kp,value);
            }
        }
    }
 
    //client
    public static void main(String args[]) throws IOException, InterruptedException{
        //获取配置
        Configuration conf=new Configuration();
 
        //修改命令行的配置
        String[] otherArgs = new GenericOptionsParser(conf, args).getRemainingArgs();
        if (otherArgs.length != 2) {
            System.err.println("Usage: temp <in> <out>");
            System.exit(2);
        }
        //创建Job
        Job job=new Job(conf,"temp");
        //1.设置job运行的类
        job.setJarByClass(RunTempJob.class);
        //2.设置map和reduce的类
        job.setMapperClass(RunTempJob.TempMapper.class);
        job.setReducerClass(RunTempJob.TempReducer.class);
        //3.设置map的输出的key和value 的类型
        job.setMapOutputKeyClass(KeyPair.class);
        job.setMapOutputValueClass(Text.class);
        //4.设置输入文件的目录和输出文件的目录
        FileInputFormat.addInputPath(job,new Path(otherArgs[0]));
        FileOutputFormat.setOutputPath(job,new Path(otherArgs[1]));
        //5.设置Reduce task的数量 每个年份对应一个reduce task
        job.setNumReduceTasks(3);//3个年份
        //5.设置partition sort Group的class
        job.setPartitionerClass(FirstPartition.class);
        job.setSortComparatorClass(SortTemp.class);
        job.setGroupingComparatorClass(GroupTemp.class);
        //6.提交job 等待运行结束并在客户端显示运行信息
        boolean isSuccess= false;
        try {
            isSuccess = job.waitForCompletion(true);
        } catch (ClassNotFoundException e) {
            e.printStackTrace();
        }
        //7.结束程序
        System.exit(isSuccess ?0:1);
    }
}
```

**三、生成效果：**

HDFS中三个reduce task会生成三个输出。


