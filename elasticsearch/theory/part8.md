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
