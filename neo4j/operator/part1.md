## neo4j 基本操作语句汇总

## 创建节点 —— CREATE命令

**Neo4j的CQL创建没有性质的节点**

**CREATE命令语法**

CREATE (<node-name>:<label-name>)

语法说明：

| 语法元素 | 描述 
| - | :-: | 
| 创建 | 	这是一个Neo4j的定制列表命令 | 
| <节点名称> | 这是我们要创建一个节点名称 |
| <标签名称> | 这是一个节点的标签名称 |

**实例：**
```
CREATE (emp:Employee)

这里是EMP节点名称
员工是EMP节点的标签名称

```

**Neo4j的CQL创建节点具有属性**

Neo4j的CQL“Create”命令用于创建具有属性节点。 它创建了一些属性（键 - 值对）来存储数据的节点。

**CREATE命令语法：**

CREATE (
   <node-name>:<label-name>
   { 	
      <Property1-name>:<Property1-Value>
      ........
      <Propertyn-name>:<Propertyn-Value>
   }
)

| 语法元素 | 描述 
| - | :-: | 
| <节点名称> | 这是我们要创建一个节点名称 |
| <标签名称> | 这是一个节点的标签名称 |
| <Property1名> ... <propertyN中名称> | 	属性是键 - 值对。 定义了将要分配给创建节点的属性的名称 |
| <Property1值> ... <propertyN中值> | 属性是键 - 值对。 定义了将要分配给一个创建节点的属性的值 |

**实例：**
```
CREATE (dept:Dept { deptno:10,dname:"Accounting",location:"Hyderabad" })

如果你观察成功的消息，它告诉我们

1) 一个标签，即创建“部”
2) 创建一个节点，即“部门”
3) 三个属性是创建即DEPTNO，DNAME，位置

```

## 查询节点 —— match命令

Neo4j的CQL MATCH命令用于

- 要获取有关数据库节点和属性数据
- 要获得有关节点，从数据库中的关系和属性数据

MATCH命令语法：

MATCH 
(
   <node-name>:<label-name>
)

| 语法元素 | 描述 
| - | :-: | 
| <节点名称> | 这是我们要创建一个节点名称 |
| <标签名称> | 这是一个节点的标签名称 |












