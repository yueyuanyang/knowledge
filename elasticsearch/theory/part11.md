http://dsc.imoxiu.com/columbus/rs/test/spider?product=ktime&content={%22code%22:200,data:[{%22product%22:%22kime%22},{%22code%22:200}]}

{
    "code": 200
    "date":[
		{
			"docid":55645445465455,
			"product":"ktime",
			"ctime":158971564,
			"title" : "",
			"desc" : "",
			"cate1" : "",
			"cate2" : "",
			"cate3" : "",
			"cate4" : "",
			"content":{
			  "title": "",
			  "desc" : "",
			  "url":"https://share.izuiyou.com/detail/52643612?&app=zuiyou",
			  "images":"[[{"img": "https://file.izuiyou.com/img/view/id/283001530/sz/148"}]]",
			  "originId":52643612
			}
		}
	]
}


http://tools.router.test.imoxiu.cn/tools/comic/cate?id=1&token=testtoken&timestamp=1508561799&package=com.haolan.comics&sign=testsign&mobileInfo={%22dpi%22:%22540*960%22,%22imei%22:%22864286020185427%22,%22imsi%22:%22EIhPa47c5tIIQ0WwvXt%22,%22model%22:%22L827%22,%22release%22:%224.3%22,%22ver%22:%221.2.3.0%22,%22vcode%22:13

需求1：计算所有商品的价格总和
GET /ecommerce/product/_search
{
   "size":0 -- 返回列表(商品的记录)
   "aggs": { --- 内置函数
       "sum_price" : { -- 聚合别名
	       "sum": { -- 内置函数
		       "field":"price"
		   }
	   }   
   }
}

需求2：找出最贵的商品
GET /ecommerce/product/_search
{
   "aggs": {
      "max_price": {
	      "max":{
		     "field" : "price"
		  }
	  }
   }
}

需求3：计算价钱的平均值
GET /ecommerce/product/_search
{
   "size":0
   "aggs": {
       "avg_aggs": {
	       "avg" : {
		     "field":"price"
		   }
	   }
   }
}

需求4：计算每个tag下的商品数量
需要用terms标签，需要先将所用字段正排索引下，否则报错
正排索引：

PUT /ecommerce/_mapping/product
{
  "properties": {
      "tags": {
	     "type": "text",
		 "fielddata":true
	  }
  }
}

GET /ecommerce/product/_search
{
    "size":0
	"aggs": {
	    "group_by_tag" : {
		    "terms": {
			   "field": "tags"
			}
		}
	}
}

需求5：对名称中包含yagao的商品，计算每个tag下的商品数量
GET /ecommerce/product/_search
{
   "query": {
      "match" : {
	      "name" : "yagao"
	  }
   },
   "aggs": {
       "group_by_tags" : {
	   
	       "terms" : {
		      "field" : "tags"
		   }
	   }
   }
}

需求6：先按照tags分组，在计算每组的商品的平均价格
GET /ecommerce/product/_search
{
    "size": 0
	"aggs" : {
	  "group_by_tag" : {
	      "terms": {
		     "field":"tags"
		  },
		 "aggs": {
			"avg_aggs" : {
			   "avg" : {
				"field": "price"
			   }
			}
		 }
	  }
   }
}

需求7：先按照tags分组，在计算每组的商品的平均价格，并按照价格倒序排序
GET /ecommerce/product/_search
{
   "size" : 0,
   "aggs" ： {
      "group_by_tag" : {
	     "terms" : {
		    "field": "tags"
			"order" : {
			  "avg_price" : "desc"
			}
		 },
		 "aggs" : {
		    "avg_price": {
			   "avg" : {
			      "field": "price"
			   }
			
			}
		 }
	  }
   }
}

需求8：按照指定的价格范围区间进行分组，然后在每组内再按照tags进行分组，最后在计算每组的平均价格
GET /ecommerce/product/_search
{
  "size": 0,
  "aggs": {
    "range_by_price": {
      "range": {
        "field": "price",
        "ranges": [
          {
            "from": 0,
            "to": 20
          },
          {
            "from": 20,
            "to": 40
          },
          {
            "from": 40,
            "to": 50
          }
        ]
      },
      "aggs": {
        "group_by_tags": {
          "terms": {
            "field": "tags"
          },
          "aggs": {
            "avg_price": {
              "avg": {
                "field": "price"
              }
            }
          }
        }
      }
    }
  }
}

需求9： mappings 设置
PUT /website
{
    "mappings": {
	    "article": {
		    "properties": {
			    "author_id" : {
				    "type":long
				},
				"title": {
			        "type": "text",
                    "analyzer" : "english"
				},
                "content" : {
				    "type": "text"
				},
                "post_date" : {
				   "type":"date",
				   "index": "not_analyzed"
				
				}				
			}
		
		}
	
	}
}

