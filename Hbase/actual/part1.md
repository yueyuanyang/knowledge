## HBase新版本Java API编程实战及基本操作方法封装

### Hbase 版本 0.98 

```
import java.io.IOException;
import java.util.Arrays;
import java.util.List;

import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.hbase.Cell;
import org.apache.hadoop.hbase.CellUtil;
import org.apache.hadoop.hbase.HBaseConfiguration;
import org.apache.hadoop.hbase.HColumnDescriptor;
import org.apache.hadoop.hbase.HTableDescriptor;
import org.apache.hadoop.hbase.TableName;
import org.apache.hadoop.hbase.client.Admin;
import org.apache.hadoop.hbase.client.Connection;
import org.apache.hadoop.hbase.client.ConnectionFactory;
import org.apache.hadoop.hbase.client.Delete;
import org.apache.hadoop.hbase.client.Get;
import org.apache.hadoop.hbase.client.Put;
import org.apache.hadoop.hbase.client.Result;
import org.apache.hadoop.hbase.client.ResultScanner;
import org.apache.hadoop.hbase.client.Scan;
import org.apache.hadoop.hbase.client.Table;
import org.apache.hadoop.hbase.util.Bytes;

public class MyHbaseApi {

    public static void main(String[] args) {
        Admin admin=null;
        Connection con=null;

        try {

            //1.获得配置文件对象
            Configuration conf=HBaseConfiguration.create();
                //设置配置参数
            conf.set("hbase.zookeeper.quorum", "192.168.52.140");
            //2.建立连接
             con=ConnectionFactory.createConnection(conf);
            //3.获得会话
             admin=con.getAdmin();
            //System.out.println(con);
            //System.out.println(admin);
            //4.操作
                //建立数据库
                    //创建表名对象
            TableName tn=TableName.valueOf("stu");
                    //a.判断数据库是否存在
            if(admin.tableExists(tn)){
                System.out.println("====> 表存在，删除表....");
                //先使表设置为不可编辑
                admin.disableTable(tn);
                //删除表
                admin.deleteTable(tn);
                System.out.println("表删除成功.....");
            }
            System.out.println("===>表不存在,创建表......");
            //创建表结构对象
            HTableDescriptor htd=new HTableDescriptor(tn);
            //创建列族结构对象
            HColumnDescriptor hcd1=new HColumnDescriptor("fm1");
            HColumnDescriptor hcd2=new HColumnDescriptor("fm2");
            htd.addFamily(hcd1);
            htd.addFamily(hcd2);
            //创建表
            admin.createTable(htd);

            System.out.println("创建表成功...");

            //向表中插入数据
                //a.单个插入
            Put put =new Put(Bytes.toBytes("row01"));//参数是行健row01
            put.addColumn(Bytes.toBytes("fm1"), Bytes.toBytes("col1"), Bytes.toBytes("value01"));

            //获得表对象
            Table table=con.getTable(tn);
            table.put(put);

            //批量插入
            Put put01 =new Put(Bytes.toBytes("row02"));//参数是行健row02
            put01.addColumn(Bytes.toBytes("fm2"), Bytes.toBytes("col2"), Bytes.toBytes("value02")).
            addColumn(Bytes.toBytes("fm2"), Bytes.toBytes("col3"), Bytes.toBytes("value03"));

            Put put02 =new Put(Bytes.toBytes("row03"));//参数是行健row01
            put02.addColumn(Bytes.toBytes("fm1"), Bytes.toBytes("col4"), Bytes.toBytes("value04"));

            List<Put> puts=Arrays.asList(put01,put02);

            //获得表对象
            Table table02=con.getTable(tn);
            table02.put(puts);

        //读取操作
            //scan
            Scan scan=new Scan();
            //获得表对象
            Table table03=con.getTable(tn);
            //得到扫描的结果集
            ResultScanner rs=table03.getScanner(scan);
            for(Result result:rs){
                //得到单元格集合
                List<Cell> cs=result.listCells();
                for(Cell cell:cs){
                    //取行健
                    String rowKey=Bytes.toString(CellUtil.cloneRow(cell));
                    //取到时间戳
                    long timestamp = cell.getTimestamp();
                    //取到族列
                    String family = Bytes.toString(CellUtil.cloneFamily(cell));  
                    //取到修饰名
                    String qualifier  = Bytes.toString(CellUtil.cloneQualifier(cell));  
                    //取到值
                    String value = Bytes.toString(CellUtil.cloneValue(cell));  

                    System.out.println(" ===> rowKey : " + rowKey + ",  timestamp : " + 
                    timestamp + ", family : " + family + ", qualifier : " + qualifier + ", value : " + value);
                }
            }
            System.out.println(" ===================get取数据==================");

            //get
            Get get = new Get(Bytes.toBytes("row02"));
            get.addColumn(Bytes.toBytes("fm2"), Bytes.toBytes("col2"));
            Table t04 = con.getTable(tn);
            Result r = t04.get(get);
            List<Cell> cs = r.listCells();
            for (Cell cell : cs) {
                String rowKey = Bytes.toString(CellUtil.cloneRow(cell));  //取行键
                long timestamp = cell.getTimestamp();  //取到时间戳
                String family = Bytes.toString(CellUtil.cloneFamily(cell));  //取到族列
                String qualifier  = Bytes.toString(CellUtil.cloneQualifier(cell));  //取到修饰名
                String value = Bytes.toString(CellUtil.cloneValue(cell));  //取到值

                System.out.println(" ===> rowKey : " + rowKey + ",  timestamp : " + 
                timestamp + ", family : " + family + ", qualifier : " + qualifier + ", value : " + value);
            }


          //删除数据
            System.out.println(" ===================delete删除数据==================");
            Delete delete = new Delete(Bytes.toBytes("row02"));
            delete.addColumn(Bytes.toBytes("fm2"), Bytes.toBytes("col2"));
            Table t05 = con.getTable(tn);
            t05.delete(delete);

            System.out.println(" ===================delete删除数据后==================");
            //scan
            scan = new Scan();
            table03 = con.getTable(tn);  //获得表对象
            rs = table03.getScanner(scan);
            for (Result result : rs) {
                cs = result.listCells();
                for (Cell cell : cs) {
                    String rowKey = Bytes.toString(CellUtil.cloneRow(cell));  //取行键
                    long timestamp = cell.getTimestamp();  //取到时间戳
                    String family = Bytes.toString(CellUtil.cloneFamily(cell));  //取到族列
                    String qualifier  = Bytes.toString(CellUtil.cloneQualifier(cell));  //取到修饰名
                    String value = Bytes.toString(CellUtil.cloneValue(cell));  //取到值

                    System.out.println(" ===> rowKey : " + rowKey + ",  timestamp : " + 
                    timestamp + ", family : " + family + ", qualifier : " + qualifier + ", value : " + value);
                }
            }

        } catch (IOException e) {
            e.printStackTrace();
        }

        //5.关闭

                try {
                    if (admin != null){
                        admin.close();
                    }
                    if(con != null){
                        con.close();
                    }
                } catch (IOException e) {
                    e.printStackTrace();
                }


    }

}

```

