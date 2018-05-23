### Elasticsearch的查询 —— match_all(匹配全部),terms(精确多个),term(相等的查询),range(范围查询),filter(过滤)

### 1. match_all 直接请求全部
```
{ 
  "query" : { 
  "match_all" : {} 
}
```

### 2. Terms 精确匹配多个关键字
```
curl -XGET 'http:///localhost:9400/index_test/_search?pretty=true' -d 
{
    "query": 
      { 
       "terms" : 
           {
             "name": ["zhang", "zhangyan"]
            }
       }
     } 
} 
```

### 3. Term 严格相等的查询

严格相等的查询，例如查找中国人民，那么他就是使用中国人民去查询。使用term查询，如果该字段是不分词，只有完整的输入目标字段，才能正确的匹配。

```
curl -POST 'localhost:9200/megacorp/employee/_search' -d '
{
    "query": {
        "term" : {
            "interests": "music"
        }
    },
    "_source" : ["first_name","last_name","interests"]
}'
```

### range 根据范围查询

如果type是时间格式，可以使用内置的now表示当前，参数有from、 to、 include_lower、 include_upper、gt、gte、lt、lte。 如

```
curl -XPOST 'localhost:9200/person/worker/_search?pretty' -d '
{
    "query": {
        "range" : {
            "birthday": {
                "gte": "2017-02-01",
                "lte": "2017-05-01"
            }
        }
    },
    "_source" : ["first_name","last_name","birthday"]
}'
```

### filter 过滤

过滤查询允许我们对查询结果进行筛选。比如：我们查询about和interests中包含music关键字的员工，但是我们想过滤出birthday大于2017/02/01的结果

这个查询很快，因为不需要执行打分过程，query参数，在filter中也都存在。此外，还有比较重要的参数就是连接操作：

```
curl -XPOST :9200/megacorp/employee/_search?pretty' -d '
{
    "query": {
        "filtered": {
            "query" : {
                "multi_match": {
                    "query": "music",
                    "fields": ["about","interests"]
                }
            },
            "filter": {
                "range" : {
                    "birthday": {
                        "gte": 2017-02-01
                    }
                }
            }
        }
    },
    "_source" : ["first_name","last_name","about", "interests"]
}'
```

### java api 实现

```
QueryBuilder queryBuilder = QueryBuilders.termQuery("字段","term值");
SearchResponse response = client.prepareSearch("索引名称")
            .setTypes("type名称")
.setSearchType(SearchType.DFS_QUERY_THEN_FETCH)
            .setQuery(queryBuilder)
            .execute()
            .actionGet();
            
//1、term query:分词精确查询，查询hotelName 分词后包含 hotel的term的文档
QueryBuilders.termQuery("hotelName","hotel")

//2、terms Query:多term查询，查询hotelName包含hotel或tes 中的任何一个或多个的文档
QueryBuilders.termsQuery("hotelName","hotel","test")

//3、range query:范围查询 查询hotelNo，
QueryBuilders.rangeQuery("hotelNo")
        .gt("10143262306")                //大于
        .lt("101432623062055348221")    //小于
        .includeLower(true)             //包括下界
        .includeUpper(false);             //包括上界

//4、exist query:查询字段不为null的文档。查询字段address 不为null的数据
QueryBuilders.existsQuery("address")

//5、missing query:返回没有字段或值为null或没有值的文档。
QueryBuilders.missingQuery("accountGuid")
//java client标记该方法已经过时，推荐用exist代替 如下
QueryBuilders.boolQuery().mustNot(QueryBuilders.existsQuery("accountGuid"));

//6、idx Query:根据ID查询
QueryBuilders.idsQuery().addIds("exchange_operate_monitor_db$32293","exchange_operate_monitor_db$32294")

//7、match query:直接请求全部
QueryBuilders.matchQuery();//查询匹配会进行分词
QueryBuilders.matchAllQuery();//查询匹配会进行分词

//8、布尔查询
QueryBuilders.boolQuery()
QueryBuilders.boolQuery().must();//文档必须完全匹配条件，相当于and
QueryBuilders.boolQuery().mustNot();//文档必须不匹配条件，相当于not
QueryBuilders.boolQuery().should();//至少满足一个条件，这个文档就符合should，相当于or

//9、以一个短语的形式查询，完全匹配查询
QueryBuilders.matchPhraseQuery()

```




