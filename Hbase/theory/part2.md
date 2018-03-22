## HBase的RowKey设计原则

HBase是三维有序存储的，通过rowkey（行键），column key（column family和qualifier）和TimeStamp（时间戳）这个三个维度可以对HBase中的数据进行快速定位。

HBase中rowkey可以唯一标识一行记录，在HBase查询的时候，有以下几种方式：

- 通过get方式，指定rowkey获取唯一一条记录

- 通过scan方式，设置startRow和stopRow参数进行范围匹配

- 全表扫描，即直接扫描整张表中所有行记录

## RowKey设计的3条原则

### 原则一：rowkey长度原则
rowkey是一个二进制码流，可以是任意字符串，最大长度 64kb ，实际应用中一般为10-100bytes，以 byte[] 形式保存，一般设计成定长。

建议越短越好，不要超过16个字节，原因如下：

1. 数据的持久化文件HFile中是按照KeyValue存储的，如果rowkey过长，比如超过100字节，1000w行数据，光rowkey就要占用100 * 1000w=10亿个字节，将近1G数据，这样会极大影响HFile的存储效率；

2. MemStore将缓存部分数据到内存，如果rowkey字段过长，内存的有效利用率就会降低，系统不能缓存更多的数据，这样会降低检索效率。

3. 目前操作系统都是64位系统，内存8字节对齐，控制在16个字节，8字节的整数倍利用了操作系统的最佳特性。

### 原则二 ：rowkey散列原则

如果rowkey按照时间戳的方式递增，不要将时间放在二进制码的前面，建议将rowkey的高位作为散列字段，由程序随机生成，低位放时间字段，这样将提高数据均衡分布在每个RegionServer，以实现负载均衡的几率。如果没有散列字段，首字段直接是时间信息，所有的数据都会集中在一个RegionServer上，这样在数据检索的时候负载会集中在个别的RegionServer上，造成热点问题，会降低查询效率。

```
在HBase当中，表会被划分为1...n个Region，被托管在RegionServer中。Region二个重要的属性:StartKey与EndKey表示这个 
Region维护的rowKey范围，当我们要读/写数据时，如果rowKey落在某个start-end key范围内，
那么就会定位到目标region并且读/写到相关的数据。

如果我们在建表的时候，不预先设定好分区，随着表内数据的增多，HBase会自动把表进行split，但是这样做有几点问题，
第一，分区的原则，可能并不是我们想要的；
第二，在表内数据相对较少的时候，无法充分展现分布式并发处理的优势；
第三，split操作，其实是比较耗资源的，如果数据增长过快，可能会相对频繁的发生。

那么，预分区又是如何实现的，我们看下面这个语句：

create 'testtable', 'common', 'data', {SPLITS => ['1','2','3']}

建好之后，我们可以在HBase的web页面当中，看到表的分区的分布情况：
```
| Name | Region Server |  start key | end key | locality | requests 
|- | :-: | -: | :-: | :-: | :-: |
|testtable:443444 | master:1602|  | 1 | 0.0| 21 |
|testtable:234225 | slave01:1602| 1 | 2 | 0.0| 22 | 
|testtable:234234 | slave02:1602| 2 | 3 | 0.0| 31 |
|testtable:453343 | slave03:1602| 3 |  | 0.0| 23 |

```
我们看到，我们用1,2,3三个数，把一个table划分为了4个region，这四个region分别是：

x<1,  1<=x<2,  2<=x<3,  3<=x

那么我们在insert的数据的时候，又该怎么做呢？看下面的scala代码：

val p = new Put(Bytes.toBytes(String.valueOf(random.nextInt(4)) + new Date().getTime))

也可以取模

val p = new Put(Bytes.toBytes((ID mod 5) + id))

这句话是为一条新的数据，生成一个RowKey，那么我们的key值由2部分组成，随机散列+时间戳，
其中随机散列包括：【0,1,2,3】一共4个值，根据上面我们划分的region，这4个值，将会映射到4个不同的region当中。

好吧，刚才我们直接把一种比较好的解决方案告诉给大家了。
```

### 原则三：rowkey唯一原则

必须在设计上保证其唯一性，rowkey是按照字典顺序排序存储的，因此，设计rowkey的时候，要充分利用这个排序的特点，将经常读取的数据存储到一块，将最近可能会被访问的数据放到一块。

### 加盐

这里所说的加盐不是密码学中的加盐，而是在rowkey的前面增加随机数，具体就是给rowkey分配一个随机前缀以使得它和之前的rowkey的开头不同。分配的前缀种类数量应该和你想使用数据分散到不同的region的数量一致。加盐之后的rowkey就会根据随机生成的前缀分散到各个region上，以避免热点。

