## Elasticsearch master node、 data node、 client node的区别与各自特点

### es集群里的master node、data node和client node到底是怎么个意思，分别有何特点？

### master节点

主要功能是维护元数据，管理集群各个节点的状态，数据的导入和查询都不会走master节点，所以master节点的压力相对较小，因此master节点的内存分配也可以相对少些；但是master节点是最重要的，如果master节点挂了或者发生脑裂了，你的元数据就会发生混乱，那样你集群里的全部数据可能会发生丢失，所以一定要保证master节点的稳定性。

### data node

是负责数据的查询和导入的，它的压力会比较大，它需要分配多点的内存，选择服务器的时候最好选择配置较高的机器（大内存，双路CPU，SSD... 土豪~）；data node要是坏了，可能会丢失一小份数据。

### client node

是作为任务分发用的，它里面也会存元数据，但是它不会对元数据做任何修改。client node存在的好处是可以分担下data node的一部分压力；为什么client node能分担data node的一部分压力？因为es的查询是两层汇聚的结果，第一层是在data node上做查询结果汇聚，然后把结果发给client node，client node接收到data node发来的结果后再做第二次的汇聚，然后把最终的查询结果返回给用户；所以我们看到，client node帮忙把第二层的汇聚工作处理了，自然分担了data node的压力。

这里，我们可以举个例子，当你有个大数据查询的任务（比如上亿条查询任务量）丢给了es集群，要是没有client node，那么压力直接全丢给了data node，如果data node机器配置不足以接受这么大的查询，那么就很有可能挂掉，一旦挂掉，data node就要重新recover，重新reblance，这是一个异常恢复的过程，这个过程的结果就是导致es集群服务停止... 但是如果你有client node，任务会先丢给client node，client node要是处理不来，顶多就是client node停止了，不会影响到data node，es集群也不会走异常恢复。

对于es 集群为何要设计这三种角色的节点，也是从分层逻辑去考虑的，只有把相关功能和角色划分清楚了，每种node各尽其责，才能发挥出分布式集群的效果。

## Elasticsearch集群的脑裂问题

所谓脑裂问题（类似于精神分裂），就是同一个集群中的不同节点，对于集群的状态有了不一样的理解。

今天，Elasticsearch集群出现了查询极端缓慢的情况，通过以下命令查看集群状态：

> curl -XGET 'es-1:9200/_cluster/health'

发现，集群的总体状态是red，本来9个节点的集群，在结果中只显示了4个；但是，将请求发向不同的节点之后，我却发现即使是总体状态是red的，但是可用的节点数量却不一致。

正常情况下，集群中的所有的节点，应该对集群中master的选择是一致的，这样获得的状态信息也应该是一致的，不一致的状态信息，说明不同的节点对master节点的选择出现了异常——也就是所谓的脑裂问题。这样的脑裂状态直接让节点失去了集群的正确状态，导致集群不能正常工作。

**可能导致的原因**：
```
1. 网络：由于是内网通信，网络通信问题造成某些节点认为master死掉，而另选master的可能性较小；进而检查Ganglia集群监控，也没有发现异常的内网流量，故此原因可以排除。

2. 节点负载：由于master节点与data节点都是混合在一起的，所以当工作节点的负载较大（确实也较大）时，导致对应的ES实例停止响应，而这台服务器如果正充当着master节点的身份，那么一部分节点就会认为这个master节点失效了，故重新选举新的节点，这时就出现了脑裂；同时由于data节点上ES进程占用的内存较大，较大规模的内存回收操作也能造成ES进程失去响应。所以，这个原因的可能性应该是最大的。
```

### 应对问题的办法：

1. 对应于上面的分析，推测出原因应该是由于节点负载导致了master进程停止响应，继而导致了部分节点对于master的选择出现了分歧。为此，一个直观的解决方案便是将master节点与data节点分离。为此，我们添加了三台服务器进入ES集群，不过它们的角色只是master节点，不担任存储和搜索的角色，故它们是相对轻量级的进程。可以通过以下配置来限制其角色：
```
node.master: true  
node.data: false 
```
当然，其它的节点就不能再担任master了，把上面的配置反过来即可。这样就做到了将master节点与data节点分离。当然，为了使新加入的节点快速确定master位置，可以将data节点的默认的master发现方式有multicast修改为unicast：
```
discovery.zen.ping.multicast.enabled: false  
discovery.zen.ping.unicast.hosts: ["master1", "master2", "master3"]  
```
2. 还有两个直观的参数可以减缓脑裂问题的出现：

discovery.zen.ping_timeout（默认值是3秒）：默认情况下，一个节点会认为，如果master节点在3秒之内没有应答，那么这个节点就是死掉了，而增加这个值，会增加节点等待响应的时间，从一定程度上会减少误判。

discovery.zen.minimum_master_nodes（默认是1）：这个参数控制的是，一个节点需要看到的具有master节点资格的最小数量，然后才能在集群中做操作。官方的推荐值是(N/2)+1，其中N是具有master资格的节点的数量（我们的情况是3，因此这个参数设置为2，但对于只有2个节点的情况，设置为2就有些问题了，一个节点DOWN掉后，你肯定连不上2台服务器了，这点需要注意）。




