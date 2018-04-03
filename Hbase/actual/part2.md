## HBase - Filter - 过滤器的介绍以及使用

HBase 不仅提供了这些简单的查询，而且提供了更加高级的过滤器（Filter）来查询。

### 1.1 过滤器的两类参数

过滤器可以根据列族、列、版本等更多的条件来对数据进行过滤，基于 HBase 本身提供的三维有序（行键，列，版本有序），这些过滤器可以高效地完成查询过滤的任务，带有过滤器条件的 RPC 查询请求会把过滤器分发到各个 RegionServer（这是一个服务端过滤器），这样也可以降低网络传输的压力。
使用过滤器至少需要两类参数：

#### 1.1.1 一类是抽象的操作符

HBase 提供了枚举类型的变量来表示这些抽象的操作符：
- LESS
- LESS_OR_EQUAL
- EQUAL
- NOT_EQUAL
- GREATER_OR_EQUAL
- GREATER
- NO_OP

#### 1.1.2 另一类是比较器

代表具体的逻辑，例如字节级的比较，字符串级的比较等。

#### 1.2 比较器

比较器作为过滤器的核心组成之一，用于处理具体的比较逻辑，例如字节级的比较，字符串级的比较等。

#### 1.2.1 RegexStringComparator

支持正则表达式的值比较
```

Scan scan = new Scan();
RegexStringComparator comp = new RegexStringComparator("you."); // 以 you 开头的字符串
SingleColumnValueFilter filter = new SingleColumnValueFilter(Bytes.toBytes("family"), 
                                 Bytes.toBytes("qualifier"), CompareOp.EQUAL, comp);
scan.setFilter(filter);
```

#### 1.2.2 SubStringComparator

用于监测一个子串是否存在于值中，并且不区分大小写。
```
Scan scan = new Scan();
SubstringComparator comp = new SubstringComparator("1129"); // 查找包含 1129 的字符串
SingleColumnValueFilter filter = new SingleColumnValueFilter(Bytes.toBytes("family"), 
                                  Bytes.toBytes("qualifier"), CompareOp.EQUAL, comp);
scan.setFilter(filter);
```

#### 1.2.3 BinaryPrefixComparator

前缀二进制比较器。与二进制比较器不同的是，只比较前缀是否相同。

```
Scan scan = new Scan();
BinaryPrefixComparator comp = new BinaryPrefixComparator(Bytes.toBytes("yting")); //前缀为yting
SingleColumnValueFilter filter = new SingleColumnValueFilter(Bytes.toBytes("family"), 
                          Bytes.toBytes("qualifier"),  CompareOp.EQUAL, comp);
scan.setFilter(filter);
```

#### 1.2.4 BinaryComparator

二进制比较器，用于按字典顺序比较 Byte 数据值。
```
Scan scan = new Scan();
BinaryComparator comp = new BinaryComparator(Bytes.toBytes("xmei")); //
ValueFilter filter = new ValueFilter(CompareOp.EQUAL, comp);
scan.setFilter(filter);
```

#### 1.3 列值过滤器

#### 1.3.1 SingleColumnValueFilter

SingleColumnValueFilter 用于测试值的情况（相等，不等，范围等）

下面一个检测列族 family 下的列 qualifier 的列值和字符串 "my-value" 相等的部分示例代码 : 
```
Scan scan = new Scan();
SingleColumnValueFilter filter = new SingleColumnValueFilter(Bytes.toBytes("family"), 
                               Bytes.toBytes("qualifier"), CompareOp.EQUAL, Bytes.toBytes("my-value"));
scan.setFilter(filter);
```

#### 1.3.2 SingleColumnValueExcludeFilter

跟 SingleColumnValueFilter 功能一样，只是不查询出该列的值。

下面的代码就不会查询出 family 列族下 qualifier 列的值（列都不会查出来）

```
Scan scan = new Scan();
SingleColumnValueExcludeFilter filter = new SingleColumnValueExcludeFilter(Bytes.toBytes("family"), Bytes.toBytes("qualifier"), 
                                        CompareOp.EQUAL, Bytes.toBytes("my-value"));
scan.setFilter(filter);
```

#### 1.4 键值元数据过滤器

HBase 采用 "键值对" 保存内部数据，键值元数据过滤器评估一行的 "键" 是否保存在（如 ColumnFamily:Column qualifiers）。

#### 1.4.1 FamilyFilter 

