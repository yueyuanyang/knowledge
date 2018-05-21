### Elasticsearch shield权限管理详解

ElasticSearch本身没有权限管理模块，只要获取服务器的地址和端口，任何人都可以随意读写ElasticSearch的API并获取数据,这样非常不安全。如果获取了ES的访问IP和端口，一条命令就可以删除整个索引库。好在Elastic公司开发了安全插件shield来解决权限管理问题. https://www.elastic.co/products/shield

### 一、shield安装

```
bin/plugin install license
bin/plugin install shield
```

#### 第二步：重启Elasticsearch

```
bin/elasticsearch
```
#### 第二步：重启Elasticsearch
```
bin/shield/esusers useradd adminName -r admin
```
adminName是可以自定义的，执行该命令以后会提示输入密码，再次输入密码进行确认，之后管理员用户就添加成功了。

### 二、Java Api中使用shield

加入权限管理以后在JAVA Api中需要做修改：

#### 第一步：加入jar包 

直接导入 到elasticsearch-2.3.3/plugins/shield目录下拷贝shield-2.3.3.jar，加到CLASSPATH中. 或者maven导入

```
 <repositories>
      <!-- add the elasticsearch repo -->
      <repository>
         <id>elasticsearch-releases</id>
         <url>https://maven.elasticsearch.org/releases</url>
         <releases>
            <enabled>true</enabled>
         </releases>
         <snapshots>
            <enabled>false</enabled>
         </snapshots>
      </repository>
   </repositories>
   <dependencies>
      <!-- add the shield jar as a dependency -->
      <dependency>
         <groupId>org.elasticsearch.plugin</groupId>
         <artifactId>shield</artifactId>
         <version>2.2.0</version>
      </dependency>
   </dependencies>
```

#### 第二步：修改setting:

```
import org.elasticsearch.shield.ShieldPlugin;

Settings settings = Settings.settingsBuilder()
      .put("cluster.name", "bropen")
                   .put("shield.user","bropen:password")
                   .build();
```

#### 第三步：修改client

```
Client client = TransportClient.builder()
        .addPlugin(ShieldPlugin.class)
                 .settings(settings).build()
                 .addTransportAddress(new      
InetSocketTransportAddress(InetAddress.getByName("192.168.0.224"), 9300));
```

### 三、shield 用户管理

#### 3.1 新增用户
```
./elasticsearch-2.3.3/bin/shield/esusers  useradd bropen

回车后输入两次密码确认 
Enter new password: 
Retype new password:
```
#### 3.2 查看用户
```
./elasticsearch-2.3.3/bin/shield/esusers list

会列出当前所有的user和角色：
es_admin       : admin
bropen         : admin
```
#### 3.3 修改密码

给用户名为bropen的管理员修改密码：
```
./elasticsearch-2.3.3/bin/shield/esusers  passwd bropen

回车后输入两次密码确认
Enter new password:
Retype new password:
```

#### 3.4 删除用户
```
./elasticsearch-2.3.3/bin/shield/esusers userdel bropen
```
#### 查看所有可用命令：
```
./elasticsearch-2.3.3/bin/shield/esusers  -h
```

#### 可以查看当前用户是不是管理员角色：
```
./elasticsearch-2.3.3/bin/shield/esusers list
```

#### 给名为bropen的管理员添加角色：
```
./elasticsearch-2.3.3/bin/shield/esusers roles bropen -a admin
```
