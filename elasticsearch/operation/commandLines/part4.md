## Elasticsearch的查询 —— wildcard(通配符查询),regexp(正则)，prefix(前缀查询),Fuzzy Queries（模糊查询）

### 1.通配符查询(wildcard) 

wildcard是通配符查询，它和prefix查询类似，也是一个基于词条的低级别查询。但是它能够让你指定一个模式(Pattern)，而不是一个前缀(Prefix)。它使用标准的shell通配符：?用来匹配任意字符，`*`用来匹配零个或者多个字符。

```
//以下查询能够匹配包含W1F 7HW和W2F 8HW的文档：
GET /my_index/address/_search
{
    "query": {
        "wildcard": {
            "postcode": "W?F*HW" 
        }
    }
}
```

#### 2. 正则表达式查询(regexp)

regexp是正则表达式查询。假设现在你想匹配在W地域(Area)的所有邮政编码。使用前缀匹配时，以WC开头的邮政编码也会被匹配，在使用通配符查询时也会遇到类似的问题。我们只想匹配以W开头，紧跟着数字的邮政编码。使用regexp查询能够让你写下更复杂的模式：

```
GET /my_index/address/_search
{
    "query": {
        "regexp": {
            "postcode": "W[0-9].+" 
        }
    }
}


```

这个正则表达式的规定了词条需要以W开头，紧跟着一个0到9的数字，然后是一个或者多个其它字符。

wildcard和regexp查询的工作方式和prefix查询完全一样。它们也需要遍历倒排索引中的词条列表来找到所有的匹配词条，然后逐个词条地收集对应的文档ID。它们和prefix查询的唯一区别在于它们能够支持更加复杂的模式。

尽管对于前缀匹配，可以在索引期间准备你的数据让它更加高效，通配符和正则表达式匹配只能在查询期间被完成。**虽然使用场景有限，但是这些查询也有它们的用武之地**。

注意： 
prefix，wildcard以及regexp查询基于词条进行操作。如果你在一个analyzed字段上使用了它们，它们会检查字段中的每个词条，而不是整个字段。比如，假设我们的title字段中含有”Quick brown fox”，它会产生词条quick，brown和fox。

```
//这个查询能够匹配：
{ "regexp": { "title": "br.*" }}
//而不会匹配：
{ "regexp": { "title": "Qu.*" }} 
{ "regexp": { "title": "quick br*" }}
```

### 3.prefix 前缀查询

为了找到所有以 W1 开始的邮编，可以使用简单的 prefix 查询：

```
GET /my_index/address/_search
{
    "query": {
        "prefix": {
            "postcode": "W1"
        }
    }
}

```
prefix 查询是一个词级别的底层的查询，它不会在搜索之前分析查询字符串，它假定传入前缀就正是要查找的前缀。

### 4.Fuzzy Queries（模糊查询）

模糊查询可以在Match和 Multi-Match查询中使用以便解决拼写的错误，模糊度是基于Levenshteindistance计算与原单词的距离。使用如下：

```
curl -XGET 'localhost:9200/megacorp/employee/_search' -d '
{
    "query": {
        "multi_match" : {
            "query" : "rock climb",
            "fields": ["about", "interests"],
            "fuzziness": "AUTO"
        }
    },
    "_source": ["about", "interests", "first_name"],
    "size": 1
}'
```

### java api

```
SearchResponse response = client.prepareSearch("索引名称")
            .setTypes("type名称")
.setSearchType(SearchType.DFS_QUERY_THEN_FETCH)
            .setQuery(queryBuilder)
            .execute()
            .actionGet();
//1、prefix query:匹配分词前缀 如果字段没分词，就匹配整个字段前缀
QueryBuilders.prefixQuery("hotelName","花园")

//2、wildcard query:通配符查询，支持* 任意字符串；？任意一个字符
QueryBuilders.wildcardQuery("channelCode","ctr*")
QueryBuilders.wildcardQuery("channelCode","c?r?")

//3、regexp query:正则表达式匹配分词
QueryBuilders.regexpQuery()

//4、fuzzy query:分词模糊查询，通过增加fuzziness模糊属性来查询,如能够匹配hotelName为tel前或后加一个字母的文档，
fuzziness 的含义是检索的term 前后增加或减少n个单词的匹配查询
QueryBuilders.fuzzyQuery("hotelName", "tel").fuzziness(Fuzziness.ONE)
                       
```




