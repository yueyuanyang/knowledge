### 创建mapping

```
curl -XPOST "http://127.0.0.1:9200/productindex/product/_mapping?pretty" -d ' 
{
    "product": {
            "properties": {
                "title": {
                    "type": "string",
                    "store": "yes"
                },
                "description": {
                    "type": "string",
                    "index": "not_analyzed"
                },
                "price": {
                    "type": "double"
                },
                "onSale": {
                    "type": "boolean"
                },
                "type": {
                    "type": "integer"
                },
                "createDate": {
                    "type": "date"
                }
            }
        }
  }
'

{
  "acknowledged" : true
}
```

### 给productindex加了一个type，并写入了product的mapping信息

```
curl -XGET "http://127.0.0.1:9200/productindex/_mapping?pretty"
{
  "productindex" : {
    "mappings" : {
      "product" : {
        "properties" : {
          "createDate" : {
            "type" : "date",
            "format" : "strict_date_optional_time||epoch_millis"
          },
          "description" : {
            "type" : "string",
            "index" : "not_analyzed"
          },
          "onSale" : {
            "type" : "boolean"
          },
          "price" : {
            "type" : "double"
          },
          "title" : {
            "type" : "string",
            "store" : true
          },
          "type" : {
            "type" : "integer"
          }
        }
      }
    }
  }
}
```
### 不能修改一个已经存在的字段的类型

原因:  为什么不能修改一个字段的type？原因是一个字段的类型修改以后，那么该字段的所有数据都需要重新索引。Elasticsearch底层使用的是lucene库，字段类型修改以后索引和搜索要涉及分词方式等操作，不允许修改类型在我看来是符合lucene机制的。







