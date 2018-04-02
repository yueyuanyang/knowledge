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

**异步半同步区别**

- 异步复制
简单的说就是master把binlog发送过去，不管slave是否接收完，也不管是否执行完，这一动作就结束了.

- 半同步复制
简单的说就是master把binlog发送过去，slave确认接收完，但不管它是否执行完，给master一个信号我这边收到了，这一动作就结束了。（谷歌写的代码，5.5上正式应用。）

- 异步的劣势
当master上写操作繁忙时，当前POS点例如是10，而slave上IO_THREAD线程接收过来的是3，此时master宕机，会造成相差7个点未传送到slave上而数据丢失。


**特殊的情况**

- slave的中继日志relay-bin损坏。
```
Last_SQL_Error: Error initializing relay log position: I/O error reading the header from the binary log
Last_SQL_Error: Error initializing relay log position: Binlog has bad magic number; 
It's not a binary log file that can be used by this version of MySQL
```

这种情况SLAVE在宕机，或者非法关机，例如电源故障、主板烧了等，造成中继日志损坏，同步停掉。

- 人为失误需谨慎：多台slave存在重复server-id

这种情况同步会一直延时，永远也同步不完，error错误日志里一直出现上面两行信息。解决方法就是把server-id改成不一致即可。

```
Slave: received end packet from server, apparent master shutdown:
Slave I/O thread: Failed reading log event, reconnecting to retry, log 'mysql-bin.000012' at postion 106
```

### 问题处理

**删除失败**

在master上删除一条记录，而slave上找不到。
```
Last_SQL_Error: Could not execute Delete_rows event on table hcy.t1; 
Can't find record in 't1',
Error_code: 1032; handler error HA_ERR_KEY_NOT_FOUND; 
the event's master log mysql-bin.000006, end_log_pos 254
```

**解决方法：**

由于master要删除一条记录，而slave上找不到故报错，这种情况主上都将其删除了，那么从机可以直接跳过。可用命令：
```
stop slave;
set global sql_slave_skip_counter=1;
start slave;
```
如果这种情况很多，可用我写的一个脚本skip_error_replcation.sh，默认跳过10个错误（只针对这种情况才跳，其他情况输出错误结果，等待处理），这个脚本是参考maakit工具包的mk-slave-restart原理用shell写的，功能上定义了一些自己的东西，不是无论什么错误都一律跳过。）

**主键重复**

在slave已经有该记录，又在master上插入了同一条记录。

```
Last_SQL_Error: Could not execute Write_rows event on table hcy.t1; 
Duplicate entry '2' for key 'PRIMARY', 
Error_code: 1062; 
handler error HA_ERR_FOUND_DUPP_KEY; the event's master log mysql-bin.000006, end_log_pos 924
```

**解决方法：**

在slave上用desc hcy.t1; 先看下表结构：
```
mysql> desc hcy.t1;
+-------+---------+------+-----+---------+-------+
| Field | Type    | Null | Key | Default | Extra |
+-------+---------+------+-----+---------+-------+
| id    | int(11) | NO   | PRI | 0       |       | 
| name  | char(4) | YES  |     | NULL    |       | 
+-------+---------+------+-----+---------+-------+
```
删除重复的主键
```
mysql> delete from t1 where id=2;
Query OK, 1 row affected (0.00 sec)

mysql> start slave;
Query OK, 0 rows affected (0.00 sec)

mysql> show slave status\G;
……
Slave_IO_Running: Yes
Slave_SQL_Running: Yes
……
mysql> select * from t1 where id=2;
在master上和slave上再分别确认一下。
```

**更新丢失**

在master上更新一条记录，而slave上找不到，丢失了数据。

```
Last_SQL_Error: Could not execute Update_rows event on table hcy.t1; 
Can't find record in 't1', 
Error_code: 1032; 
handler error HA_ERR_KEY_NOT_FOUND; 
the event's master log mysql-bin.000010, end_log_pos 794
```
**解决方法：**

在master上，用mysqlbinlog 分析下出错的binlog日志在干什么。
```
/usr/local/mysql/bin/mysqlbinlog --no-defaults -v -v --base64-output=DECODE-ROWS mysql-bin.000010 | grep -A '10' 794

#120302 12:08:36 server id 22  end_log_pos 794  Update_rows: table id 33 flags: STMT_END_F
### UPDATE hcy.t1
### WHERE
###   @1=2 /* INT meta=0 nullable=0 is_null=0 */
###   @2='bbc' /* STRING(4) meta=65028 nullable=1 is_null=0 */
### SET
###   @1=2 /* INT meta=0 nullable=0 is_null=0 */
###   @2='BTV' /* STRING(4) meta=65028 nullable=1 is_null=0 */
# at 794
#120302 12:08:36 server id 22  end_log_pos 821  Xid = 60
COMMIT/*!*/;
DELIMITER ;
# End of log file
ROLLBACK /* added by mysqlbinlog */;
/*!50003 SET COMPLETION_TYPE=@OLD_COMPLETION_TYPE*/;
```

