### centos7 主机名修改

#### 主机名修改

为了便于识别和机器映射，需要修改主机饿名称
> hostnamectl set-hostname  asiainfo-159

####  ip及主机名映射

ip及域名的地址映射

```
// 打开软件
vi /etc/hosts 192.168.8.168 
// 填写主机名
silent-1 
// 重启网关
systemctl restart network
```

### galera版 mysql 集群安装

galera版的mysql，为插件化的mysql组件，提供了多台机器同步复制的功能，以保证mysql的高并发和高容错

本次集群安装才用yum形式去构建mysql galera版集群

| 名称 | 机器 |
| ------ | ------ |
| 机器-1 | 192.169.8.168 |
| 机器-2 | 192.169.8.158 |
| 机器-3 | 192.169.8.159 |

1) 创建galera 本地yum源(192.169.8.168 上操作)

其中: galera 下载地址: http://releases.galeracluster.com/mysql-wsrep-5.7/centos/7/x86_64/

构建自己的yum源

```
# vim /etc/yum.repos.d/galera.repo
// 编辑内容
[galera]
name=galera
baseurl=http://releases.galeracluster.com/mysql-wsrep-5.7/centos/7/x86_64/
gpgchectk=0

# yum list | egrep 'wsrep|galera' // 查看galera仓库

-------------------------------------------------------------------

// MariaDB 数据库安装(非必须)
# vim /etc/yum.repos.d/MariaDB.repo

// 编辑内容 
[mariadb]
name = MariaDB
baseurl = http://yum.mariadb.org/10.3/centos73-amd64/
gpgcheck = 0
```

根据本地yum源下载相应的rmp包

```
yum -y install mysql-wsrep-5.7.x86_64 galera.x86_64 --nogpgcheck

// `注意`：下载并修改yum参数,用于缓存数据
vim /etc/yum.conf
// 修改
keepcache=1

// 启动yum安装的mysql 数据库

systemctl start mysqld
systemctl enable mysqld
```

**构建自己的yum仓库**

**创建yum repo**

由于yum源下载数据较慢，故安装本地yum源服务器，以构建本地yum.repo

构建ftp和仓库的具体步骤如下：

```
1) 拷贝rmp 到 galerar中
find  /var/cache/yum/x86_64/7/ -iname "*.rpm" -exec cp -a {} galera/ \;

2) 构建 yum repo 仓库
2.1) 创建 vsftpd 服务器 createrepo
yum -y install vsftpd createrepo

//拷贝到ftp服务器上
cp -r galera /var/ftp

2.2)创建yum仓库
createrepo /var/ftp/galera

2.3) 关闭防火墙
systemctl stop firewalld;systemctl disable firewalld

2.4) 启动ftp服务器
systemctl start vsftpd
systemctl enable vsftpd
```

在其他机器上安装本地yum.repo只需要将url指向ftp本地地址即可

在其他机器上(192.169.8.159,192.169.8.158两台机器上相同的操作)：

```
// 创建本地镜像repo
vi /etc/yum.repos.d/galera.repo

// 配置文件
[galera]
name=galera
baseurl=ftp://asiainfo-168/galera
gpgcheck=0

```

完成以上步骤后，可以在机器上执行相同的操作：

```
yum -y install mysql-wsrep-5.7.x86_64 galera.x86_64 --nogpgcheck

// 启动yum安装的mysql 数据库
systemctl start mysqld
systemctl enable mysqld
```

### mysql 集群配置

安装完mysql集群后需要进行数据库的配置

**修改集群密码**

如果出现忘记root 的初始化密码，可进行如下的操作:

```
// my.conf添加
skip-grant-tables

// mysql 中
update mysql.user set authentication_string = password('root'), password_expired = 'N', password_last_changed = now() where user = 'root';
// 重启mysql
systemctl restart mysqld

```

mysql 5.7 要求新安装的mysql 服务器，需要修改原始密码