下面看对于上面的基本操作的封装,封装好了，以后就可以直接用

```
import java.io.IOException;
import java.util.HashMap;
import java.util.List;
import java.util.Map;

import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.hbase.Cell;
import org.apache.hadoop.hbase.CellUtil;
import org.apache.hadoop.hbase.HBaseConfiguration;
import org.apache.hadoop.hbase.HColumnDescriptor;
import org.apache.hadoop.hbase.HTableDescriptor;
import org.apache.hadoop.hbase.TableName;
import org.apache.hadoop.hbase.client.Admin;
import org.apache.hadoop.hbase.client.Connection;
import org.apache.hadoop.hbase.client.ConnectionFactory;
import org.apache.hadoop.hbase.client.Delete;
import org.apache.hadoop.hbase.client.Get;
import org.apache.hadoop.hbase.client.Put;
import org.apache.hadoop.hbase.client.Result;
import org.apache.hadoop.hbase.client.Table;
import org.apache.hadoop.hbase.util.Bytes;

public class HBaseUtil {
    private static Configuration conf;
    private static Connection con;
    // 初始化连接
    static {
        conf = HBaseConfiguration.create(); // 获得配制文件对象
        conf.set("hbase.zookeeper.quorum", "192.168.52.140");
        try {
            con = ConnectionFactory.createConnection(conf);// 获得连接对象
        } catch (IOException e) {
            e.printStackTrace();
        }
    }

    // 获得连接
    public static Connection getCon() {
        if (con == null || con.isClosed()) {
            try {
                con = ConnectionFactory.createConnection(conf);
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
        return con;
    }

    // 关闭连接
    public static void close() {
        if (con != null) {
            try {
                con.close();
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
    }

    // 创建表
    public static void createTable(String tableName, String... FamilyColumn) {
        TableName tn = TableName.valueOf(tableName);
        try {
            Admin admin = getCon().getAdmin();
            HTableDescriptor htd = new HTableDescriptor(tn);
            for (String fc : FamilyColumn) {
                HColumnDescriptor hcd = new HColumnDescriptor(fc);
                htd.addFamily(hcd);
            }
            admin.createTable(htd);
            admin.close();
        } catch (IOException e) {
            e.printStackTrace();
        }
    }

    // 删除表
    public static void dropTable(String tableName) {
        TableName tn = TableName.valueOf(tableName);
        try {
            Admin admin = con.getAdmin();
            admin.disableTable(tn);
            admin.deleteTable(tn);
            admin.close();
        } catch (IOException e) {
            e.printStackTrace();
        }
    }

    // 插入或者更新数据
    public static boolean insert(String tableName, String rowKey,
            String family, String qualifier, String value) {
        try {
            Table t = getCon().getTable(TableName.valueOf(tableName));
            Put put = new Put(Bytes.toBytes(rowKey));
            put.addColumn(Bytes.toBytes(family), Bytes.toBytes(qualifier),
                    Bytes.toBytes(value));
            t.put(put);
            return true;
        } catch (IOException e) {
            e.printStackTrace();
        } finally {
            HBaseUtil.close();
        }
        return false;
    }

    // 删除
    public static boolean del(String tableName, String rowKey, String family,
            String qualifier) {
        try {
            Table t = getCon().getTable(TableName.valueOf(tableName));
            Delete del = new Delete(Bytes.toBytes(rowKey));

            if (qualifier != null) {
                del.addColumn(Bytes.toBytes(family), Bytes.toBytes(qualifier));
            } else if (family != null) {
                del.addFamily(Bytes.toBytes(family));
            }
            t.delete(del);
            return true;
        } catch (IOException e) {
            e.printStackTrace();
        } finally {
            HBaseUtil.close();
        }
        return false;
    }
    //删除一行
    public static boolean del(String tableName, String rowKey) {
        return del(tableName, rowKey, null, null);
    }
    //删除一行下的一个列族
    public static boolean del(String tableName, String rowKey, String family) {
        return del(tableName, rowKey, family, null);
    }

    // 数据读取
    //取到一个值
    public static String byGet(String tableName, String rowKey, String family,
            String qualifier) {
        try {
            Table t = getCon().getTable(TableName.valueOf(tableName));
            Get get = new Get(Bytes.toBytes(rowKey));
            get.addColumn(Bytes.toBytes(family), Bytes.toBytes(qualifier));
            Result r = t.get(get);
            return Bytes.toString(CellUtil.cloneValue(r.listCells().get(0)));
        } catch (IOException e) {
            e.printStackTrace();
        }
        return null;
    }

    //取到一个族列的值
        public static Map<String, String> byGet(String tableName, String rowKey, String family) {
            Map<String, String> result = null ;
            try {
                Table t = getCon().getTable(TableName.valueOf(tableName));
                Get get = new Get(Bytes.toBytes(rowKey));
                get.addFamily(Bytes.toBytes(family));
                Result r = t.get(get);
                List<Cell> cs = r.listCells();
                result = cs.size() > 0 ? new HashMap<String, String>() : result;
                for (Cell cell : cs) {
                    result.put(Bytes.toString(CellUtil.cloneQualifier(cell)), Bytes.toString(CellUtil.cloneValue(cell)));
                }
            } catch (IOException e) {
                e.printStackTrace();
            }
            return result;
        }
    //取到多个族列的值
        public static Map<String, Map<String, String>> byGet(String tableName, String rowKey) {
            Map<String, Map<String, String>> results = null ;
            try {
                Table t = getCon().getTable(TableName.valueOf(tableName));
                Get get = new Get(Bytes.toBytes(rowKey));
                Result r = t.get(get);
                List<Cell> cs = r.listCells();
                results = cs.size() > 0 ? new HashMap<String, Map<String, String>> () : results;
                for (Cell cell : cs) {
                    String familyName = Bytes.toString(CellUtil.cloneFamily(cell));
                    if (results.get(familyName) == null)
                    {
                        results.put(familyName, new HashMap<String,  String> ());
                    }
                    results.get(familyName).put(Bytes.toString(CellUtil.cloneQualifier(cell)), Bytes.toString(CellUtil.cloneValue(cell)));
                }
            } catch (IOException e) {
                e.printStackTrace();
            }
            return results;
        }
}

```

