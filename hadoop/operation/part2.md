## mapreduce对温度数据进行自定义排序、分组、分区等

### 一、需求说明

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






