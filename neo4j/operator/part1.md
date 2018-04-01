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
### 多个标签节点

**句法**

CREATE (<node-name>:<label-name1>:<label-name2>.....:<label-namen>)

| 语法元素 | 描述 
| - | :-: | 
| 创建 | 这是一个Neo4j的CQL关键字
| <节点名称> | 这是我们要创建一个节点名称 |
| <标签名1>，<标签名2> | 它是一个节点的标签名称的列表 |

**实例**

```
此示例演示如何创建多个标签名称为“电影”节点。

影院，影片，电影，图片：由我们的客户端提供多标签名称

CREATE (m:Movie:Cinema:Film:Picture)

```

### 单个标签有关系

**句法**

CREATE (<node1-name>:<label1-name>)- [(<relationship-name>:<relationship-label-name>)]
	->(<node2-name>:<label2-name>)
   
**语法说明**

| 语法元素 | 描述 
| - | :-: | 
| 创建 | 这是一个Neo4j的CQL关键字
| <节点1名> | 这是一个从节点的名称 |
| <节点2名>| 这是A到节点的名称 |
| <LABEL1名称>| 这是一个从节点的标签名称 |
| <LABEL1名称> | 这是一个到节点的标签名称。 |
| <关系名称>| 它是一个关系的一个名字 |
| 	<相关标签名称> | 这是一个关系的标签名称 |

```
CREATE (p1:Profile1)-[r1:LIKES]->(p2:Profile2)
在这里，p1和Profile1的都是“从节点”的节点名称和节点标签名称

P2和Profile2的都是的“节点”节点名称和节点标签名称

R1是有关系的名字

喜欢是有关系的标签名称

```

### 简单的WHERE子句语法

WHERE <condition>
 
**复杂的WHERE子句语法**

WHERE <condition> <boolean-operator> <condition>
   
我们可以通过使用布尔运算符将多个条件对同样的命令。

**<条件>的语法：**

<property-name> <comparison-operator> <value>

| 语法元素 | 描述 
| - | :-: | 
| 哪里 | 这是一个Neo4j的CQL关键字|
| <属性名称> | 这是一个节点或关系的属性名 |
| <比较运算符>| 这是Neo4j的CQL比较operators.Please的一个是指在Neo4j的CQL可用比较运营商 |
| <值>| 这就像一些文字，字符串文字等文本值 |

**布尔运算符中的Neo4j CQL**

Neo4j的支持以下布尔运算符中的Neo4j CQL使用WHERE子句来支持多个条件。

| 语法元素 | 描述 
| - | :-: | 
| and | 这是一个Neo4j的CQL关键字支持与操作。 它就像SQL和运营商 |
| OR | 	这是一个Neo4j的CQL关键字来支持或操作。 它就像SQL和运营商 |
| NOT| 这是一个Neo4j的CQL关键字方式支持非操作。 它就像SQL和运营商 |
| XOR| 它是一个的Neo4j CQL关键词来支持XOR操作。 它就像SQL和运营商 |

**比较运营商的Neo4j CQL**

Neo4j的支持以下比较运营商的Neo4j CQL使用WHERE子句来支持条件。

| 语法元素 | 描述 
| - | :-: | 
| = | 这是一个Neo4j的定制列表“等于”操作符 |
| <> | 这是一个Neo4j的定制列表“不等于”操作符 |
| < | 这是一个Neo4j的定制列表“不等于”操作符 |
| > | 这是一个Neo4j的定制列表“大于”操作符 |
| <= | 	这是一个Neo4j的定制列表“小于或等于”操作符 |
| >= | 这是一个Neo4j的定制列表“大于或等于”操作符 |

**实例**
```
MATCH (emp:Employee) 
WHERE emp.name = 'Abc'
RETURN emp

MATCH (emp:Employee) 
WHERE emp.name = 'Abc' OR emp.name = 'Xyz'
RETURN emp

```

**创建WHERE子句的关系**

在Neo4j的CQL，我们可以创建不同的方法拖车节点之间的关系。

- 创建两个现有的节点之间的关系

- 一次创建它们之间的两个节点和关系

- 创建WHERE子句两个已存在节点之间的关系

**句法**

MATCH (<node1-label-name>:<node1-name>),(<node2-label-name>:<node2-name>) 
WHERE <condition>
CREATE (<node1-label-name>)-[<relationship-label-name>:<relationship-name>
       {<relationship-properties>}]->(<node2-label-name>) 
	
```
输入在数据浏览器下面的命令来创建客户和信用卡式节点之间的关系。

MATCH (cust:Customer),(cc:CreditCard) 
WHERE cust.id = "1001" AND cc.id= "5001" 
CREATE (cust)-[r:DO_SHOPPING_WITH{shopdate:"12/12/2014",price:55000}]->(cc) 
RETURN r

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




