如下提供了两种修改密码的方法，如下所示:

查看原始密码的方法

```
newpass=`grep 'password' /var/log/mysqld.log  | awk '{print $NF}'`;
echo $newpass
```

**方法一**：

登陆mysql后，命令界面中进行修改

```
step 1: SET PASSWORD = PASSWORD('Asiainfo@123');
step 2: ALTER USER 'root'@'localhost' PASSWORD EXPIRE NEVER;
step 3: flush privileges;

// 验证wsrep
show status like 'wsrep%'
```

**方法二**：

脚本进行密码的修改

```
newpass=`grep 'password' /var/log/mysqld.log  | awk '{print $NF}'`;mysqladmin -p"$newpass" password 'Asiainfo@123'
```

### galera 配置实例

修改完mysql的基本操作后，我们可以进行galera的相关配置

第一步:

```
//mysql中创建拷贝用户
GRANT ALL ON *.* TO 'asiainfo'@'192.168.8.%' IDENTIFIED BY 'Asiainfo@123';
flush privileges;
```
第二步:

```
// 位置
vim /etc/my.cnf
//配置
server-id=1 // 集群id
binlog_format=row // binlog 格式
default_storage_engine=InnoDB // 存储引擎
innodb_file_per_table=1 // 独立表空间
innodb_autoinc_lock_mode=2

wsrep_on=ON
wsrep_provider=/usr/lib64/galera/libgalera_smm.so
wsrep_cluster_name='asiainfo' // 集群名字
wsrep_cluster_address='gcomm://' // 介绍人
wsrep_node_name='asiainfo-158'
wsrep_node_address='192.168.8.158'
wsrep_sst_auth=asiainfo:Asiainfo@123  // 登陆用户和密码
wsrep_sst_method=rsync // 数据传输的方式

// 重启mysql
systemctl restart mysqld

//验证是否启动
ss -tnlp |egrep '3306|4567'

// mysql命令行中查看
// 验证wsrep
show status like 'wsrep%'
```

注意事项:

1) server-id=1 集群的id 每个主机的id在整集群中唯一

2) wsrep_cluster_name 集群的名字，相同名字主机为一个集群

3) wsrep_cluster_address 介绍人, 当初始化的时候为空，独立起来一个集群
其他机器填写相应的介绍人的域名或IP,如wsrep_cluster_address='gcomm://192.168.8.158,192.168.8.159

4) wsrep_sst_auth mysql 创建用户的用户名和密码



### mycat 相关配置

mycat 采用离线下载

mycat 下载地址：
> http://dl.mycat.io/1.6.5/Mycat-server-1.6.5-release-20180122220033-unix.tar.gz

**安装及配置**

mycat 为java语言编写的插件，所有需在本机上安装java环境

```
// 下载 java包
# cd /opt/soft // 软件安装目录
# tar zxvf jdk-8u181-linux-x64.tar.gz

// 配置环境变量
# vim /etc/profile

JAVA_HOME=/opt/soft/jdk1.8.0_181
CLASSPATH=.:$JAVA_HOME/lib/tools.jar:$JAVA_HOME/lib/dt.jar
PATH=$JAVA_HOME/bin:$HOME/bin:$HOME/.local/bin:$PATH

// 生效配置文件
# source /etc/profile
```

**mycat 安装**

1) 解压及安装
```
// 解压
# cd /opt/soft
# tar zxvf mycat.tar.gz
```
简单配置文件样例:

