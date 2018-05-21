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
