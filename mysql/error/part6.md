##   mysql无主键无索引表导致同步延迟

### Mysql主从同步延迟发生

**现象：**

```
Show slave status\G;

  Relay_Master_Log_File: mysql_bin.002597 

   Exec_Master_Log_Pos: 645248292

   Seconds_Behind_Master: 2447

   Slave_SQL_Running_State: Reading event from the relay log
   
```

**pos一直保持不变，并且behind一直在增加**

备库执行：

> Show processlist;

**SQL thread State列状态如下：**

> Reading event from the relay log

代表 线程已经从中继日志读取一个事件，可以对事件进行处理了。

**查看binlog：**

```
/home/mysql/mysql3502/bin/mysqlbinlog --start-position="645248292" --base64-output=decode-rows -v ./mysql_bin.002597 >mysqlbinlog.log

vi mysqlbinlog.log
/*!50530 SET @@SESSION.PSEUDO_SLAVE_MODE=1*/;
/*!40019 SET @@session.max_insert_delayed_threads=0*/;
/*!50003 SET @OLD_COMPLETION_TYPE=@@COMPLETION_TYPE,COMPLETION_TYPE=0*/;
DELIMITER /*!*/;
# at 645248292
#170226  2:28:35 server id 5123502  end_log_pos 645248340 CRC32 0x081bbaca      GTID [commit=yes]
SET @@SESSION.GTID_NEXT= '5c580bf9-0161-4e14-943b-751a3a09508b:5817253'/*!*/;
# at 645248340
.
.
BEGIN
/*!*/;
# at 645248419
#170226  2:28:35 server id 5123502  end_log_pos 645248550 CRC32 0xbf8a6c8c      Table_map: `****`.`****` mapped to number 41501
# at 645248550
#170226  2:28:35 server id 5123502  end_log_pos 645256612 CRC32 0x7c9e1837      Delete_rows: table id 41501
# at 645256612
#170226  2:28:35 server id 5123502  end_log_pos 645264682 CRC32 0xa1e4d6e7      Delete_rows: table id 41501
# at 645264682
#170226  2:28:35 server id 5123502  end_log_pos 645272881 CRC32 0xa1d560d0      Delete_rows: table id 41501
# at 645272881
#170226  2:28:35 server id 5123502  end_log_pos 645281037 CRC32 0x6e9df23b      Delete_rows: table id 41501
# at 645281037
#170226  2:28:35 server id 5123502  end_log_pos 645289102 CRC32 0x8bb31581      Delete_rows: table id 41501
# at 645289102
#170226  2:28:35 server id 5123502  end_log_pos 645297192 CRC32 0xcc940bb0      Delete_rows: table id 4150

```

