## 线上MYSQL同步报错故障处理总结
### 前言

在发生故障切换后，经常遇到的问题就是同步报错，数据库很小的时候，dump完再导入很简单就处理好了，但线上的数据库都150G-200G，如果用单纯的这种方法，成本太高，故经过一段时间的摸索，总结了几种处理方法。

### 生产环境架构图

目前现网的架构，保存着两份数据，通过异步复制做的高可用集群，两台机器提供对外服务。在发生故障时，切换到slave上，并将其变成master，坏掉的机器反向同步新的master，在处理故障时，遇到最多的就是主从报错。下面是我收录下来的报错信息

### 常见错误

**最常见的3种情况**

这3种情况是在HA切换时，由于是异步复制，且sync_binlog=0，会造成一小部分binlog没接收完导致同步报错。

- 第一种：在master上删除一条记录，而slave上找不到。
```
Last_SQL_Error: Could not execute Delete_rows event on table hcy.t1; 
Can't find record in 't1', 
Error_code: 1032;`handler error HA_ERR_KEY_NOT_FOUND`; 
the event's master log mysql-bin.000006, end_log_pos 254
```

- 第二种：主键重复。在slave已经有该记录，又在master上插入了同一条记录。
```
Last_SQL_Error: Could not execute Write_rows event on table hcy.t1; 
Duplicate entry '2' for key 'PRIMARY', 
Error_code: 1062; 
handler error HA_ERR_FOUND_DUPP_KEY; the event's master log mysql-bin.000006, end_log_pos 924

```
- 第三种：在master上更新一条记录，而slave上找不到，丢失了数据。
```
Last_SQL_Error: Could not execute Update_rows event on table hcy.t1;
Can't find record in 't1', 
Error_code: 1032; 
handler error HA_ERR_KEY_NOT_FOUND; the event's master log mysql-bin.000010, end_log_pos 263
```