```
效果应该是让每个节点提供的请求处理都是均等的，同时数据能够相对均匀的分布到各个region中。
为此我们最后采取的方法是数据加盐（salting）存储与hbase协处理器查询数据。

先介绍一下Hbase加盐存储，思路比较简单，每个region预分区都会指定一个startkey与endkey，
然后插入数据的时候对rowkey进行hash取余，产生的code为盐值，添加到rowkey前面作为rowkey的组成部分。
比如，我们预分区指定1000个region，每个region的startkey与endkey为000～999依次增加，
region1：000-001,region2:001-002,....region1000:999-。然后插入数据rowkey="20150524002300_1232"，
       
intsplitsCount= 1000;
StringrowKey= "20150524002300_1232";
int saltingCode = rowKey.hashCode()%splitsCount;
StringsaltingKey= ""+ saltingCode;
if(saltingCode < 10)
    {
        saltingKey = "00" + saltingKey;
    }else if(saltingCode < 100){
        saltingKey = "0" + saltingKey;
    }
    rowKey = saltingKey + rowKey;
    
这样就会使插入的数据相对均匀的分布到1000个region中去，然后MR程序进行处理时，每个region默认一个map处理，
相对处理速度会有很大的提升，我们之前跑几个小时的map任务采用该方法后，只需要20分钟左右，效果还是非常明显的。

上面讲了存储，现在在讲一下怎么查询数据，由于插入的数据被我们默认都添加了盐值，导致本来在hbase
连续存储的数据被分散到了多个region中，所以无论是根据rowkey查询单条记录，还是由startkey与endkey
进行查询，都不能再简单的调用hbase接口进行查询，解决的方法是采用hbase协处理器进行查询，
hbase协处理器包括两种，一种是观察者(Observer)，另外一种是终端(Endpoint)，我们这里需要使用的是后
一种endpoint，基本思路是endpoint类似于关系型数据库中的存储过程，作用于每个region，每个region分别
加盐查询，讲解过返回到客户端，客户端进行合并，就是最后的查询结果，
比如我们查询"201501010000"与"20150524000000"之间的数据，region1查询"000201501010000"
与"00020150524000000"，region2查询"001201501010000"与"00120150524000000"... 最后1000个region均返回结果，
进行合并就是我们要查询的结果。相应的具体实现后面文章给出。    
```

### 哈希

哈希会使同一行永远用一个前缀加盐。哈希也可以使负载分散到整个集群，但是读却是可以预测的。使用确定的哈希可以让客户端重构完整的rowkey，可以使用get操作准确获取某一个行数据

### 反转

第三种防止热点的方法时反转固定长度或者数字格式的rowkey。这样可以使得rowkey中经常改变的部分（最没有意义的部分）放在前面。这样可以有效的随机rowkey，但是牺牲了rowkey的有序性。

反转rowkey的例子以手机号为rowkey，可以将手机号反转后的字符串作为rowkey，这样的就避免了以手机号那样比较固定开头导致热点问题

### 时间戳反转

一个常见的数据处理问题是快速获取数据的最近版本，使用反转的时间戳作为rowkey的一部分对这个问题十分有用，可以用 Long.Max_Value - timestamp 追加到key的末尾，例如 [key][reverse_timestamp] , [key] 的最新值可以通过scan [key]获得[key]的第一条记录，因为HBase中rowkey是有序的，第一条记录是最后录入的数据。

比如需要保存一个用户的操作记录，按照操作时间倒序排序，在设计rowkey的时候，可以这样设计

[userId反转][Long.Max_Value - timestamp]，在查询用户的所有操作记录数据的时候，直接指定反转后的userId，startRow是[userId反转][000000000000],stopRow是[userId反转][Long.Max_Value - timestamp]

如果需要查询某段时间的操作记录，startRow是[user反转][Long.Max_Value - 起始时间]，stopRow是[userId反转][Long.Max_Value - 结束时间]

其他一些建议

- 尽量减少行和列的大小在HBase中，value永远和它的key一起传输的。当具体的值在系统间传输时，它的rowkey，列名，时间戳也会一起传输。如果你的rowkey和列名很大，甚至可以和具体的值相比较，那么你将会遇到一些有趣的问题。HBase storefiles中的索引（有助于随机访问）最终占据了HBase分配的大量内存，因为具体的值和它的key很大。可以增加block大小使得storefiles索引再更大的时间间隔增加，或者修改表的模式以减小rowkey和列名的大小。压缩也有助于更大的索引。

- 列族尽可能越短越好，最好是一个字符

- 冗长的属性名虽然可读性好，但是更短的属性名存储在HBase中会更好


