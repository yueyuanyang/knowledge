## Mapreduce原理全剖析过程

### 1.mapreduce原理全剖析map+shuffle+reducer 

![p1](https://github.com/yueyuanyang/knowledge/blob/master/hadoop/theory/img/p1.png)

**wordcount为例**

- 1.mapper开始运行，调用InputFormat组件读取文件逻辑切片（逻辑切片不是block块，切片大小默认和block块大小相同）
- 2.经过inputformat组件处理后，文件以<k,v>的形式进入我们自定义的mapper逻辑
- 3.mapper逻辑中输出结果会调用OutPutCollector组件写入环形缓冲区。
- 4.环形缓冲区的存储达到默认阀值会调用Spliller组件将内容分区且排序（快排算法，外部排序算法）后溢写到磁盘文件中，mapper组后结果不满环形缓冲区也会溢写到磁盘。
- 5.mapper结束后磁盘中的结果小文件会合并（merge），产生大文件（分区且排序，归并算法）。
- 6.reducer启动后会到不同的map结果文件中下载相同区号的结果文件，再合并这些来自不同map的结果文件，再将这些文件合并（归并算法），产生的大文件是分区且排序且分好组了的，分组调用默认的GroupingComparator组件。
- 7.reducer把下载的所有map输出文件合并完成之后就会开始读取文件，将读入的内容以<k,v>的形式输入到我们用户自定义的reducer处理逻辑中。
- 8.用户逻辑完成之后以<k,v>的形式调用OutPutFormat组件输出到hdfs文件系统中去保存。

### 2.wordcount详细shuffle机制

#### 一、map方法执行之前
![](https://github.com/yueyuanyang/knowledge/blob/master/hadoop/theory/img/p2.jpg)

 我们知道，HDFS里的文件是分块存放在Datanode上面的，而我们写的mapper程序也是跑在各个节点上的。这里就涉及到一个问题，哪一个节点上的mapper读哪一些节点上的文件块呢？hadoop会自动将这个文件分片（split），得到好多split，这每一个split放到一个节点的一个mapper里面去读。然后在每一台有mapper任务的节点上都执行了这么一个操作，将分得到的split切割成一行一行的键值对，然后传给map方法。键是这每一行在split中的偏移量，值是每一行得到的字符串。

#### 二、执行map方法

![](https://github.com/yueyuanyang/knowledge/blob/master/hadoop/theory/img/p3.png)

写过wordcount的朋友都知道，这个过程就是读到每一行，切割字符串，生成键值对写出去。

#### 三、shuffle操作（一）

这个过程是在有map任务的节点上完成的

![](https://github.com/yueyuanyang/knowledge/blob/master/hadoop/theory/img/p4.jpg)

1. partition

将得到的键值对按照一定的规则分组，例如例子中将首字母为a的全部分到一组，将首字母为b的分到一组。这里只是为了讲明白这个方式，进行了过程简化，实际不一定是分为两组，也不一定是按照首字母分组。

2. sort

对每一个组中的键值对根据键的哈希码排序。

3. combine

将具有相同键的键值对合成一个新的键值对，这个新的键值对的键是原来的键，键值是所有键的键值之和。

### 四、shuffle操作（二）

这个过程是在有reduce任务的节点上完成的。

![](https://github.com/yueyuanyang/knowledge/blob/master/hadoop/theory/img/p5.jpg)

1. 拉取partition

hadoop决定有多少个reducer的时候会规定有多少个partition，每一个reducer拉取自己要处理的那个分组的全部成员。例如，某台节点要处理所有以a开头的键值对，它就会将所有mapper中的以a开头的那一组全部拉取过来。

2. merge

在每一个reducer上，将具有相同键的键值对生成另外一个新的键值对，键是以前的键，键值是一个以前键值的集合。

3. sort

在每一台reducer节点上，将新生成的键值对进行排序，根据 哈希码值。

五、reduce操作

![](https://github.com/yueyuanyang/knowledge/blob/master/hadoop/theory/img/p6.jpg)

 写过wordcount的朋友都知道，在reduce方法中，hadoop回传过来一个一个的键值对，键是每一个单词，键值就是四中新生成的键值对的键值。执行reduce操作，就是将每一个键值对中的键值累加起来。然后以键值对的形式将结果写出去。

六、文件写入HDFS

![](https://github.com/yueyuanyang/knowledge/blob/master/hadoop/theory/img/p7.jpg)

在每一台reducer节点上将文件写入，实际上是写成一个一个的文件块，但对外的表现形式是一整个大的结果文件。







