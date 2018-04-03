## 查看mysql主从配置的状态及修正 slave不启动问题

## 1、查看master的状态
```
show master status;  //Position不应该为0
show processlist;  
//state状态应该为Has sent all binlog to slave; waiting for binlog to be updated
```
### 2、查看slave状态
```
show slave status;
//Slave_IO_Running 与 Slave_SQL_Running 状态都要为Yes
show processlist;
//应该有两行state值为：
Has read all relay log; waiting for the slave I/O thread to update it
Waiting for master to send event
```

### 3、错误日志
```
MySQL安装目录 /usr/local/mysql
MySQL日志目录 /usr/local/mysql/data/   形如，Hostname.err
```

### 4、Change master to
如果从库的Slave未启动，Slave_IO_Running为NO。
可能是主库是的master的信息有变化，
查看主库show master status;
记录下File,Position字段，假设为‘mysql-bin.000004’,98;
在从库执行：
 ```
 mysql>stop slave;
 mysql>change master to master_log_file='mysql-bin.000004',master_log_pos=98;
 mysql>start slave;
```

### 5、SET global sql_slave_skip_counter=n;

如果从库的slave_sql_running为NO。
Err文件中记录：
Slave:Error "Duplicate entry '1' for key 1" on query.....
可能是master未向slave同步成功，但slave中已经有了记录。造成的冲突可以在从库上执行
set global sql_slave_skip_counter=n;
跳过几步。再restart slave就可以了。

### 6、同步错误处理
发现mysql slave服务器经常因为一些特殊字符或者符号产生的更新语句报错，整个同步也会因此而卡在那，最初的办法只是手动去出错的机器执行下面三条SQL语句，跳过错误即可。
 ```
 mysql>stop slave;
 mysql>set GLOBAL SQL_SLAVE_SKIP_COUNTER=1;
 mysql>start slave;
```

PS：本人多次遇到从数据库的同步进程自动停掉的问题，有时简单通过slave stop,slave start即可解决。有时slave start启动后又会自动停掉，这时使用 change master重设主数据库信息的方式解决了问题
