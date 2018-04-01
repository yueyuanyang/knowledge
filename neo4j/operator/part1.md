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

Neo4j的 CQL MATCH命令用于

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

**实例**

```
MATCH (dept:Dept)
```

### Neo4j CQL - return子句

eo4j的CQL RETURN子句用于 

- 要检索节点的一些特性
- 要检索节点的所有属性
- 要检索节点和关联关系的一些性质
- 要检索节点和关联关系的所有属性

**返回的命令语法：**

RETURN 
   <node-name>.<property1-name>,
   ........
   <node-name>.<propertyn-name>

**语法说明**

| 语法元素 | 描述 
| - | :-: | 
| <节点名称> | 这是我们要创建一个节点名称 |
| <Property1名> ... <propertyN中名称> | 属性是键 - 值对。 <属性名称>定义了将要分配给创建节点的属性的名称 |

**实例：**

```
RETURN dept.deptno

部门是一个节点名
DEPTNO是部门节点的属性名称

```
### Neo4j CQL - MATCH及RETURN

在Neo4j的CQL，我们不能用MATCh或RETURN单独命令，所以我们要结合这两个命令来检索数据库中的数据。

Neo4j的CQL MATCH + RETURN命令用于 -

- 要检索节点的一些特性
- 要检索节点的所有属性
- 要检索节点和关联关系的一些性质
- 要检索节点和关联关系的所有属性

**匹配返回命令语法：**

MATCH Command
RETURN Command

| 语法元素 | 描述 
| - | :-: | 
| match命令 | 这是Neo4j的CQL MATCH命令 |
| RETURN指令 | 这是Neo4j的CQL RETURN命令 |

**实例**
```
MATCH (dept: Dept)
RETURN dept.deptno,dept.dname
```
这里 -

- 部门是一个节点名
- 这里是部节点标签名称
- DEPTNO是部门节点的属性名称
- DNAME是部门节点的属性名称




















