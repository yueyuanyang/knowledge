## Elasticsearch匹配查询 —— match(基本查询),multi_match(多域查询),Boosting(查询权重),bool(多条件查询)

匹配查询共有三种类型:
- 分别是布尔（boolean）
- 短语（phrase）
- 短语前缀（phrase_prefix）

默认的匹配查询是布尔类型，这意味着，ElasticSearch引擎首先分析查询字符串，根据分析器对其进行分词.

### 1. match(基本查询)

查询字符串是“Microsoft Azure Party”，被分析器分词之后，产生三个小写的单词：microsoft，azure和party，然后根据分析的结果构造一个布尔查询，默认情况下，引擎内部执行的查询逻辑是：只要eventname字段值中包含有任意一个关键字microsoft、azure或party

```
{
  "query":{  
      "match":{  
         "eventname":"Microsoft Azure Party"
      }
}
```

### 2. Boosting

我们上面使用同一个搜索请求在多个field中查询，你也许想提高某个field的查询权重,在下面的例子中，我们把interests的权重调成3，这样就提高了其在结果中的权重，这样把_id=4的文档相关性大大提高了，如下：

 ```
 curl -XGET 'localhost:9200/megacorp/employee/_search' -d '
{
    "query": {
        "multi_match" : {
            "query" : "rock",
            "fields": ["about", "interests^3"]
        }
    }
}'
 ```
 Boosting不仅仅意味着计算出来的分数(calculated score)直接乘以boost factor，最终的boost value会经过归一化以及其他一些内部的优化
 
 ### 3.Bool Query
 
 我们可以在查询条件中使用AND/OR/NOT操作符，这就是布尔查询(Bool Query)。布尔查询可以接受一个must参数(等价于AND)，一个must_not参数(等价于NOT)，以及一个should参数(等价于OR)。比如，我想查询about中出现music或者climb关键字的员工，员工的名字是John，但姓氏不是smith，我们可以这么来查询：
 
```
curl -XGET 'localhost:9200/megacorp/employee/_search' -d '
{
    "query": {
        "bool": {
              "must": {
                  "bool" : { 
                      "should": [
                          { "match": { "about": "music" }},
                          { "match": { "about": "climb" }} ] 
                  }
              },
              "must": {
                  "match": { "first_nale": "John" }
              },
              "must_not": {
                  "match": {"last_name": "Smith" }
              }
          }
    }
}'
```
 