```
# cd /opt/soft/mycat/conf

# vim server.xml
//  编辑内容
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mycat:server SYSTEM "server.dtd">
<mycat:server xmlns:mycat="http://io.mycat/">
	<system>
	<property name="useSqlStat">0</property>  <!-- 1为开启实时统计、0为关闭 -->
	<property name="useGlobleTableCheck">0</property>  <!-- 1为开启全加班一致性检测、0为关闭 -->
	<property name="sequnceHandlerType">2</property>
	<property name="processorBufferPoolType">0</property>
	<property name="handleDistributedTransactions">0</property>
		<property name="useOffHeapForMerge">1</property>
		<property name="memoryPageSize">1m</property>
		<property name="spillsFileBufferSize">1k</property>
		<property name="useStreamOutput">0</property>
		<property name="systemReserveMemorySize">384m</property>
		<property name="useZKSwitch">true</property>
	</system>
	<user name="asiainfo_in">
		<property name="password">123456</property>
		<property name="schemas">asiainfo</property>	
	</user>
</mycat:server>

// schame.xml 配置文件

<?xml version="1.0"?>
<!DOCTYPE mycat:schema SYSTEM "schema.dtd">
<mycat:schema xmlns:mycat="http://io.mycat/">

        <schema name="asiainfo" checkSQLschema="false" sqlMaxLimit="100" dataNode="dn1">
        </schema>
        <dataNode name="dn1" dataHost="asiainfopool" database="asiainfo" />
        <dataHost name="asiainfopool" maxCon="1000" minCon="10" balance="0"
                          writeType="0" dbType="mysql" dbDriver="native" switchType="1"  slaveThreshold="100">
                <heartbeat>show status like 'wsrep%'</heartbeat>
                <writeHost host="asiainfo-158" url="asiainfo-158:3306" user="asiainfo_out" password="Asiainfo@123"> </writeHost>
                <writeHost host="asiainfo-159" url="asiainfo-159:3306" user="asiainfo_out" password="Asiainfo@123"> </writeHost>
                <writeHost host="asiainfo-168" url="asiainfo-168:3306" user="asiainfo_out" password="Asiainfo@123"> </writeHost>
        </dataHost>
</mycat:schema>               
```
注意:  

balance

writeType

switchType

启动配置好的mycat服务

```
// mycat 启动使用
/mycat/bin/mycat start

jps // 查看进程
ps aux | grep java

// 8066 数据端口 
ss -tnlp | grep java

// 修改mycat 日志级别(debug)
vim /usr/local/mycat/conf/log4j2.xml

```

创建连接信息

> mysql -h192.168.8.168 -uasiainfo_in -p123456 -P8066

### haproxy 高可用配置
**haproxy下载和安装**

haproxy下载地址：http://www.haproxy.org/download

**1) 本案例使用yum 安装**

```
// yum 安装haproxy
# yum -y install haproxy

/ /创建haproxy.cfg配置文件
# vim /etc/haproxy/haproxy.cfg

// 给haproxy目录授权：
# sudo chmod -R 777 haproxy/
```
**2) 创建haproxy.cfg配置文件**

```
global
    log 127.0.0.1   local0  ##记日志的功能
    maxconn 4096
    #chroot /usr/local/haproxy
    user haproxy
    group haproxy
    daemon

defaults
    log global
    option dontlognull 	
    retries 3
    option redispatch
    maxconn 2000
    timeout connect 5000
    timeout client 50000
    timeout server 50000
    mode http
    option httplog

listen  admin_stats 192.168.8.200:48800 ##统计页面
    stats uri /admin-status 
    stats auth  admin:admin
    mode http
    option httplog

##客户端就是通过这个ip和端口进行连接，这个vip和端口绑定的是mycat8066端口
listen mycat_service #192.168.8.200:18066 
    bind 192.168.8.200:8096    
    mode tcp
    option tcplog
    option httpchk OPTIONS * HTTP/1.1\r\nHost:\ www
    balance roundrobin
    server mycat_168 192.168.8.168:8066 check port 48700 inter 5s rise 2 fall 3
    server mycat_159 192.168.8.159:8066 check port 48700 inter 5s rise 2 fall 3
    timeout server 20000

##客户端就是通过这个ip和端口进行连接，这个vip和端口绑定的是mycat9066端口
#192.168.57.200:19066 
listen mycat_admin 
    bind 192.168.8.200:8097
    mode tcp
    option tcplog
    option httpchk OPTIONS * HTTP/1.1\r\nHost:\ www
    balance roundrobin
    server mycat_168 192.168.8.168:9066 check port 48700 inter 5s rise 2 fall 3
    server mycat_159 192.168.8.159:9066 check port 48700 inter 5s rise 2 fall 3
    timeout server 20000
    
```
**3) haproxy记录日志**：

