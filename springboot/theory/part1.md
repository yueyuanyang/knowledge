## spring boot 编写架构及常用注解

## 编写基本架构作用

### 基本架构

数据访问的全部架构图解：

![架构图解](https://github.com/yueyuanyang/knowledge/blob/master/springboot/img/1.png)

**JAVA中Action层, Service层 ，modle层 和 Dao层的功能区分**

首先这是现在最基本的分层方式，结合了SSH架构。

- modle层: 对应的数据库表的实体类。
- Dao层: 使用了Hibernate连接数据库、操作数据库（增删改查）。
- Service层：引用对应的Dao数据库操作，在这里可以编写自己需要的代码（比如简单的判断）。
- Action层：引用对应的Service层，在这里结合Struts的配置文件，跳转到指定的页面，当然也能接受页面传递的请求数据，也可以做些计算处理。

以上的Hibernate，Struts，都需要注入到Spring的配置文件中，Spring把这些联系起来，成为一个整体。

一般java都是三层架构: 数据访问层（dao） 业务逻辑层（biz 或者services） 界面层（ui）
action 是业务层的一部分，是一个管理器 （总开关）（作用是取掉转）（取出前台界面的数据，调用biz方法，转发到下一个action或者页面）  
模型成（model）一般是实体对象(把现实的的事物变成java中的对象)作用是一暂时存储数据方便持久化（存入数据库或者写入文件）而是作为一个包裹封装一些数据来在不同的层以及各种java对象中使用  
dao是数据访问层 就是用来访问数据库实现数据的持久化（把内存中的数据永久保存到硬盘中）

Dao主要做数据库的交互工作 Modle 是模型 存放你的实体类 Service 做相应的业务逻辑处理 Action是一个控制器



