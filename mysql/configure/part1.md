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
