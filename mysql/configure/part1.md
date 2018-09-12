## mysql 数据库配置(1)

## mysql 5.7 发现产生日志的时间和当期的系统时间不一致
```
// 查看时间格式
show globle variables like ‘log_timestamps’; 
```
修改
```
暂时修改
set globle log_timestamps=SYSTEM
永久设置
#vim /etc/my.cnf
[mysqld]
log_timestamps=SYSTEM
```

### 查看mysql的运行时长
```
mysql> show global status like 'uptime';
+---------------+---------+
| Variable_name | Value   |
+---------------+---------+
| Uptime        | 3414707 |
+---------------+---------+
```

### 查看时间参数
```
mysql> show global variables like '%timeout';
+----------------------------+----------+
| Variable_name              | Value    |
+----------------------------+----------+
| connect_timeout            | 10       |
| delayed_insert_timeout     | 300      |
| innodb_lock_wait_timeout   | 50       |
| innodb_rollback_on_timeout | OFF      |
| interactive_timeout        | 28800    |
| lock_wait_timeout          | 31536000 |
| net_read_timeout           | 30       |
| net_write_timeout          | 60       |
| slave_net_timeout          | 3600     |
| wait_timeout               | 28800    |
+----------------------------+----------+
```
### mysql请求链接进程被主动kill
```
mysql> show global status like 'com_kill';
+---------------+-------+
| Variable_name | Value |
+---------------+-------+
| Com_kill      | 21    |
+---------------+-------+
```

### 查看文件大小是否超过 max_allowed_packet ，如果超过则需要调整参数，或者优化语句。

```
mysql> show global variables like 'max_allowed_packet';
+--------------------+---------+
| Variable_name      | Value   |
+--------------------+---------+
| max_allowed_packet | 1048576 |
+--------------------+---------+
```
