## Hive -hivevar 参数传递

命令行模式，或者说目录模式，可以使用hive 执行命令。

**选项说明：**

-e : 执行短命令
-f :  执行文件（适合脚本封装）
-S : 安静模式，不显示MR的运行过程
-hivevar : 传参数 ，专门提供给用户自定义变量。
-hiveconf : 传参数，包括了hive-site.xml中配置的hive全局变量。

**例子1**：hive -e 查询

```
命令: hive -S -e "use default; select * from kimbo_test limit 3;"

``

**例子2**：hive -f 执行文件

```
命令: hive -S -f test_k.hql          -- 返回3条记录
```

**例子3**：hive -f 参数传递，执行文件

```
命令: hive -hivevar v_date='20170630' -S -f test_par.hql    -- 返回3条记录

命令: hive -hivevar v_date='20170101' -S -f test_par.hql    -- 返回0条记录

```

**查看文件内容：**

```
　　cat test_par.hql

　　　　use default; select * from kimbo_test where dt='${hivevar:v_date}' limit 3;

　　cat test_k.hql 

　　　　use default; select * from kimbo_test limit 3;
    
```
