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

### 向量空间模型

**向量空间模型提供了一种多词条查询的比较方法**。它的输出是一个代表了文档和查询之间匹配程度的分值。为了计算该分值，文档和查询都被表示成向量。

一个向量实际上就是一个包含了数值的一维数组，比如：

> [1,2,5,22,3,8]

在向量空间模型中，向量中的每个数值都是由TF/IDF计算得到的一个词条的权重。

**TIP**
```
尽管TF/IDF是在向量空间模型中默认被用来计算词条权重的方法，它并不是唯一可用的方法。在ES中，其它诸如Okapi-BM25的计算模型也是可用的。TF/IDF由于其简洁性，高效性，产生的搜索结果的高质量而经受了时间的考验，从而被当做是默认方法。
```
假设我们查询了"happy hippopotamus"。一个像happy这样的常见单词的权重是较低的，然而像hippopotamus这样的罕见单词则拥有较高的权重。假设happy的权重为2而hippopotamus的权重为5。我们可以使用坐标来表达这个简单的二维向量 - [2, 5] - 一条从坐标(0, 0)到坐标(2, 5)的直线，如下所示：

![es_1](https://github.com/yueyuanyang/knowledge/blob/master/elasticsearch/img/es_1.png)

现在，假设我们有三份文档：

- I am happy in summer.
- After Christmas I’m a hippopotamus.
- The happy hippopotamus helped Harry.

我们可以为每份文档创建一个类似的向量，它由每个查询词条的权重组成 - 也就是出现在文档中的词条happy和hippopotamus，然后将它绘制在坐标中，如下图：

- 文档1：(happy,____________) — [2,0]
- 文档2：( ___ ,hippopotamus) — [0,5]
- 文档3：(happy,hippopotamus) — [2,5]

![es_2](https://github.com/yueyuanyang/knowledge/blob/master/elasticsearch/img/es_2.png)

向量的一个很棒的性质是它们能够被比较。通过测量查询向量和文档向量间的角度，我们可以给每份文档计算一个相关度分值。文档1和查询之间的角度较大，因此它的相关度较低。文档2和查询更靠近，所以它的相关度更高，而文档3和查询之间则是一个完美的匹配。

**TIP**

```
实际上，只有二维向量(使用两个词条的查询)才能够被简单地表示在坐标中。
幸运的是，线性代数 - 数学的一个分支，能够处理向量 - 提供了用来比较多维向量间角度的工具，
这意味着我们能够使用上述原理对包含很多词条的查询进行处理。
```

### Elasticsearch 5 (Lucene 6) 的 BM25 算法

Elasticsearch 前不久发布了 5.0 版本, 基于 Lucene 6, 默认使用了 BM25 评分算法.

**BM25** 的 **BM 是缩写自 Best Match**, 25 貌似是经过 25 次迭代调整之后得出的算法. 它也是基于 **TF/IDF** 进化来的. Wikipedia 那个公式看起来很吓唬人, 尤其是那个求和符号, 不过分解开来也是比较好理解的.

总体而言, 主要还是分三部分, **TF - IDF - Document Length**

![es_3](https://github.com/yueyuanyang/knowledge/blob/master/elasticsearch/img/es_3.png)

IDF 还是和之前的一样. 公式 IDF(q) = 1 + ln(maxDocs/(docFreq + 1))

**f(q, D) 是 tf(term frequency)**

|d| 是文档的长度, avgdl 是平均文档长度.

先不看 IDF 和 Document Length 的部分, 变成 **tf * (k + 1) / (tf + k)**

相比传统的 TF/IDF **(tf(q in d) = sqrt(termFreq))** 而言, **BM25 抑制了 tf 对整体评分的影响程度**, 虽然同样都是增函数, 但是, BM25 中, tf 越大, 带来的影响无限趋近于 (k + 1), 这里 k 值通常取 [1.2, 2], 而传统的 TF/IDF 则会没有临界点的无限增长.

而**文档长度的影响**, 同样的, 可以看到, 命中搜索词的情况下, 文档越短, 相关性越高, 具体影响程度又可以由公式中的 b 来调整, 当设值为 0 的时候, 就跟之前 'TF/IDF' 那篇提到的 "norms": { "enabled": false } 一样, 忽略文档长度的影响.

综合起来,

k = 1.2

b = 0.75

**idf * (tf * (k + 1)) / (tf + k * (1 - b + b * (|d|/avgdl)))**

最后再对所有的 terms 求和. 就是 Elasticsearch 5 中一般查询的得分了.
