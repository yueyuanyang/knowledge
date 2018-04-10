
## Eclipse上传项目到Git


### 一、首先需要在eclipse上面安装一个插件：操作步骤：

1. 安装软件

![p1](https://github.com/yueyuanyang/knowledge/blob/master/git/img/p1.png)

2. 配置地址

![p2](https://github.com/yueyuanyang/knowledge/blob/master/git/img/p2.png)

**地址是：http://download.eclipse.org/egit/updates**

然后在我们同意协议，就接着下载了。下载完毕后，需要重启Eclipse

### 二、在我们的Eclipse上配置Git

![p3](https://github.com/yueyuanyang/knowledge/blob/master/git/img/p3.png)

- 这里的user.name 是你在https://github.com上注册用户名
- user.email是你在github上绑定的邮箱。

在这里配置user.name即可 

### 三、新建项目，并且把项目提交到本地仓库

![p4](https://github.com/yueyuanyang/knowledge/blob/master/git/img/p4.png)

![p5](https://github.com/yueyuanyang/knowledge/blob/master/git/img/p5.png)

![p6](https://github.com/yueyuanyang/knowledge/blob/master/git/img/p6.png)

至此，我们已经成功创建了GIT仓库，但是文件夹处于未提交状态，类似SVN，在文件夹上有个问号的标识。

**接下来我们需要提交代码到本地仓库。**

![p7](https://github.com/yueyuanyang/knowledge/blob/master/git/img/p7.png)

**这时候我们在提交的时候可能会报一个错误：**

`There are no staged files.`

解决办法如下图：

![p8](https://github.com/yueyuanyang/knowledge/blob/master/git/img/p8.png)

这时候我们点击提交的时候就可以看到正确的界面了：

![p9](https://github.com/yueyuanyang/knowledge/blob/master/git/img/p9.png)

提交后，我们发现熟悉的小黄桶出现啦！这就意味着您已经成功提交到本地仓库了。

![p10](https://github.com/yueyuanyang/knowledge/blob/master/git/img/p10.png)

### 四、将本地代码提交到远程GIT仓库中

首先打开自己的GitHub，创建一个仓库：

![p11](https://github.com/yueyuanyang/knowledge/blob/master/git/img/p11.png)

点击Create repository后我们的仓库就创建好了。接着我们看到一个界面，有一个HTTP地址，如下图，这个就是你http协议的远程仓库地址

![p12](https://github.com/yueyuanyang/knowledge/blob/master/git/img/p12.png)

准备工作完毕后，我们打开eclipse来将项目上传到远程仓库：


下面输入的是你GITHUB的账号密码

![p13](https://github.com/yueyuanyang/knowledge/blob/master/git/img/p13.png)

![p14](https://github.com/yueyuanyang/knowledge/blob/master/git/img/p14.png)

![p15](https://github.com/yueyuanyang/knowledge/blob/master/git/img/p15.png)

这样我们就把代码提交到远程仓库了。我们打开github来查看：

![p16](https://github.com/yueyuanyang/knowledge/blob/master/git/img/p16.png)

OK，我们已经成功上传了。

希望可以帮到大家！









