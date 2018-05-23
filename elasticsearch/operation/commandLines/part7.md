## Elasticsearch DSL查询语言 —— size(返回个数),from(分页开始),to(分页结束),sort(排序),source(返回fields)

### 1.size(返回个数)
```
curl -XPOST 'localhost:9200/bank/_search?pretty' -d '
{
  "query": { "match_all": {} },
  "size": 1
}'
```

### 2.sort(排序)
下面的命令指定了文档返回的排序方式：
```
curl -XPOST 'localhost:9200/bank/_search?pretty' -d '
{
  "query": { "match_all": {} },
  "sort": { "balance": { "order": "desc" } }
}'
```

### 3. from分页
下面的命令请求了第10-20的文档。
```
curl -XPOST 'localhost:9200/bank/_search?pretty' -d '
{
  "query": { "match_all": {} },
  "from": 10,
  "size": 10
}'
```

### 4. _source返回特定的字段
之前的返回数据都是返回文档的所有内容，这种对于网络的开销肯定是有影响的，下面的例子就指定了返回特定的字段：
```
curl -XPOST 'localhost:9200/bank/_search?pretty' -d '
{
  "query": { "match_all": {} },
  "_source": ["account_number", "balance"]
}'
```