**看看测试类**

```
package com.yc.hbase.util;

import static org.junit.Assert.*;

import java.util.Map;

import org.junit.Test;

public class HBaseUtilTest {

    @Test
    public void testCreateTable() {
        //创建表
        HBaseUtil.createTable("myTest", "myfc1", "myfc2", "myfc3");
        HBaseUtil.close();
        HBaseUtil.createTable("myTest02", "myfc1", "myfc2", "myfc3");
        HBaseUtil.close();
    }

    @Test
    public void testDropTable() {
        //删除表
        HBaseUtil.dropTable("myTest");
        HBaseUtil.dropTable("myTest02");
    }

    @Test
    public void testInsert(){
        //插入数据
        HBaseUtil.insert("myTest", "1", "myfc1", "sex", "men");
        HBaseUtil.insert("myTest", "1", "myfc1", "name", "xiaoming");
        HBaseUtil.insert("myTest", "1", "myfc1", "age", "32");
        HBaseUtil.insert("myTest", "1", "myfc2", "name", "xiaohong");
        HBaseUtil.insert("myTest", "1", "myfc2", "sex", "woman");
        HBaseUtil.insert("myTest", "1", "myfc2", "age", "23");
    }

    @Test
    public void testByGet(){
        //得到一行下一个列族下的某列的数据
        String result = HBaseUtil.byGet("myTest", "1", "myfc1", "name");
        System.out.println("结果是的： " + result);
        assertEquals("xiaosan", result);
    }

    @Test
    public void testByGet02(){
        //得到一行下一个列族下的所有列的数据
        Map<String, String> result = HBaseUtil.byGet("myTest", "1", "myfc1");
        System.out.println("结果是的： " + result);
        assertNotNull(result);
    }

    @Test
    public void testByGet03(){
        //得到一行的所有列族的数据
        Map<String, Map<String, String>> result = HBaseUtil.byGet("myTest", "1");

        System.out.println("所有列族的数据是:  "+result);
        System.out.println("结果是的： " + result.get("myfc1"));
        assertNotNull(result);
    }
}

```





