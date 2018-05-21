### Elasticsearch索引的父子关系(index parent-child)

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

####2.1索引父文档

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
#### 2.2索引子文档

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








