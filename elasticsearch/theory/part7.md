## ElasticSearch 的分数 (_score) 是怎么计算得出 (2.X & 5.X)

**Elasticsearch 默认是按照文档与查询的相关度(匹配度)的得分倒序返回结果的. 得分 (_score) 就越大, 表示相关性越高**.

所以, 相关度是啥? 分数又是怎么计算出来的? (全文检索和结构化的 SQL 查询不太一样, 虽然看起来结果比较'飘忽', 但也是可以追根问底的)

在 Elasticsearch 中, 标准的算法是**Term Frequency/Inverse Document Frequency**, 简写为 TF/IDF, (刚刚发布的 5.0 版本, 改为了据说更先进的 BM25 算法)

### Term Frequency

某单个关键词(term) 在某文档的某字段中出现的频率次数, 显然, 出现频率越高意味着该文档与搜索的相关度也越高

具体计算公式是 **tf(q in d) = sqrt(termFreq)**

另外, 索引的时候可以做一些设置, "index_options": "docs" 的情况下, 只考虑 term 是否出现(命中), 不考虑出现的次数.
```
PUT /my_index
{
  "mappings": {
    "doc": {
      "properties": {
        "text": {
          "type":          "string",
          "index_options": "docs"
        }
      }
    }
  }
}
```
### Inverse document frequency

某个关键词(term) 在索引(单个分片)之中出现的频次. 出现频次越高, 这个词的相关度越低. 相对的, 当某个关键词(term)在一大票的文档下面都有出现, 那么这个词在计算得分时候所占的比重就要比那些只在少部分文档出现的词所占的得分比重要低. 说的那么长一句话, 用人话来描述就是 **"物以稀为贵"**, 比如, '的', '得', 'the' 这些一般在一些文档中出现的频次都是非常高的, 因此, 这些词占的得分比重远比特殊一些的词(如'Solr', 'Docker', '哈苏')占比要低,

具体计算公式是 **idf = 1 + ln(maxDocs/(docFreq + 1))**

### Field-length Norm

字段长度, 这个字段长度越短, 那么字段里的每个词的相关度也就越大. 某个关键词(term) 在一个短的句子出现, 其得分比重比在一个长句子中出现要来的高.

具体计算公式是 **norm = 1/sqrt(numFieldTerms)**

默认每个 analyzed 的 string 都有一个 norm 值, 用来存储该字段的长度,

用 "norms": { "enabled": false } 关闭以后, 评分时, 不管文档的该字段长短如何, 得分都一样.
```
PUT /my_index
{
  "mappings": {
    "doc": {
      "properties": {
        "text": {
          "type": "string",
          "norms": { "enabled": false }
        }
      }
    }
  }
}
```
**最后的得分是三者的乘积 tf * idf * norm**

以上描述的是最原始的针对**单个关键字(term)**的搜索. 如果是有**多个搜索关键词(terms)**的时候, 还要用到的**Vector Space Model**

如果查询复杂些, 或者用到一些修改了分数的查询, 或者索引时候修改了字段的权重, 比如 function_score 之类的,计算方式也就又更复杂一些.
