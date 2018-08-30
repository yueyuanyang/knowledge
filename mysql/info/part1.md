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
