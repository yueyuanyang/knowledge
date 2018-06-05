## 图形数据库的建模技巧

### Node
 - entites(identity)
 - properties
   Attributes+metadata
   
### RelationShips
- connect entites
- Strusture
- properties
 Strength/weight/quality+metadata
 
### label 
- Role
- Group Node

### 性质对应关系
普通名词 -> 标签(label)

user ->:User

email ->:Email

动词带动对象 -> 关系(RelationShip)

sent -> SENT

wrote -> WROTE

正确的名词 -> 属性节点

Ian ->({name:"Ian"})

### Atrributes:Property or Ralationoship?
![neo4j2](https://github.com/yueyuanyang/knowledge/blob/master/neo4j/img/neo4j-2.png)

### 当什么时候使用关系(RelationShips)

you need to specify the weight,strength,or some other quality of the relationship(您需要指定关系的weight(重量，体重; 重担，重任; 重要; [统] 权，加重值，权重)，strength(力量; 优点，长处)或其他quality(质量，品质; 美质，优点; 才能，能力，技能，素养; 品种))
- friendShip strength(友谊的权重)
- proficieiency in a skill(熟练掌握技巧)

**AND/OR**

attribute value comprises a complex value type:(属性值包含复数值类型：)
- Address(first line,second line ,zip code etc) 地址（第一线，第二线，邮政编码等）

**AND/OR**

Atrribute values arw interconnected(属性值是相互关联的)
- Taxonomy of skills(技能的分类)

### modeling skills as nodes

![neo4j3](https://github.com/yueyuanyang/knowledge/blob/master/neo4j/img/neo4j-3.png)

### 什么时候使用属性(User properties when)

there is no  need to quality the relationship(没有必要`quality(质量，品质; 美质，优点; 才能，能力，技能，素养; 品种)`关系)

**and**

atrribute value is a simple value type(属性值是一个简单的`值`类型)

### align Relationships with Use cases(将关系与用例对齐)

- Relationships are the "royal road" into the graph(关系是图中的“皇室之路”)
- when querying,well-named relationships help discover only what is absolutely necessary
and eliminate unnecessary portions of the graph fro m consideration (当查询时，优质的命名关系只能帮助发现绝对必要的东西并考虑消除图表中不必要的部分)
- connect the same things(nodes) in differnt ways(differently named relationships)
Specialize the grap for different use case(以不同方式连接相同的东西（节点）（不同命名的关系）
专注于不同用例的图形)


### common graph structures (共同的图形结构)

connect more then 2 nodes in a single context

- lan bought a book in waterstones

N-ary relationships 
- sue invited sarah,Bob and charlie to her party

Relate somethging to a relationship

![neo4j](https://github.com/yueyuanyang/knowledge/blob/master/neo4j/img/neo4j-4.png)

### Rich Context,Multiple Dimensions

![neo4j](https://github.com/yueyuanyang/knowledge/blob/master/neo4j/img/neo4j-5.png)

### Trap : Verbing
- Be as simple as passible
- but beware verbing

Lanahage habit:varb -> noun

send an email -> Email

Search Google -> google

### Example:[:EMAILED] to(:email)

![neo4j](https://github.com/yueyuanyang/knowledge/blob/master/neo4j/img/neo4j-6.png)

### considerations(注意事项)
- an intermediate node provides flexibility,It allow more then two nodes to be  connected in a single context(中间节点提供了灵活性
它允许多于两个节点在单个环境中连接)
- but it can be overkill and will have an impact on performance(但它可能会过度，并会对性能产生影响)


### Linked Lists
- Entites are linked in a sequence (实体按顺序链接)
- You need to traverse the sequence (你需要遍历序列)
- You may need to indentify the beginning or end(first/last,earliest/latest,etc.)(您可能需要识别开始或结束（第一个/最后一个，最早/最近等）)
- examples(Event stream;Episodes od a TV series;Job history)(例子（事件流;电视连续剧的集数;工作经历）)