默认haproxy是不记录日志的，为了记录日志还需要配置syslog模块，在Linux下是rsyslogd服务

```
# yum -y install rsyslog

//记录haproxy日志的配置
# cd /etc/rsyslog.d/  

// 如果没有这个目录，新建：cd /etc    
mkdir rsyslog.d
cd /etc/rsyslog.d/
sudo touch haproxy.conf
sudo vi /etc/rsyslog.d/haproxy.conf

内容如下
$ModLoad imudp
$UDPServerRun 514
local0.* /var/log/haproxy.log

# vi /etc/rsyslog.conf
在#### RULES ####上面一行的地方加入以下内容：
# Include all config files in /etc/rsyslog.d/
$IncludeConfig /etc/rsyslog.d/*.conf
#### RULES ####
 
在 local7.* /var/log/boot.log 的下面加入以下内容（增加后的效果如下）：
# Save boot messages also to boot.log 
local7.*       /var/log/boot.log 
local0.*      /var/log/haproxy.log 
 
//保存，重启 rsyslog 服务
# service rsyslog restart

```

**4) 配置监听 mycat 是否存活**
   
这个步骤需要在安装mycat的机器上都操作，比如168,159

mycat上都需要添加检测端口 48700 的脚本，为此需要用到 xinetd，xinetd 为linux 系统的基础服务。 

首先在 xinetd 目录下面增加脚本与端口的映射配置文件 

1、如果 xinetd 没有安装，使用如下命令安装： 

> # yum install xinetd -y 

2.检查/etc/xinetd.conf 的末尾是否有这一句：

includedir /etc/xinetd.d  没有就加上
 
3.检查 /etc/xinetd.d 文件夹是否存在，不存在则创建 
```
cd /etc 
mkdir xinetd.d 
```

4、增加 /etc/xinetd.d/mycat_status 

监听 mycat 是否存活的配置,执行以下命令：
```
cd /etc 
mkdir xinetd.d 
cd /etc/xinetd.d/ 
vim /etc/xinetd.d/mycat_status

内容如下： 

service mycat_status
{
    flags = REUSE
    socket_type = stream
    port = 48700
    wait = no
    user = root
    server = /usr/local/bin/mycat_status
    log_on_failure += USERID
    disable = no
}

注意：等号两边有的有空格
给脚本授权，sudo chmod 777/etc/xinetd.d/mycat_status,这脚本需要执行权限
```

5 /usr/local/bin/mycat_status 脚本 

touch mycat_status

内容如下：
```
#!/bin/bash
#/usr/local/bin/mycat_status.sh
# This script checks if a mycat server is healthy running on localhost. It will
# return:
# "HTTP/1.x 200 OK\r" (if mycat is running smoothly)
# "HTTP/1.x 503 Internal Server Error\r" (else)
mycat=`/opt/soft/mycat/bin/mycat status |grep 'not running' | wc -l`
if [ "$mycat" = "0" ];
then
    /bin/echo -e "HTTP/1.1 200 OK\r\n"
else
    /bin/echo -e "HTTP/1.1 503 Service Unavailable\r\n"
fi

```

这个脚本也需要执行权限：

> sudo chmod 777 /usr/local/bin/mycat_status

检测脚本是否编写正确执行下面的代码：

> /usr/local/bin/mycat_status.sh，如果输出：HTTP/1.1 200 OK，（200需要mycat开启）则表示正确。

