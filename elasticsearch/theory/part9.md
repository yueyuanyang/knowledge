## Elasticsearch初识搜索引擎-query phase和fetch phase 原理

### Elasticsearch初识搜索引擎-query phase

#### 1、query phase

- 1)搜索请求发送到某一个coordinate node（协调节点）上，构建一个priority queue，长度以paging操作from和size为准，默认为10
- 2)coordinate node将请求转发到所有shard，每个shard本地搜索，并构建一个本地的priority queue
- 3)各个shard将自己的priority queue返回给coordinate node，并构建一个全局的priority queue

