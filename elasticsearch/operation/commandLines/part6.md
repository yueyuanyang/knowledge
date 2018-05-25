## Elasticsearch匹配查询 —— Bool Query(布尔查询),Match Phrase Query(匹配短语查询),Match Phrase Prefix Query(匹配短语前缀查询)

Elasticsearch FullText Query 匹配查询共有三种类型:
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
 **java api**
 
 查询的内容会通过分词，分词后的数据进行检索，只要包含其中一个分词就会被检索出来
 ``` 
 QueryBuilders.matchQuery("hotelName", "test林")
 ```
**Multi-field 查询**

正如我们之前所看到的，想在一个搜索中查询多个 document field （比如使用同一个查询关键字同时在title和summary中查询），你可以使用multi_match查询，使用如下：

```
curl -XGET 'localhost:9200/megacorp/employee/_search' -d 
{
    "query": {
        "multi_match" : {
            "query" : "rock",
            "fields": ["about", "interests"]
        }
    }
}'

```

**java api —— multi-match query**

多字段检索，检索相同的内容

type对内容相关度会有一定的影响，根据你的应用场景来分析你的查询写法

type 默认是 best_fields

**best_fields** ：score为匹配的字段的最大值；

**most_fields** ：score为每个字段分值的总和；

**cross_field** ：score为将字段合并为一个整个大字段，获得的分值
```
QueryBuilders.multiMatchQuery("beijing test","hotelName","hotelNo").operator(MatchQueryBuilder.Operator.OR)
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

**common term query**

一种略高级的查询，充分考虑了stop-word的低优先级，提高了查询精确性。他将查询短语分词，将分词后的term分为高频词和低频词

高频词（也可以理解为stopwords）比如 的，个，是 ；英文的 the is 等 无意义且出现频率极高的词

**低频词**就是我们要时间查询的词，比如酒店 宾馆等

查询过程：通过匹配低频词检索出我们需要的数据，然后在这个基础上继续匹配高频词，既能完成检索数据又能充分匹配高频词

因为我们可能搜索词 happy 和 not happy 如果不匹配高频词 搜索的结果将会是一样的 显然这不是我们需要的

举例如果想要能够匹配如下 "hotelName":"新增的第一个酒店"
```
QueryBuilders.commonTermsQuery("hotelName","的酒个店").cutoffFrequency(0.001f)
cutoffFrequency 设置高频词的score因子
```

**query string**

```
curl -XGET 'localhost:9200/megacorp/employee/_search' -d '
{
    "query": {
        "query_string" : {
            "query": "(saerch~1 algorithm~1) AND (grant ingersoll) OR (tom morton)",
            "fields": ["_all", "summary^2"]
        }
    },
    "_source": [ "title", "summary", "authors" ],
    "highlight": {
        "fields" : {
            "summary" : {}
        }
    }
}'
```
默认是全部field进行搜索_all;可以明确指定某个field;支持分词等

QueryString 被分词 默认分词之间是OR 关系， 支持 AND OR 来决定每个term之间的关系；同时 支持通配符，正则 等操作

如下查询 hotelName 字段 查询 QueryString 为 "四 AND 酒 AND 店 " 查询结果是 hotelName 同时包含 "四" "酒" "店" 如果不明确 AND 则是OR的关系，包含任意一个term即被命中

```
QueryBuilder queryBuilder = QueryBuilders.queryStringQuery("四 AND 酒 AND  店 ").defaultField("hotelName");
```

**simple query string**

简单查询
```
curl -POST 'localhost:9200/megacorp/employee/_search' -d '
{
    "query": {
        "simple_query_string" : {
        "query": "(saerch~1 algorithm~1) + (grant ingersoll) | (tom morton)",
        "fields": ["_all", "summary^2"]
        }
    },
    "_source": [ "title", "summary", "authors" ],
    "highlight": {
        "fields" : {
            "summary" : {}
        }
    }
}'
```

支持符号操作，不像query string那样支持正则等表达式

**支持的符号如**： + 表示 AND ， | 表示 OR ， - 表示 否 还有其他支持符号就查看文档吧

**flag**明确指定支持的符号有哪些，

查询 hotelName 字段 查询 simpleQueryString 为 "四+酒+店 " 查询结果是 hotelName 同时包含 "四" "酒" "店"
```
QueryBuilders.simpleQueryStringQuery("四+酒+店").field("hotelName")
```
下面看下flag 的作用，flag 明确指明 支持OR操作，此时 + 不生效，三个词是默认的 OR关系
```
QueryBuilders.simpleQueryStringQuery("四+酒+店").field("hotelName").flags(SimpleQueryStringFlag.OR)
```
 
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

 **java api —— matchPhraseQuery**
 
查询的内容通过分词，严格按照分词的出现的顺序进行查询，也就是必须包含所有分词，且出现数据一致
```
QueryBuilders.matchPhraseQuery("hotelName", "智售试酒").slop(2)
```
**总结**：

是否分词后查询，前提（查询的字段是进行分词索引的，如果不是分词索引则不生效）

例如 hotelName 包含值： 'test' 'test three' 'three' 'test one three' '林test' 'test yang three'

查询短语为 test three

对比**matchQuery**： 能够查询出来 全部内容

mathPhraseQuery ：只能够查询出来 test three 这一个数据 因为'test yang three' 三个词包含了 'test' 和 'three' 但是他们之间有一个 'yang'这个分词所以没匹配上

**那如何让查询也把'test yang three'呢？**
```
slop(1)，1含义是两个分词之间可以最多还有其他1个分词，这时就能够查询到'test yang three'
如果想也查询到'test yang ss three' slop则应该slop(2) 
```
其他参数可以自己摸索，如设置分词器等。设置operator and或者or 控制分词的查询的关系

//分词后AND关系,必须同时包含所有分词
```
QueryBuilders.matchQuery("hotelName","test林").operator(MatchQueryBuilder.Operator.AND)
```
//分词后OR关系，至少包含一个分词
```
QueryBuilders.matchQuery("hotelName","test林").operator(MatchQueryBuilder.Operator.OR)
```

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


