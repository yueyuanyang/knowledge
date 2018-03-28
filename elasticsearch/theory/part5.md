## Elasticsearch 索引原理

### 索引

Elasticsearch索引的精髓：

> 一切设计都是为了提高搜索的性能

另一层意思：为了提高搜索的性能，难免会牺牲某些其他方面，比如插入/更新，否则其他数据库不用混了

前面看到往Elasticsearch里插入一条记录，其实就是直接PUT一个json的对象，这个对象有多个fields，比如上面例子中的name, sex, age, about, interests，那么在插入这些数据到Elasticsearch的同时，Elasticsearch还默默1的为这些字段建立索引–倒排索引，因为Elasticsearch最核心功能是搜索。

### Elasticsearch 是如何做到快速索引的

Elasticsearch使用的倒排索引比关系型数据库的B-Tree索引快

**什么是B-Tree索引?**

二叉树查找效率是logN，同时插入新的节点不必移动全部节点，所以用树型结构存储索引，能同时兼顾插入和查询的性能。

因此在这个基础上，再结合磁盘的读取特性(顺序读/随机读)，传统关系型数据库采用了B-Tree/B+Tree这样的数据结构：

![B-Tree索引](https://github.com/yueyuanyang/knowledge/blob/master/elasticsearch/img/b-tree.png)

为了提高查询的效率，减少磁盘寻道次数，将多个值作为一个数组通过连续区间存放，一次寻道读取多个数据，同时也降低树的高度。

**什么是倒排索引?**

先上中的图解

![索引总图解](https://github.com/yueyuanyang/knowledge/blob/master/elasticsearch/img/index.png)

**部分详解：**

**Posting List**
Elasticsearch分别为每个field都建立了一个倒排索引，Kate, John, 24, Female这些叫term，而[1,2]就是Posting List。Posting list就是一个int的数组，存储了所有符合某个term的文档id。

通过posting list这种索引方式似乎可以很快进行查找，比如要找age=24的同学，爱回答问题的小明马上就举手回答：我知道，id是1，2的同学。但是，如果这里有上千万的记录呢？如果是想通过name来查找呢？

**Term Dictionary**
Elasticsearch为了能快速找到某个term，将所有的term排个序，二分法查找term，logN的查找效率，就像通过字典查找一样，这就是Term Dictionary。现在再看起来，似乎和传统数据库通过B-Tree的方式类似啊，为什么说比B-Tree的查询快呢？

**Term Index**
B-Tree通过减少磁盘寻道次数来提高查询性能，Elasticsearch也是采用同样的思路，直接通过内存查找term，不读磁盘，但是如果term太多，term dictionary也会很大，放内存不现实，于是有了Term Index，就像字典里的索引页一样，A开头的有哪些term，分别在哪页，可以理解term index是一颗树：

![Index](https://github.com/yueyuanyang/knowledge/blob/master/elasticsearch/img/term-index.png)

这棵树不会包含所有的term，它包含的是term的一些前缀。通过term index可以快速地定位到term dictionary的某个offset，然后从这个位置再往后顺序查找。





