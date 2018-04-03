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