6、/etc/services 中加入 mycat_status 服务
```
cd /etc 
vi services
在末尾加入以下内容：
mycat_status    48700/tcp    # mycat_status

保存  
重启 xinetd 服务  
service xinetd restart

```
7、验证 mycat_status 服务是否启动成功 

> netstat -antup|grep 48700 

**4) 启动haproxy**

1.haproxy安装完成后配置文件检查，启动：

```
/usr/local/haproxy/sbin/haproxy -f /usr/local/haproxy/haproxy.cfg -c   检查配置文件是否正确
/usr/local/haproxy/sbin/haproxy -f /usr/local/haproxy/haproxy.cfg   启动配置文件 
```

 2，启动mycat   cd /usr/local/mycat/bin ./mycat start
      3,启动haproxy 
      /usr/local/haproxy/sbin/haproxy -f /usr/local/haproxy/haproxy.cfg -c   检查配置文件是否正确
     /usr/local/haproxy/sbin/haproxy -f /usr/local/haproxy/haproxy.cfg   启动配置文件
 
4，如果出现下面的错误：

> /usr/local/haproxy/sbin/haproxy.main()] Cannot raise FD limit to 8207, limit is 4096 

使用： ulimit a，查看打开文件数量

使用ulimit 8222设置打开文件数

然后在启动haproxy，有时启动需要加

> sudo /usr/local/haproxy/sbin/haproxy -f /usr/local/haproxy/haproxy.cfg 

5，通过haproxy登入mycat

>  mysql -umycat -p -h192.168.0.105 -P8096

用户名是mycat的用户名，密码是mycat的密码  

6. 开启远程访问haproxy端口，重启防火墙：

```
firewall-cmd --zone=public --add-port=48800/tcp --permanent
firewall-cmd --permanent --zone=public --add-port=48800/udp
firewall-cmd --reload

```  

7.打开浏览器，输入http://192.168.9.165:48800/admin-status
看到下面的页面说明启动，配置成功：

8.查看haproxy的日志：

> sudo tail -f /var/log/haproxy.log


### keepalived安装

**1) yum 源安装**
```
# yum -y install openssl-devel openssl
# yum -y install keepalived

```

**2) 创建配置文件和脚本**
```
mkdir /etc/keepalived/scripts
cd /etc/keepalived/scripts
```

**vim /etc/keepalived/keepalived.conf**

**master**
```
global_defs {
   router_id haproxy01
}
vrrp_script chk_haproxy
{
     script "/etc/keepalived/scripts/haproxy_check.sh"
     interval 2
     timeout 2
     fall 3
}
vrrp_instance haproxy {
    state MASTER
    interface ens160
    virtual_router_id 200
    priority  150
    authentication {
        auth_type PASS
        auth_pass 1111
    }

    virtual_ipaddress {
      192.168.8.200/24
    }

    track_script {
         chk_haproxy
    }
    notify_master /etc/keepalived/scripts/haproxy_master.sh
    notify_backup /etc/keepalived/scripts/haproxy_backup.sh
    notify_fault /etc/keepalived/scripts/haproxy_fault.sh
    notify_stop /etc/keepalived/scripts/haproxy_stop.sh
}

```

**backup**
```

global_defs {
    router_id haproxy01
}

vrrp_script chk_haproxy
{
     script "/etc/keepalived/scripts/haproxy_check.sh"
     interval 2
     timeout 2
     fall 3
}
vrrp_instance haproxy {
    state BACKUP
    interface ens160
    virtual_router_id 200
    priority  100
    authentication {
        auth_type PASS
        auth_pass 1111
    }
    virtual_ipaddress {
       192.168.8.200/24
    }
    track_script {
         chk_haproxy
    }
    notify_master  /etc/keepalived/scripts/haproxy_master.sh
    notify_backup /etc/keepalived/scripts/haproxy_backup.sh
    notify_fault /etc/keepalived/scripts/haproxy_fault.sh
    notify_stop /etc/keepalived/scripts/haproxy_stop.sh
}
```

