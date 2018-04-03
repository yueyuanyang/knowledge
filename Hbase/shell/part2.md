## HBase filter shell操作

### 1. 创建表

```
create 'test1', 'lf', 'sf'

lf: column family of LONG values (binary value)
-- sf: column family of STRING values
```

### 2. 导入数据

```
put 'test1', 'user1|ts1', 'sf:c1', 'sku1'
put 'test1', 'user1|ts2', 'sf:c1', 'sku188'
put 'test1', 'user1|ts3', 'sf:s1', 'sku123'

put 'test1', 'user2|ts4', 'sf:c1', 'sku2'
put 'test1', 'user2|ts5', 'sf:c2', 'sku288'
put 'test1', 'user2|ts6', 'sf:s1', 'sku222'

rowkey(user2|ts4)  // 用户|时间
列：(sf:c1) // 对什么产品做了什么操作，作为列(c1)
比如，c1: click from homepage; c2: click from ad; s1: search from homepage; b1: buy
```

### 3. 查询案例

**案例一: 谁的值等于 sku188**

```
scan 'test1', FILTER=>"ValueFilter(=,'binary:sku188')"

ROW                          COLUMN+CELL                    
 user1|ts2                   column=sf:c1, timestamp=1409122354918, value=sku188
```

**案例二: 谁的值包含 sku188**

```
scan 'test1', FILTER=>"ValueFilter(=,'substring:88')"

ROW                          COLUMN+CELL    
 user1|ts2                   column=sf:c1, timestamp=1409122354918, value=sku188
 user2|ts5                   column=sf:c2, timestamp=1409122355030, value=sku288
 
```

**案例三： 通过广告点击进来的(column为c2)值包含88的用户**

```
scan 'test1', FILTER=>"ColumnPrefixFilter('c2') AND ValueFilter(=,'substring:88')"
 
ROW                          COLUMN+CELL
user2|ts5                   column=sf:c2, timestamp=1409122355030, value=sku288
```

**案例四： 通过搜索进来的(column为s)值包含123或者222的用户**

```
scan 'test1', FILTER=>"ColumnPrefixFilter('s') AND ( ValueFilter(=,'substring:123') OR ValueFilter(=,'substring:222'))"

ROW                          COLUMN+CELL
 user1|ts3                   column=sf:s1, timestamp=1409122354954, value=sku123
 user2|ts6                   column=sf:s1, timestamp=1409122355970, value=sku222
```

**案例五： rowkey为user1开头的 **

```
scan 'test1', FILTER => "PrefixFilter ('user1')"

ROW                          COLUMN+CELL
 user1|ts1                   column=sf:c1, timestamp=1409122354868, value=sku1
 user1|ts2                   column=sf:c1, timestamp=1409122354918, value=sku188
 user1|ts3                   column=sf:s1, timestamp=1409122354954, value=sku123
```

**案例六： FirstKeyOnlyFilter: 一个rowkey可以有多个version,同一个rowkey的同一个column也会有多个的值, 只拿出key中的第一个column的第一个version
KeyOnlyFilter: 只要key,不要value**

```
scan 'test1', FILTER=>"FirstKeyOnlyFilter() AND ValueFilter(=,'binary:sku188') AND KeyOnlyFilter()"

ROW                          COLUMN+CELL
 user1|ts2                   column=sf:c1, timestamp=1409122354918, value=
```