-------------------------------------------------------------

需求10：搜索全部(指定account_number,blance返回两个字段)
GET /bank/_search
{
  "query": {
     "match_all": {}
	 "_source":["account_number","blance"]
  }
}

需求11：指定一个字段进行搜索
GET /bank/_search
{
    "query": {
	   "match": {
	      "address": "mill"
	   }
	}
}

需求12：解析搜索
GET /bank/_search
{
   "query" : {
       "match_phrase": {
	      "address":"mill"
	   }
   }
}

需求13: 同一个field,搜索多个值
GET /bank/_search
{
   "query": {
      "bool" : {
	     "must": {
		    "match" : {"address" : "mill"}
			"match" : {"address" : "lane"}
		 }
	  
	  }
   
   }
}

------------------ 过滤搜索 ----------------------------

需求13:  查找balance大小20000, 小于3000的文档
GET /bank/_search
{
   "query" : {
      "bool" : {
	     "must" : { "match_all": {}}
	     "filter" : {
		    "range" : {
			    "balance" : {
				  "gte" : 20000,
				  "lte" : 30000
 				}
			}
		 }
	  }
   }
}


需求14 : 聚合操作
GET /bank/_search
{
    "size" : 0,
	"aggs" : {
	   "group_by_state" : {
	      "terms" : {
		     "field" : "state.keyword"
		  }
	   }
	}
}

-------------------------ES 安装和配置------------------------------- 
需求15：ES 后台运行
./bin/elasticsearch -d -p pid  --- 信息将在$ES_HOME/logs/目录下找到
kill `cat pid` --- 关闭es

需求16：ES 命令行集群配置
./bin/elasticsearch -d -Ecluster.name=my_cluster -Enode.name=node_1

需求17 ：ES JVM命令行配置
1) 配置文件 config/jvm.options
-Xms2g #设置最小堆的值为2g
-Xmx2g #设置组大堆的值为2g

2) 环境变量设置
ES_JAVA_OPTS="-Xms2g -Xmx2g" ./bin/elasticsearch #设置最小和最大堆的值为2g
ES_JAVA_OPTS="-Xms4000m -Xmx4000m" ./bin/elasticsearch  #设置最小和最大堆的值为 4000M

----------------------------- ES  配置管理 ----------------------------------------
需求18 ：ES 集群配置管理
path.data  -- 数据存放位置
path.logs  -- 数据日志存放位置
cluster.name -- 集群名字(相同的集群名称，会被划分为同一个集群，命名：最好与业务相关)
node.name -- 建议使用HOSTNAME
network.host --绑定到非回送地址

需求19 ：discovery settings(发现设置)
Elasticsearch使用名为"Zen Discovery"的自定义发现实现进行节点到节点的群集和master选举.
在投入生产之前，应该配置两个重要的发现设置。
discovery.zen.ping.unicast(单播).hosts -- master选举的node节点
discovery.zen.minimum_master_nodes -- 一个节点需要看到的具有master节点资格的最小数量，然后才能在集群中做操作((master_eligible_nodes / 2) + 1)

需求20 ：settings the heap size(设置heap大小)
1) 将最小堆大小(Xms)和最大堆大小(Xmx)设置为彼此相等。
2) Elasticsearch可用的堆越多，可用于缓存的内存就越多。但请注意，太多的堆可能会使您长时间垃圾收集暂停。
3) 将Xmx设置为不超过物理RAM的50％，以确保有足够的物理RAM留给内核文件系统缓存。
4) 不要将Xmx设置为JVM用于压缩对象指针（压缩oops）的临界值以上; 确切的截止值有所不同，但接近32 GB。 
您可以通过在日志中查找一行来验证您是否处于限制之下
5) 更好的是，尽量保持低于基于零的压缩oops的阈值; 准确的截止值会有所不同，但大多数系统上的26 GB是安全的，但在某些系统上可能高达30 GB。 
您可以通过启动带有JVM选项-XX：+ UnlockDiagnosticVMOptions -XX：+ PrintCompressedOopsMode的Elasticsearch并查找如下所示的行来验证您是否处于限制之下：
-XX:HeapDumpPath= 设置dump输出路径
-XX:ErrorFile=... 设置Error

需求21 ：关闭 ES 集群配置
./bin/elasticsearch -p /tmp/elasticsearch-pid -d
cat /tmp/elasticsearch-pid && echo
15516
kill -SIGTERM 15516