在slave上，查找下更新后的那条记录，应该是不存在的。

```
mysql> select * from t1 where id=2;
Empty set (0.00 sec)
```
然后再到master查看
```
mysql> select * from t1 where id=2;
+----+------+
| id | name |
+----+------+
|  2 | BTV  | 
+----+------+
1 row in set (0.00 sec)
```
把丢失的数据在slave上填补，然后跳过报错即可。
```
mysql> insert into t1 values (2,'BTV');
Query OK, 1 row affected (0.00 sec)

mysql> select * from t1 where id=2;    
+----+------+
| id | name |
+----+------+
|  2 | BTV  | 
+----+------+
1 row in set (0.00 sec)

mysql> stop slave ;set global sql_slave_skip_counter=1;start slave;
Query OK, 0 rows affected (0.01 sec)
Query OK, 0 rows affected (0.00 sec)
Query OK, 0 rows affected (0.00 sec)

mysql> show slave status\G;
……
 Slave_IO_Running: Yes
 Slave_SQL_Running: Yes
……
```

**中继日志损坏**

中继日志损坏

```
Last_SQL_Error: Error initializing relay log position: I/O error reading the header from the binary log
Last_SQL_Error: Error initializing relay log position: Binlog has bad magic number;  
It's not a binary log file that can be used by  this version of MySQL
```

**手工修复**

**解决方法：**找到同步的binlog和POS点，然后重新做同步，这样就可以有新的中继日值了。

例子：
```
mysql> show slave status\G;
*************************** 1. row ***************************
              Master_Log_File: mysql-bin.000010
          Read_Master_Log_Pos: 1191
               Relay_Log_File: vm02-relay-bin.000005
                Relay_Log_Pos: 253
        Relay_Master_Log_File: mysql-bin.000010
             Slave_IO_Running: Yes
            Slave_SQL_Running: No
              Replicate_Do_DB: 
          Replicate_Ignore_DB: 
           Replicate_Do_Table: 
       Replicate_Ignore_Table: 
      Replicate_Wild_Do_Table: 
  Replicate_Wild_Ignore_Table: 
                   Last_Errno: 1593
                   Last_Error: Error initializing relay log position: I/O error reading the header from the binary log
                 Skip_Counter: 1
          Exec_Master_Log_Pos: 821

Slave_IO_Running ：接收master的binlog信息
                   Master_Log_File
                   Read_Master_Log_Pos

Slave_SQL_Running：执行写操作
                   Relay_Master_Log_File
                   Exec_Master_Log_Pos
```
以执行写的binlog和POS点为准。
```
Relay_Master_Log_File: mysql-bin.000010
Exec_Master_Log_Pos: 821
mysql> stop slave;
Query OK, 0 rows affected (0.01 sec)

mysql> CHANGE MASTER TO MASTER_LOG_FILE='mysql-bin.000010',MASTER_LOG_POS=821;
Query OK, 0 rows affected (0.01 sec)

mysql> start slave;
Query OK, 0 rows affected (0.00 sec)


mysql> show slave status\G;
*************************** 1. row ***************************
               Slave_IO_State: Waiting for master to send event
                  Master_Host: 192.168.8.22
                  Master_User: repl
                  Master_Port: 3306
                Connect_Retry: 10
              Master_Log_File: mysql-bin.000010
          Read_Master_Log_Pos: 1191
               Relay_Log_File: vm02-relay-bin.000002
                Relay_Log_Pos: 623
        Relay_Master_Log_File: mysql-bin.000010
             Slave_IO_Running: Yes
            Slave_SQL_Running: Yes
              Replicate_Do_DB: 
          Replicate_Ignore_DB: 
           Replicate_Do_Table: 
       Replicate_Ignore_Table: 
      Replicate_Wild_Do_Table: 
  Replicate_Wild_Ignore_Table: 
                   Last_Errno: 0
                   Last_Error: 
                 Skip_Counter: 0
          Exec_Master_Log_Pos: 1191
              Relay_Log_Space: 778
              Until_Condition: None
               Until_Log_File: 
                Until_Log_Pos: 0
           Master_SSL_Allowed: No
           Master_SSL_CA_File: 
           Master_SSL_CA_Path: 
              Master_SSL_Cert: 
            Master_SSL_Cipher: 
               Master_SSL_Key: 
        Seconds_Behind_Master: 0
Master_SSL_Verify_Server_Cert: No
                Last_IO_Errno: 0
                Last_IO_Error: 
               Last_SQL_Errno: 0
               Last_SQL_Error: 
               
```






