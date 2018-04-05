## 设计模式 —— 适配器

**定义：**适配器模式将某个类的接口转换成客户端期望的另一个接口表示，主的目的是兼容性，让原本因接口不匹配不能一起工作的两个类可以协同工作。其别名为包装器(Wrapper)。

属于结构型模式

主要分为三类：
- 类适配器模式
- 对象的适配器模式
- 接口的适配器模式。

本文定义：

需要被适配的类、接口、对象（我们有的），简称 src（source） 
最终需要的输出（我们想要的），简称 dst (destination，即Target) 
适配器称之为 Adapter 。

一句话描述适配器模式的感觉： src->Adapter->dst,即src以某种形式（三种形式分别对应三种适配器模式）给到Adapter里，最终转化成了dst。

拿我们Android开发最熟悉的展示列表数据的三大控件：ListView，GridView，RecyclerView的Adapter来说，它们三个控件需要的是View(dst),而我们有的一般是datas(src),所以适配器Adapter就是完成了数据源datas 转化成 ItemView的工作。 
带入src->Adapter->dst中，即datas->Adapter->View.
