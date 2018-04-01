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








