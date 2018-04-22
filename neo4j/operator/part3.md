### Cypher 实战
Cypher 句法有四个不同的部分组成
- start —— 查找图形的起始节点
- match —— 匹配图形模式，可以定位兴趣数据的子图形
- where —— 基于某些标准过滤数据
- return —— 返回感兴趣的结果

### 模式匹配

1. 单个命名的关系连接的两个节点的模糊匹配

```
start user=node(1)
match (user)-[:HAS_SEEN]->(movie)
return movie
```

2.如果不在意关系的方向，或者不清楚哪一个是关系的起始节点或终止节点，则可以两端都使用单个连字符连接在，这种情况下解读为“任意”。如果想知道两个用户是否是朋友，忽略掉方向，可以使用下面的模
式。

```
start user=node(1)
match (user)-[:HAS_SEEN]-(movie)
return movie
```

3. 使用节点和关系标示

在Cypher查询中，节点和关系都可以与标识关联，这种关联使得以后可以在同样的查询中引用同一个图形实体

关系也可以命名，使用稍微不同的语法：
```
start user=node(1)
match (user)-[r:HAS_SEEN]-(movie)
return movie
```

### 复杂模式匹配

#### 复杂模式匹配

1. 查找用户1的朋友看过的所有电影

```
start john=node(1)
match john-[:IS_FRIEND_OF]->()-[:HAS_SEEN]->(movie)
return movie
```
2. 查找用户1的朋友看过的所有电影,用户1已经看过的电影

```
start john=node(1)
match john-[:IS_FRIEND_OF]->()-[:HAS_SEEN]->(mover),
john-[r:HAS_SEEN]->(movie)
return movie
```
作为结果的节点必须匹配所有逗号隔开的模式，相当于一个“并”（AND）语句。

3. 查找用户1的朋友看过的所有电影,John没有看过的电影
```
start john=node(1)
match john-[:IS_FRIEND_OF]->(user)-[:HAS_SEEN]->(mover),
where NOT john-[r:HAS_SEEN]->(movie)
return movie
```

### 查找起始节点

#### 查找起始节点

1. 通过编号查找节点

通过直接编号查找加载起始节点

```
start john=node(1)
return john
```

2. 通过多个节点编号加载多个节点

在start语句中使用多个编号，需要列出以逗号隔开的编号参数值

```
start user=node(1,3)
match user-[:HAS_SEEN]->movie
return distinct movie
```

3. 使用**索引**查找起始节点

在Cypher查询的start语句中做Lucene索引查找，而不是通过编号加载节点

```
start john=node:users(name="John Johnson")
return john
```
node：users部分指定了一个感兴趣的来自users索引的节点。当给索引添加节点时，索引的名字必须匹配使用的索引名字。圆括号中的参数指定了在索引中查找的键——值对；在这种情况下，匹配的是name属性与值“John Johnson”

这种索引查找句法需要正好键-值对的匹配，并且是大小写敏感的，因此具有name属性“john johnson”或者“John Johnso”将不会返回。同时，如果多个节点以同一键-值对（如果在数据库中有多于一个的John Johnson）索引，所有匹配的节点都要加载；

4. 如果使用Lucene查询的完全功能查找起始节点，可以使用与**本地Lucene原始查询**稍微不同的句法

```
start john=node:users("name:John Johnson")
return john
```
通配符或多属性匹配

```
start john=node:users("name:John* AND yearOfBirth<1980")
return john
```
### 使用基于模式的索引查找起始节点

1. 使用标签通过名字查找用户节点

```
match (john:USER)
where john.name='john johnson'
return john
```

























