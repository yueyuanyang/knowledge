## Elasticsearch初识搜索引擎-query phase和fetch phase 原理

### Elasticsearch初识搜索引擎-query phase

#### 1、query phase

- 1)搜索请求发送到某一个coordinate node（协调节点）上，构建一个priority queue，长度以paging操作from和size为准，默认为10
- 2)coordinate node将请求转发到所有shard，每个shard本地搜索，并构建一个本地的priority queue
- 3)各个shard将自己的priority queue返回给coordinate node，并构建一个全局的priority queue

#### 2、图解query phase

![p1](https://github.com/yueyuanyang/knowledge/blob/master/elasticsearch/img/p1.png)

#### 3、replica shard如何提升搜索吞吐量？

一次请求要打到所有shard的一个replica或primary上去，若每个shard都有多个replica，那么同时并发过来的搜索请求可以同时打到其他的replica上去

### Elasticsearch初识搜索引擎-fetch phase

#### 1、fetch phase工作流程
- 1)coordinate node构建完priority queue之后，就发送mget请求去所有shard上获取对应的document
- 2)各个shard将document返回给coordinate node
- 3)coordinate node 将合并后的document结果返回给client客户端

#### 2、图解

![p2](https://github.com/yueyuanyang/knowledge/blob/master/elasticsearch/img/p2.png)

#### 3、疑问

问：若搜索的时候不加from和size怎么办？

答：一般搜索，如果不加from和size，就默认前10条数据，按照_score排序

