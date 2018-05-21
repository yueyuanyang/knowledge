# Elasticsearch索引的父子关系(index parent-child)

### 目录

- 父子文档的mapping定义
- 父子文档的索引定义
- 索引子文档定义
- 搜索有相同父id的子文档 
- 通过父文档查询子文档
- 通过子文档查询父文档
- 多级文档定义
- 多级文档查询

## 简单嵌套

#### 一、父子文档的mapping

建立文档的父子关系最重要的一步是在创建索引的时候在mapping中声明哪个是父文档哪个是子文档。官网文档给了一个公司在很多城市的分支(branch)和每个分支有相关员工(employee)的例子，如果想把员工和他们工作的分公司关联起来，我们需要告诉Elasticsearch文档之间的父子关系，这里employee是child type,branch是parent type，在mapping中声明：
```
curl -XPUT  "http://192.168.0.224:9200/company" -d '
{
  "mappings": {
    "branch": {},
    "employee": {
      "_parent": {
        "type": "branch"
      }
    }
  }
}
'
```
这样我们创建了一个索引，并制定了2个type和它们之间的父子关系.

### 二、父子文档的索引

#### 2.1 索引父文档

索引父文档和索引一般的文档没有任何区别。 
准备几条公司的数据，存在company.json文件中：
```
{ "index": { "_id": "london" }}
{ "name": "London Westminster", "city": "London", "country": "UK" }
{ "index": { "_id": "liverpool" }}
{ "name": "Liverpool Central", "city": "Liverpool", "country": "UK" }
{ "index": { "_id": "paris" }}
{ "name": "Champs Élysées", "city": "Paris", "country": "France" }
```
使用Bulk端点批量导入：

```
curl -XPOST "http://192.168.0.224:9200/company/branch/_bulk?pretty" --data-binary @company.json
```
#### 2.2 索引子文档

索引子文档需要制定子文档的父ID，给子文档的每条文档设置parent属性的value为父文档id即可：

```
curl -XPUT "http://192.168.0.224:9200/company/employee/1?parent=london&pretty" -d '
{
  "name":  "Alice Smith",
  "dob":   "1970-10-24",
  "hobby": "hiking"
}
'
```

上面的操作我们新增了一条数据，/company/employee/1的父文档为/company/branch/london。 
同样可以批量索引子文档，把employee数据存入json文件中

```
{ "index": { "_id": 2, "parent": "london" }}
{ "name": "Mark Thomas", "dob": "1982-05-16", "hobby": "diving" }

{ "index": { "_id": 3, "parent": "liverpool" }}
{ "name": "Barry Smith", "dob": "1979-04-01", "hobby": "hiking" }

{ "index": { "_id": 4, "parent": "paris" }}
{ "name": "Adrien Grand", "dob": "1987-05-11", "hobby": "horses" }
```
执行bulk命令：
```
curl -XPOST "http://192.168.0.224:9200/company/employee/_bulk?pretty" --data-binary @employee.json
```

#### 三、通过子文档查询父文档

搜索含有1980年以后出生的employee的branch：

```
curl -XGET "http://192.168.0.224:9200/company/branch/_search?pretty" -d '
{           
  "query": {      
    "has_child": {       
      "type": "employee",
      "query": {  
        "range": {
          "dob": {             
            "gte": "1980-01-01"
          }
        }
      }
    }
  }
}
'
{
  "took" : 19,
  "timed_out" : false,
  "_shards" : {
    "total" : 5,
    "successful" : 5,
    "failed" : 0
  },
  "hits" : {
    "total" : 2,
    "max_score" : 1.0,
    "hits" : [ {
      "_index" : "company",
      "_type" : "branch",
      "_id" : "paris",
      "_score" : 1.0,
      "_source" : {
        "name" : "Champs Élysées",
        "city" : "Paris",
        "country" : "France"
      }
    }, {
      "_index" : "company",
      "_type" : "branch",
      "_id" : "london",
      "_score" : 1.0,
      "_source" : {
        "name" : "London Westminster",
        "city" : "London",
        "country" : "UK"
      }
    } ]
  }
}
```

查询name中含有“Alice Smith”的branch:

```
curl -XGET "http://192.168.0.224:9200/company/branch/_search?pretty" -d '
{
  "query": {
    "has_child": {
      "type":       "employee",
      "score_mode": "max",
      "query": {
        "match": {
          "name": "Alice Smith"
        }
      }
    }
  }
}'



{
  "took" : 20,
  "timed_out" : false,
  "_shards" : {
    "total" : 5,
    "successful" : 5,
    "failed" : 0
  },
  "hits" : {
    "total" : 2,
    "max_score" : 1.2422675,
    "hits" : [ {
      "_index" : "company",
      "_type" : "branch",
      "_id" : "london",
      "_score" : 1.2422675,
      "_source" : {
        "name" : "London Westminster",
        "city" : "London",
        "country" : "UK"
      }
    }, {
      "_index" : "company",
      "_type" : "branch",
      "_id" : "liverpool",
      "_score" : 0.30617762,
      "_source" : {
        "name" : "Liverpool Central",
        "city" : "Liverpool",
        "country" : "UK"
      }
    } ]
  }
}
```
搜索最少含有2个employee的branch：
```
curl -XGET "http://192.168.0.224:9200/company/branch/_search?pretty" -d '{
"query": {
    "has_child": {
      "type":         "employee",
      "min_children": 2,
      "query": {
        "match_all": {}
      }
    }
  }
}
'
{
  "took" : 17,
  "timed_out" : false,
  "_shards" : {
    "total" : 5,
    "successful" : 5,
    "failed" : 0
  },
  "hits" : {
    "total" : 1,
    "max_score" : 1.0,
    "hits" : [ {
      "_index" : "company",
      "_type" : "branch",
      "_id" : "london",
      "_score" : 1.0,
      "_source" : {
        "name" : "London Westminster",
        "city" : "London",
        "country" : "UK"
      }
    } ]
  }
}
```

**java API 实现方法**

```

 /**通过子文档查询父文档**/
 HasChildQueryBuilder hcqb2 = QueryBuilders.hasChildQuery("employee",
         QueryBuilders.matchQuery("name", "Barry Smith"));
 SearchResponse responseChild = searchRequestBuilder.setQuery(hcqb2).execute().actionGet();
 SearchHits searchHits1 = responseChild.getHits();
 if(searchHits1.getTotalHits() > 0){
     for (int i = 0; i < searchHits1.getTotalHits()-1;i++){
         System.out.println(searchHits1.getAt(0).getSourceAsString());
    
```

### 四、通过父文档查询子文档

搜搜工作在UK的employee：

```
curl -XGET "http://192.168.0.224:9200/company/employee/_search?pretty" -d '{           
  "query": {       
    "has_parent": {    
      "type": "branch", 
      "query": {  
        "match": {       
          "country": "UK"
        }
      }
    }
  }
}'

{
  "took" : 15,
  "timed_out" : false,
  "_shards" : {
    "total" : 5,
    "successful" : 5,
    "failed" : 0
  },
  "hits" : {
    "total" : 3,
    "max_score" : 1.0,
    "hits" : [ {
      "_index" : "company",
      "_type" : "employee",
      "_id" : "3",
      "_score" : 1.0,
      "_parent" : "liverpool",
      "_routing" : "liverpool",
      "_source" : {
        "name" : "Barry Smith",
        "dob" : "1979-04-01",
        "hobby" : "hiking"
      }
    }, {
      "_index" : "company",
      "_type" : "employee",
      "_id" : "1",
      "_score" : 1.0,
      "_parent" : "london",
      "_routing" : "london",
      "_source" : {
        "name" : "Alice Smith",
        "dob" : "1970-10-24",
        "hobby" : "hiking"
      }
    }, {
      "_index" : "company",
      "_type" : "employee",
      "_id" : "2",
      "_score" : 1.0,
      "_parent" : "london",
      "_routing" : "london",
      "_source" : {
        "name" : "Mark Thomas",
        "dob" : "1982-05-16",
        "hobby" : "diving"
      }
    } ]
  }
 }

```

**java API 实现方法**

