## HBase Shell 的常用操作总结

1. 创建表
> create 't1','f1','f2','f3' //t1是表名，f1，f2，f3是列族名

2. 查看所有的表
> list

3. 查看表的结构
> descibe 't1'

4. 增加一个列族
> disable 't1'
  alter 't1',NAME=>'f1',VERSIONS=>3
  enable 't1' 

5. 删除某个列族
> disable 't1'
  alter 't1',NAME=>'f1',METHOD=>'delete' //--注意大小写（简写：alter 't1', 'delete'=>'f1'）
  enable 't1'

6. 查看某一表是否存在
> exist 't1'

7. 查看表结构是否允许修改
> enable_is 't1'

8. 删除某张表
> disable 't1'
  drop 't1'

9. 向表中插入数据
> put 't1','r1','c1','value' // 给t1表的r1行键的f1列族的c1列插入一个值24，列族的列事先可以不存在，修改数据也是put，只需行健和列相同即可
