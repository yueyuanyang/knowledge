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


galera 集群安装
> yum -y install mysql-wsrep-5.7.x86_64 galera.x86_64 --nogpgcheck

### mysql 密码修改
#### 方法一：
```
step 1: SET PASSWORD = PASSWORD('your new password');
step 2: ALTER USER 'root'@'localhost' PASSWORD EXPIRE NEVER;
step 3: flush privileges;
```
#### 方法二：

> newpass=`grep 'password' /var/log/mysqld.log  | awk '{print $NF}'`;mysqladmin -p"$newpass" password 'Asiainfo@123'

### 构建自己的yum仓库
#### 创建yum repo
```
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



```
### 