**vim /etc/keepalived/scripts/haproxy_check.sh**

```
#!/bin/bash
STARTHAPROXY="haproxy -f /etc/haproxy/haproxy.cfg"
STOPKEEPALIVED="service haproxy stop"
LOGFILE="/var/log/keepalived-haproxy-state.log"
echo "[check_haproxy status]" >> $LOGFILE
A=`ps -C haproxy --no-header |wc -l`
echo "[check_haproxy status]" >> $LOGFILE
date >> $LOGFILE
if [ $A -eq 0 ];then
    echo $STARTHAPROXY >> $LOGFILE
    $STARTHAPROXY >> $LOGFILE 2>&1
    sleep 5
fi
if [ `ps -C haproxy --no-header |wc -l` -eq 0 ];then
    echo "fail: check_haproxy status" >> $LOGFILE
    exit 1
else
    echo "success: check_haproxy status" >> $LOGFILE
    exit 0
fi
echo "OK"

```

**vim /etc/keepalived/scripts/haproxy_master.sh**

```
#!/bin/bash
STARTHAPROXY=`haproxy -f /etc/haproxy/haproxy.cfg`
STOPHAPROXY=`ps -ef | grep haproxy | grep -v grep | awk '{print $2}'| xargs kill -s 9`
LOGFILE="/var/log/keepalived-haproxy-state.log"
echo "[master]" >> $LOGFILE
date >> $LOGFILE
echo "Being master...." >> $LOGFILE 2>&1
echo "stop haproxy...." >> $LOGFILE 2>&1
$STOPHAPROXY >> $LOGFILE 2>&1
echo "start haproxy...." >> $LOGFILE 2>&1
$STARTHAPROXY >> $LOGFILE 2>&1
echo "haproxy stared ..." >> $LOGFILE

```

**vim /etc/keepalived/scripts/haproxy_backup.sh**
```
#!/bin/bash
STARTHAPROXY=`haproxy -f /etc/haproxy/haproxy.cfg`
STOPHAPROXY=`ps -ef | grep haproxy | grep -v grep | awk '{print $2}'| xargs kill -s 9`
LOGFILE="/var/log/keepalived-haproxy-state.log"
echo "[backup]" >> $LOGFILE
date >> $LOGFILE
echo "Being backup...." >> $LOGFILE 2>&1
echo "stop haproxy...." >> $LOGFILE 2>&1
$STOPHAPROXY >> $LOGFILE 2>&1
echo "start haproxy...." >> $LOGFILE 2>&1
$STARTHAPROXY >> $LOGFILE 2>&1
echo "haproxy stared ..." >> $LOGFILE

```
**vim /etc/keepalived/scripts/haproxy_fault.sh**
```
#!/bin/bash
LOGFILE=/var/log/keepalived-haproxy-state.log
echo "[fault]" >> $LOGFILE
date >> $LOGFILE

```

**vim /etc/keepalived/scripts/haproxy_stop.sh**

```
#!/bin/bash
LOGFILE=/var/log/keepalived-haproxy-state.log
echo "[stop]" >> $LOGFILE
date >> $LOGFILE
```

```
 赋予脚本可执行权限
 > chmod 777 /etc/keepalived/scripts/*
 
 将keepalived加入自启动服务

chkconfig --add keepalived
chkconfig --level 2345 keepalived on

--启动服务
service keepalived start
```

注意: 配置好keepalived后，关闭haproxy,因为keepalived中带自启动程序


以上高可以mysql集群就部署完毕
```
访问：
// keepalived 虚拟地址
mysql -h192.168.8.200 -uasiainfo_in -p123456 -P8096

其中:
-h192.168.8.200 为keepalived 虚拟出来的IP
-uasiainfo_in 为mycat入口用户名
-p123456 为mycat入口密码
-P8096 为haproxy 代理端口
```




