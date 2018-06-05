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

you need to specify the weight,strength,or some other quality of the relationship(您需要指定关系的重量，强度或其他质量)
- friendShip strength(友谊的力量)
- proficieiency in a skill(熟练掌握技巧)

**AND/OR**

attribute value comprises a complex value type:(属性值包含复数值类型：)
- Address(first line,second line ,zip code etc) 地址（第一行，第二行，邮政编码等）

**AND/OR**

Atrribute values arw interconnected(属性值是相互关联的)
- Taxonomy of skills(技能的分类)

### modeling skills as nodes

![neo4j3](https://github.com/yueyuanyang/knowledge/blob/master/neo4j/img/neo4j-3.png)

### 什么时候使用属性(User properties when)

there is no  need to quality the relationship(没有必要质量关系)

**and**

atrribute value is a simple value type(属性值是一个简单的值类型)

### align Relationships with Use cases

- Relationships are the "royal road" into the graph
- when querying,well-named relationships help discover only what is absolutely necessary
and eliminate unnecessary portions of the graph fro m consideration
- connect the same thingd(nodes) in differnt ways(differently named relationships)
Specialize the grap for different use case