```
 /***通过父文档查询子文档**/
 HasParentQueryBuilder hpqb3 = QueryBuilders.hasParentQuery("branch",QueryBuilders.matchQuery("country","UK"));
 SearchResponse responseParent = searchRequestBuilder.setQuery(hpqb3).execute().actionGet();
 SearchHits searchHitsUK = responseParent.getHits();
 if(searchHitsUK.getTotalHits() > 0){
     for (int i = 0; i < searchHitsUK.getTotalHits()-1;i++){
         System.out.println("UK : " +searchHitsUK.getAt(0).getSourceAsString());
     }
 }
```

### 五、搜索有相同父id的子文档
#### 搜索父id为london的employee：

```
curl GET company/employee/_search?pretty -d
{"query":{  
       "has_parent":{  
         "type":"branch",  
         "query":{  
           "term":{  
               "_parent":"london"  
           }  
         }   
       }  
     }  
   } 
}  
```
**java API 实现方法**

```
Client client = TransportClient.builder().build().addTransportAddress(
        new InetSocketTransportAddress(InetAddress.getByName("127.0.0.1"), 9300));
/*** 多条件查询*/
SearchRequestBuilder searchRequestBuilder = client.prepareSearch().setIndices("company");

/**搜索有相同父id的*子文档* **/
HasParentQueryBuilder hpqb1 = QueryBuilders.hasParentQuery("branch", QueryBuilders.idsQuery().ids("london"));
SearchResponse response = searchRequestBuilder.setQuery(hpqb1).execute().actionGet();

SearchHits searchHits = response.getHits();
if(searchHits.getTotalHits() > 0){
    for (int i = 0; i< searchHits.getTotalHits() -1;i++){
        System.out.println(searchHits.getAt(0).getSourceAsString());
    }
}

```

### 六、多代父子关系问题

简单的单层父子关系肯定无法满足复杂需求，所以ES允许**多代父子关系（grandchild等）**的定义，父辈和祖辈之间按照前面的方式，但此时**子辈和父辈之间需要改变一些条件，将子文档的routing参数设置为祖辈的ID**，否则有很大的可能导致三代文档不在同一分片上，继而无法通过（has_parent or has_child）语句正确搜索到。

**重点来了，为什么不设置routing参数，多代父子文档就无法正确被搜索？**

要搞清楚这个，首先我们需要了解一下什么是分片，它是ES底层的工作单元，它只保存一部分数据，我们的一份文档会被随机发送到一个分片上，一个分片是一个 Lucene 的实例，它本身就是一个完整的搜索引擎，分片只知道自己分片内部发生的事，并不能去操作其它分片，至于统筹分配任务则是ES的事。或许你并没有设置分片数量，但ES默认给你设置了5个分片，意味着文档将被“随机”存储到这5个分片中，是真的随机吗？我们来大致了解一下当一个文档被索引（存储）的时候，发生了什么事情。

难道ES不支持多代关系的父子文档？肯定不会的。官方说了，在这个时候，你需要手动多设置一个routing参数为祖辈文档id，来取代parent（注意只是取代分片路由功能，parent还用来定义父子关系，不能抛弃！）。**当你手动设置了routing参数，那么parent的分片路由功能也将失效，ES计算的时候会选取routing的值带入hash函数中**。

**总结：**
1) 将子辈和父辈之间需要改变一些条件，将子文档的routing参数设置为祖辈的ID

### 多级文档创建
```
1) POST /my_index?pretty // 祖文档
{ "index": { "_id": "london" }}
{ "name": "London Westminster", "city": "London", "country": "UK" }

2) POST /my_index/my_type?pretty //父文档
{ "index": { "_id": young, "parent": "london" }}
{ "name": "Mark Thomas", "dob": "1982-05-16", "hobby": "diving" }

2) POST /my_index/my_type/baz?parent=bar&**routing=london** // 子文档(routing 祖文档ID)
{ "index": { "_id":1, "parent": "young" }}
{ "name": "Toms", "dob": "1988-05-16", "hobby": "shuffer" }

```

**根据子文档查询祖文档**

```
{
  "query": {
    "has_child": {
      "type": "instance",
      "query": {
        "has_child": {
          "type": "instance_permission",
          "query": {
            "terms": {
              "uuid": {
                "index": "user",
                "type": "user",
                "id": "5",
                "path": "uuids"
              }
            }
          }
        }
      }
    }
  }
}
```

**java api 实现方法**

```

```