用于过滤列族（通常在 Scan 过程中通过设定某些列族来实现该功能，而不是直接使用该过滤器）。
```
Scan scan = new Scan();
FamilyFilter filter = new FamilyFilter(CompareOp.EQUAL, new BinaryComparator(Bytes.toBytes("my-family"))); // 列族为 my-family
scan.setFilter(filter);
```
#### 1.4.2 QualifierFilter

用于列名（Qualifier）过滤。
```
Scan scan = new Scan();
QualifierFilter filter = new QualifierFilter(CompareOp.EQUAL, new BinaryComparator(Bytes.toBytes("my-column"))); // 列名为 my-column
scan.setFilter(filter);
```

#### 1.4.3 ColumnPrefixFilter 

用于列名（Qualifier）前缀过滤，即包含某个前缀的所有列名。
```
Scan scan = new Scan();
  ColumnPrefixFilter filter = new ColumnPrefixFilter(Bytes.toBytes("my-prefix")); // 前缀为 my-prefix
  scan.setFilter(filter);
```

### 1.4.4 MultipleColumnPrefixFilter

MultipleColumnPrefixFilter 与 ColumnPrefixFilter  的行为类似，但可以指定多个列名（Qualifier）前缀。
```
Scan scan = new Scan();
byte[][] prefixes = new byte[][]{Bytes.toBytes("my-prefix-1"), Bytes.toBytes("my-prefix-2")};
MultipleColumnPrefixFilter filter = new MultipleColumnPrefixFilter(prefixes); // 不解释，你懂的 、、、
scan.setFilter(filter);
```
#### 1.4.5 ColumnRangeFilter

该过滤器可以进行高效的列名内部扫描。（为何是高效呢？？？因为列名是已经按字典排序好的）HBase-0.9.2 版本引入该功能。
```
Scan scan = new Scan();
boolean minColumnInclusive = true;
boolean maxColumnInclusive = true;
ColumnRangeFilter filter = new ColumnRangeFilter(Bytes.toBytes("minColumn"), minColumnInclusive, 
                                                Bytes.toBytes("maxColumn"), maxColumnInclusive);
scan.setFilter(filter);
```

#### 1.6 DependentColumnFilter 

该过滤器尝试找到该列所在的每一行，并返回该行具有相同时间戳的全部键值对。
```
Scan scan = new Scan();
DependentColumnFilter filter = new DependentColumnFilter(Bytes.toBytes("family"), Bytes.toBytes("qualifier"));
scan.setFilter(filter);
```

#### 1.5 行键过滤器

#### 1.5.1 RowFilter 

行键过滤器，一般来讲，执行 Scan 使用 startRow/stopRow 方式比较好，而 RowFilter 过滤器也可以完成对某一行的过滤。
```
Scan scan = new Scan();
RowFilter filter = new RowFilter(CompareOp.EQUAL, new BinaryComparator(Bytes.toBytes("my-row-1")));
scan.setFilter(filter);
```

#### 1.5.2 RandomRowFilter

该过滤器是随机选择一行的过滤器。参数 chance 是一个浮点值，介于 0.1 和 1.0 之间。
```
Scan scan = new Scan();
float chance = 0.5f;
RandomRowFilter filter = new RandomRowFilter(chance); // change 在 0.1 ~ 1.0 之间的浮点值
scan.setFilter(filter);
```
#### 1.6 功能过滤器

#### 1.6.1 PageFilter

**用于按行分页。**
```
long pageSize = 10;
int totalRowsCount = 0;
PageFilter filter = new PageFilter(pageSize);
byte[] lastRow = null;
while(true) {
 Scan scan = new Scan();
 scan.setFilter(filter);
 if(lastRow != null) {
  byte[] postfix = Bytes.toBytes("postfix");
  byte[] startRow = Bytes.add(lastRow, postfix);
  scan.setStartRow(startRow);
  System.out.println("start row : " + Bytes.toString(startRow));
 }
 
 ResultScanner scanner = _hTable.getScanner(scan);
 int localRowsCount = 0;
 for(Result result : scanner) {
  System.out.println(localRowsCount++ + " : " + result);
  totalRowsCount++;
  lastRow = result.getRow(); // ResultScanner 的结果集是排序好的，这样就可以取到最后一个 row 了
 }
 scanner.close();
 
 if(localRowsCount == 0) break;
}
System.out.println("total rows is : " + totalRowsCount);
```

#### 1.6.2 FirstKeyOnlyFilter

