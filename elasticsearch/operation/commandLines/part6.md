## Elasticsearch匹配查询 —— Bool Query(布尔查询),Match Phrase Query(匹配短语查询),Match Phrase Prefix Query(匹配短语前缀查询)

匹配查询共有三种类型:
- 分别是布尔（boolean）
- 短语（phrase）
- 短语前缀（phrase_prefix）

默认的匹配查询是布尔类型，这意味着，ElasticSearch引擎首先分析查询字符串，根据分析器对其进行分词.

### 基本查询

**match(基本查询)**

查询字符串是“Microsoft Azure Party”，被分析器分词之后，产生三个小写的单词：microsoft，azure和party，然后根据分析的结果构造一个布尔查询，默认情况下，引擎内部执行的查询逻辑是：只要eventname字段值中包含有任意一个关键字microsoft、azure或party

```
{
  "query":{  
      "match":{  
         "eventname":"Microsoft Azure Party"
      }
}
```
**Multi-field 查询**

正如我们之前所看到的，想在一个搜索中查询多个 document field （比如使用同一个查询关键字同时在title和summary中查询），你可以使用multi_match查询，使用如下：

```
curl -XGET 'localhost:9200/megacorp/employee/_search' -d '
{
    "query": {
        "multi_match" : {
            "query" : "rock",
            "fields": ["about", "interests"]
        }
    }
}'
```

**Boosting**

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
 
 ### 1.Bool Query(布尔查询)
 
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


### 2. Match Phrase Query(匹配短语查询)

匹配短语查询要求查询字符串中的trems要么都出现Document中、要么trems按照输入顺序依次出现在结果中。在默认情况下，查询输入的trems必须在搜索字符串紧挨着出现，否则将查询不到。不过我们可以指定slop参数，来控制输入的trems之间有多少个单词仍然能够搜索到，如下所示：

```
curl -XGET 'localhost:9200/megacorp/employee/_search' -d '
{
    "query": {
        "multi_match": {
            "query": "climb rock",
            "fields": [
                "about",
                "interests"
            ],
            "type": "phrase",
            "slop": 3
        }
    },
    "_source": [
        "title",
        "about",
        "interests"
    ]
}
```
从上面的例子可以看出，id为4的document被搜索（about字段里面精确匹配到了climb rock），并且分数比较高；而id为1的document也被搜索到了，虽然其about中的climb和rock单词并不是紧挨着的，但是我们指定了slop属性，所以被搜索到了。如果我们将"slop":3条件删除，那么id为1的文档将不会被搜索到。

### 3.Match Phrase Prefix Query(匹配短语前缀查询)

匹配短语前缀查询可以指定单词的一部分字符前缀即可查询到该单词，和match phrase query一样我们也可以指定slop参数；同时其还支持max_expansions参数限制被匹配到的terms数量来减少资源的使用,使用如下：

```
curl -XGET 'localhost:9200/megacorp/employee/_search' -d '
{
    "query": {
        "match_phrase_prefix": {
            "summary": {
                "query": "cli ro",
                "slop": 3,
                "max_expansions": 10
            }
        }
    },
    "_source": [
        "about",
        "interests",
        "first_name"
    ]
}'
```

 
 
 
