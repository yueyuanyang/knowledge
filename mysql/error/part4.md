## 主从复制延迟分析 —— SQL线程一直为‘system lock’状态

**故障表现：（SQL线程一直为‘system lock’状态）**

```
   SQL_Remaining_Delay: NULL
   Slave_SQL_Running_State: system lock
   Master_Retry_Count: 86400

```

**故障原因**：由于所更新表没有设置合适的索引，导致每次操作都是全表（binlog格式为ROW）。。数据量越大，在update/delete等操作时,会越慢。。最后导致主从延迟增大。由于本人文字描述水品较差，后文会把浮现的过程写出来，让大家明白具体的原因在那里。

**故障解决**：

-  只要添加合适的索引即可
-  slave_rows_search_algorithms

```
执行方法：
set global slave_rows_search_algorithms='TABLE_SCAN,INDEX_SCAN,HASH_SCAN';

```

使用mariadb，但碰到客户存在无索引或索引非常烂的表时，经常会因为一个大的delete或update导致巨大的主从延迟。想到`mysql5.6`的时候号称通过引入 hash_scan 算法解决了这个问题，

### system lock 延迟的原因

这里直接给出原因供大家直接参考:
必要条件：
由于大量的小事物如UPDATE/DELETE table where一行数据，这种只包含一行DML event的语句，table是一张大表。

这个表上没有主键或者唯一键。
由于类似innodb lock堵塞，也就是slave从库修改了数据同时和sql_thread也在修改同样的数据。
确实I/O扛不住了，可以尝试修改参数。
如果是大量的表没有主键或者唯一键可以考虑修改参数slave_rows_search_algorithms 试试。但是innodb中不用主键或者主键不选择好就等于自杀。



**最后建议：**
在设计表结构时，尽量要设置PK，以减少同一张表数据的冗余情况存在。如果必须，则要对语句进行适当的优化。