该过滤器只查询每个行键的第一个键值对，在统计计数的时候提高效率。（HBase-Coprocessor 做 RowCount 的时候可以提高效率）。
```
Scan scan = new Scan();
FirstKeyOnlyFilter filter = new FirstKeyOnlyFilter(); // 只查询每个行键的第一个键值对
scan.setFilter(filter);
```
#### 1.6.3 KeyOnlyFilter
```
Scan scan = new Scan();
KeyOnlyFilter filter = new KeyOnlyFilter(); // 只查询每行键值对中有 "键" 元数据信息，不显示值，可以提升扫描的效率
scan.setFilter(filter);
```

#### 1.6.4 InclusiveStopFilter

常规的 Scan 包含 start-row 但不包含 stop-row，如果使用该过滤器便可以包含 stop-row。
```
Scan scan = new Scan();
InclusiveStopFilter filter = new InclusiveStopFilter(Bytes.toBytes("stopRowKey"));
scan.setFilter(filter);
```

#### 1.6.5 ColumnPaginationFilter

按列分页过滤器，针对列数量很多的情况使用。
```
Scan scan = new Scan();
int limit = 0;
int columnOffset = 0;
ColumnPaginationFilter filter = new ColumnPaginationFilter(limit, columnOffset);
scan.setFilter(filter);
````

### 2 自定义过滤器

做法 : 继承 FilterBase，然后打成 jar 放到 $HBASE_HOEM/lib 目录下去（注意：需要重启 HBase 集群）
写自定义过滤器的时候需要熟悉过滤器的执行流程

```
编写自己的过滤器，如下代码(该例实现分页过滤器)：

package cn.cstor.cproc.java.util;

import java.io.DataInput;
import java.io.DataOutput;
import java.io.IOException;
import java.util.List;
import org.apache.commons.logging.Log;
import org.apache.commons.logging.LogFactory;
import org.apache.hadoop.hbase.KeyValue;
import org.apache.hadoop.hbase.exceptions.DeserializationException;
import org.apache.hadoop.hbase.filter.FilterBase;
import com.google.protobuf.InvalidProtocolBufferException;

public class RowPaginationFilter extends FilterBase {

static final Log LOG = LogFactory.getLog(RowPaginationFilter.class);
private int rowsAccepted = 0;
private int offset = 0;
private int limit = 0;

public RowPaginationFilter() {

}

public RowPaginationFilter(int offset, int limit) {
   this.offset = offset;
   this.limit = limit;
}

@Override
public void reset() {
// noop
}

@Override
public boolean filterAllRemaining() {
    return this.rowsAccepted > this.limit + this.offset;
}

@Override
public boolean filterRowKey(byte[] rowKey, int offset, int length) {
   return false;
}

public ReturnCode filterKeyValue(KeyValue v) {
   return ReturnCode.INCLUDE;
}

@SuppressWarnings("deprecation")
@Override
public void filterRow(List ignored) {
   try {
     super.filterRow(ignored);
   } catch (IOException e) {
     e.printStackTrace();
   }
}

// true to exclude row, false to include row.

@Override
public boolean filterRow() {
    boolean isExclude = this.rowsAccepted < this.offset || this.rowsAccepted >= this.limit + this.offset;
    rowsAccepted++;
    return isExclude;
}

public void readFields(final DataInput in) throws IOException {

     this.offset = in.readInt();
     this.limit = in.readInt();
}

public void write(final DataOutput out) throws IOException {  
  out.write(offset);
  out.write(limit);
}

@Override  //重写该方法
    public byte[] toByteArray() {
         RowPaginationProto.RowPaginationFilter.Builder builder =
         RowPaginationProto.RowPaginationFilter.newBuilder();

        builder.setLimit(this.limit);
        builder.setOffset(this.offset);
        return builder.build().toByteArray();
    }


   //定义该方法，用于对象反序列化操作

    public static RowPaginationFilter parseFrom(final byte [] bytes) throws DeserializationException {
    RowPaginationProto.RowPaginationFilter proto = null;
        try {
            proto = RowPaginationProto.RowPaginationFilter.parseFrom(bytes);
        } catch (InvalidProtocolBufferException e) {
            throw new DeserializationException(e);
        }
        return new RowPaginationFilter(proto.getOffset(),proto.getLimit());
    }
}

注：自订filter中必须重写这两个个方法：
    public static Filter parseFrom(final byte [] pbBytes) throws DeserializationException
    public byte [] toByteArray()
    
3、将xxxx和xxxx这两个类用myeclipse打成jar包部署到hbase中（放到lib目录中或者放任意目录通过修改hbase_evn.sh配置文件HBASE_CLASSPATH指定该jar包路径也可以）注：jar包要分发到所有 regionserver上，因为过滤器是在各regionserver上执行的。

4、重启HBASE，测试过滤器是否生效。

```
