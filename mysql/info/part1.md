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

#### 安装
```
// mycat 用户使用
/mycat/bin/mycat start

jps // 查看进程
ps aux | grep java

// 8066 数据端口 
ss -tnlp | grep java

// 修改mycat 日志级别(debug)
vim /usr/local/mycat/conf/log4j2.xml

//创建连接信息
mysql -h192.168.8.168 -uasiainfo_in -p123456 -P8066

```








