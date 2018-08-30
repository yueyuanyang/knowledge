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
