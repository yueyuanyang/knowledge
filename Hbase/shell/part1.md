## HBase Shell 的常用操作总结

###  hbase shell 脚本

> hbase shell test.hbaseshell

### 查

1. 查看所有的表
```
list
```

2. 查看表的结构
```
descibe 't1'
```

3. 查看某一表是否存在
```
exist 't1'
```

4. 查看表结构是否允许修改
``` 
enable_is 't1'
```

5. 查询某个行健的所有列族的列值
```
get 't1','r1'
```

6. 查询某个行健的所有某个列族的列值

```
get 't1','r1','f1'
```

7. 获取某个行健的某两个列族的列值
```
get 't1', 'r1','f1','f2'
```

8. 获取某个行健的某个列族的某个列值
```
get 't1', 'r1', 'f1:c1'
```

9. 获取某个表的所有行健值
```
scan 't1'
```

10. 获取某个表的前3行
```
scan 't1',{LIMIT=>3}
```

11. 获取某个表的从指定位置开始的行

```
scan 't1',{STARTROW=>'rowKey',LIMIT=>10}
```

12. 获取某个表的指定列的所有行数据

```
scan 'heroes',{COLUMNS =>'f1:c1'}
```

13. 统计表的行数
```
count ‘t1′  
count ‘t1′, INTERVAL => 100000  //统计的行数间隔,默认为1000
count ‘t1′, CACHE => 1000  //CACHE为统计的数据缓存
count ‘t1′, INTERVAL => 10, CACHE => 1000  //这种方式效率很低
```

### 增

1. 创建表
```
create 't1','f1','f2','f3' //t1是表名，f1，f2，f3是列族名
```
2. 向表中插入数据
```

给t1表的r1行键的f1列族的c1列插入一个值24，列族的列事先可以不存在，
修改数据也是put，只需行健和列相同即可

put 't1','r1','c1','value'

```

### 删

1. 删除某个列族
``` 
disable 't1'
alter 't1',NAME=>'f1',METHOD=>'delete' //--注意大小写（简写：alter 't1', 'delete'=>'f1'）
enable 't1'
```
  
2. 删除某张表
```
disable 't1'
drop 't1'
```

3. 删除某行数据的列[值]
```
删除t1表，行健为r1的c1列中，时间戳为ts1的值，如果不指定ts1就删除所有列值，
显然该行的该列也不复存在。

delete 't1', 'r1', 'c1', ts1

```

4. 删除某行数据

```
deleteall 't1', r1'
```

5. 清空表

```
truncate 't1'

类似于：
Disabling  't1'
Dropping  't1'
Creating  't1'
```

### 改

1. 增加一个列族
```
disable 't1'
alter 't1',NAME=>'f1',VERSIONS=>3
enable 't1' 
```









