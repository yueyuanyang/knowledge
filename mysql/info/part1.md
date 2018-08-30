### centos7 主机名修改

#### 主机名修改
> hostnamectl set-hostname  asiainfo-159

####  ip及主机名映射
```
// 打开软件
vi /etc/hosts 92.168.242.130 
// 填写主机名
Master 
// 重启网关
systemctl restart network

```

#### galera 集群安装

```
yum -y install mysql-wsrep-5.7.x86_64 galera.x86_64 --nogpgcheck
systemctl start mysqld
systemctl enable mysqld
```

### mysql 密码修改
#### 修改集群密码
```
// my.conf添加
skip-grant-tables

// mysql 中
update mysql.user set authentication_string = password('root'), password_expired = 'N', password_last_changed = now() where user = 'root';
// 重启mysql
systemctl restart mysqld

```
#### 方法一：
```
step 1: SET PASSWORD = PASSWORD('Asiainfo@123');
step 2: ALTER USER 'root'@'localhost' PASSWORD EXPIRE NEVER;
step 3: flush privileges;
// 验证wsrep
show status like 'wsrep%'
//创建拷贝用户
GRANT ALL ON *.* TO 'asiainfo'@'192.168.8.%' IDENTIFIED BY 'Asiainfo@123';
flush privileges;
```

#### 方法二：

> newpass=`grep 'password' /var/log/mysqld.log  | awk '{print $NF}'`;mysqladmin -p"$newpass" password 'Asiainfo@123'

### 构建自己的yum仓库
#### 创建yum repo
```
0) 下载是修改yum参数
vim /etc/yum.conf
修改
keepcache=1
1) 拷贝rmp 到 galerar中
find  /var/cache/yum/x86_64/7/ -iname "*.rpm" -exec cp -a {} galera/ \;
2）构建 yum repo 仓库
2.1） 创建 vsftpd 服务器 createrepo
yum -y install vsftpd createrepo
拷贝到ftp服务器上
cp -r galera /var/ftp
2.2）创建yum仓库
createrepo /var/ftp/galera

2.3) 关闭防火墙
systemctl stop firewalld;systemctl disable firewalld
2.4) 启动ftp服务器
systemctl start vsftpd
systemctl enable vsftpd
```
#### 其他机器创建仓库
```
创建本地镜像repo
vi /etc/yum.repos.d/galera.repo
配置文件
[galera]
name=galera
baseurl=ftp://asiainfo-168/galera
gpgcheck=0

```
### galera 配置实例


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
```



