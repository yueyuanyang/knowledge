## 记一次MySQL中Waiting for table metadata lock的解决方法

在网上查询得知MySQL在进行一些alter table等DDL操作时，如果该表上有未提交的事务则会出现 `Waiting for table metadata lock` , 而一旦出现metadata lock，该表上的后续操作都会被阻塞

所以这个问题需从两方面解决：

### 1. 从 information_schema.innodb_trx 表中查看当前未提交的事务:

```
select trx_state, trx_started, trx_mysql_thread_id, trx_query 
from information_schema.innodb_trx\G

```

\G作为结束符时，MySQL Client会把结果以列模式展示，对于列比较长的表，展示更直观）

**字段意义：**

- trx_state: 事务状态，一般为RUNNING
- trx_started: 事务执行的起始时间，若时间较长，则要分析该事务是否合理
- trx_mysql_thread_id: MySQL的线程ID，用于kill
- trx_query: 事务中的sql

一般只要kill掉这些线程，DDL操作就不会Waiting for table metadata lock。

### 2. 调整锁超时阈值

lock_wait_timeout 表示获取metadata lock的超时（单位为秒），允许的值范围为1到31536000（1年）。 默认值为31536000。详见 https://dev.mysql.com/doc/refman/5.6/en/server-system-variables.html#sysvar_lock_wait_timeout 。默认值为一年！！！已哭瞎！将其调整为30分钟
```
set session lock_wait_timeout = 1800;
set global lock_wait_timeout = 1800;
```
好让出现该问题时快速故障（failfast）
