## Elasticsearch内核原理-深度剖析document写入原理

### 1、写入原理

- 1)数据写入buffer
- 2)commit point
- 3)buffer中的数据写入新的index segment
- 4)等待在os cache中的index segment被fsync强制刷到磁盘上
- 5)新的index segment被打开，供search使用
- 6)buffer被清空

每次commit point时，会有一个.del文件，标记了哪些segment中的哪些document，被标记为deleted了，搜索的时候，会依次查询所有的segment，从旧的到新的，比如被修改过的document，在旧的segment中，会标记为deleted，在新的segment中会有其新的数据

### 2、深入图解

![p3]()

### Elasticsearch内核原理-优化写入流程，实现NRT近实时

#### 1、疑问

每次都必须等待fsync将segment刷入磁盘，才能将segment打开供search使用，这样的话，从一个document写入，到他可以被搜索到，可能会超过1分钟，这就不是近实时搜索了。只要瓶颈在于fsync实际发生磁盘IO写数据到磁盘，是很耗时间的。

#### 2、写入流程改进如下

- 1)数据写入buffer
- 2)每隔一定时间（最慢1s），buffer中的数据被写入index segment文件，但是先写入os cache
- 3)只要segment写入os cache，那就直接打开供search使用，不立即执行commit

![p3]()

**数据写入os cache，并被打开供搜索的过程，叫做refresh，默认是每隔1秒refresh一次**，也就是说，每隔一秒就会将buffer中的数据写入一个新的index segment file，县写入os cache中，所以ES是近实时的，数据写入到可以被搜索默认是1s。

比如说，我们现在的时效性要求比较低，只要求一条数据写入ES，一分钟以后才让我们搜索到就可以了，那么就可以调整refresh interval

```
PUT my_index
{
  "settings": {
    "refresh_interval": "1m"
  }
}
```
