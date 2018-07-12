## Mapreduce原理全剖析过程

### 1.mapreduce原理全剖析map+shuffle+reducer 

![p1]()

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

一、map方法执行之前




