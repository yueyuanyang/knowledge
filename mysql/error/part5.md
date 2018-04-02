## mysql 常用命令

**主从复制状态*

> show slave status\G   \G 格式化

**当前用户运行的进程**

>  show processlist;

**表中查看当前未提交的事务**

```
select trx_state, trx_started, trx_mysql_thread_id, trx_query 
from information_schema.innodb_trx\G
```



