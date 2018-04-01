### neo4j 操作 - DELETE REMOVE

### 删除 —— DELETE

**删除节点子句语法**

DELETE <node-name-list>

**实例**

```
MATCH (e: Employee) DELETE e
```

**删除节点和关系子句语法**

DELETE <node1-name>,<node2-name>,<relationship-name>

**实例**

```
MATCH (cc: CreditCard)-[rel]-(c:Customer) 
DELETE cc,c,rel

```

**删除属性子句语法**

**删除标签子句语法：**

REMOVE <label-name-list>

### 修改属性

**SET子句语法**

SET  <property-name-list>

**实例**
```
MATCH (dc:DebitCard)
SET dc.atm_pin = 3456
RETURN dc
```

### 排序 —— order by 

**ORDER BY子句语法**

ORDER BY  <property-name-list>  [DESC]	

**实例**

```
MATCH (emp:Employee)
RETURN emp.empid,emp.name,emp.salary,emp.deptno
ORDER BY emp.name
```

### 连接 —— UNION

**UNION子句 —— 交集**

它结合并返回来自两个组结果共同行到单个组结果。 它不会从两个节点返回重复的行。

**限制：** 

结果列类型，并从两个结果集的名字必须匹配，这意味着列名称应该是相同的，列的数据类型应该是相同的

**UNION子句语法**

<MATCH Command1>
   UNION
<MATCH Command2>

**实例**

```
1.
MATCH (cc:CreditCard) RETURN cc.id,cc.number
UNION
MATCH (dc:DebitCard) RETURN dc.id,dc.number

2.
MATCH (cc:CreditCard)
RETURN cc.id as id,cc.number as number,cc.name as name,
   cc.valid_from as valid_from,cc.valid_to as valid_to
UNION
MATCH (dc:DebitCard)
RETURN dc.id as id,dc.number as number,dc.name as name,
   dc.valid_from as valid_from,dc.valid_to as valid_to
   
```

**UNION ALL子句 - 并集**

它结合并返回两个结果集的所有行成一个单一的结果集。 它还返回由两个节点重复行。

**限制**

结果列类型，并从两个结果集的名字必须匹配，这意味着列名称应该是相同的，列的数据类型应该是相同的。

UNION ALL子句语法
<MATCH Command1>
UNION ALL
<MATCH Command2>

**实例**

```
MATCH (cc:CreditCard)
RETURN cc.id as id,cc.number as number,cc.name as name,
   cc.valid_from as valid_from,cc.valid_to as valid_to
UNION ALL
MATCH (dc:DebitCard)
RETURN dc.id as id,dc.number as number,dc.name as name,
   dc.valid_from as valid_from,dc.valid_to as valid_to
```

### LIMIT 和 跳过 SKIP

L**IMIT子句**

行由查询返回的Neo4j CQL提供了“限制”条款进行过滤或限制数量。 它修剪从CQL查询结果集底部的结果。

**LIMIT子句语法**

LIMIT <number>
  
**实例**
```
MATCH (emp:Employee) 
RETURN emp
LIMIT 2

```

**SKIP跳过**

Neo4j的CQL提供了“跳过”条款进行过滤或限制的行数由查询返回

SKIP <number>
  
**实例**
```
MATCH (emp:Employee) 
RETURN emp
SKIP 2

我们所定义的 SKIP= 2，这意味着最后两行，它返回从底部只有两个结果

```

### 合并 —— MERGE

Neo4j的CQL MERGE使用命令 -

- 要创建节点，关系和属性

- 为了从数据库中检索数据

MERGE命令创建命令和 MATCH 命令的组合。

MERGE = CREATE + MATCH

Neo4j的CQL MERGE 图中的给定模式命令搜索，如果存在则返回结果

如果它不在图中存在，则它创建新的节点/关系并返回结果

MERGE (<node-name>:<label-name>
{
   <Property1-name>:<Pro<rty1-Value>
   .....
   <Propertyn-name>:<Propertyn-Value>
})

**实例**

```
MERGE (gp2:GoogleProfile2{ Id: 201402,Name:"Nokia"})
```




















