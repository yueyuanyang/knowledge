## 设计模式 —— 适配器

**定义** ：适配器模式将某个类的接口转换成客户端期望的另一个接口表示，主的目的是兼容性，让原本因接口不匹配不能一起工作的两个类可以协同工作。其别名为包装器(Wrapper)。

属于结构型模式

主要分为三类：
- 类适配器模式
- 对象的适配器模式
- 接口的适配器模式。

**本文定义：**

需要被适配的类、接口、对象（我们有的），简称 src（source） 最终需要的输出（我们想要的），简称 dst (destination，即Target)适配器称之为 Adapter 。
一句话描述适配器模式的感觉： src->Adapter->dst,即src以某种形式（三种形式分别对应三种适配器模式）给到Adapter里，最终转化成了dst。

### 二、类适配器模式

一句话描述：Adapter类，通过继承 src类，实现 dst 类接口，完成src->dst的适配。

别的文章都用生活中充电器的例子来讲解适配器,的确，这是个极佳的举例

充电器本身相当于Adapter，220V交流电相当于src，我们的目dst标是5V直流电。 

**我们现有的src类：**

```
public class Voltage220 {
    public int output220V() {
        int src = 220;
        System.out.println("我是" + src + "V");
        return src;
    }
}
```
**我们想要的dst接口：**
```
//介绍：dst接口：客户需要的5V电压

public interface Voltage5 {
    int output5V();
}

```

**适配器**

```
/**
 * 介绍：Adapter类：完成220V-5V的转变
 * 通过继承src类，实现 dst 类接口，完成src->dst的适配。
 */
 
 public class VoltageAdapter extends Voltage220 implements Voltage5 {
    @Override
    public int output5V() {
        int src = output220V();
        System.out.println("适配器工作开始适配电压");
        int dst = src / 44;
        System.out.println("适配完成后输出电压：" + dst);
        return dst;
    }
}

```
**Client类**

```
// 介绍：Client类：手机 .需要5V电压
public class Mobile {
    /**
     * 充电方法
     */
    public void charging(Voltage5 voltage5) {
        if (voltage5.output5V() == 5) {
            System.out.println("电压刚刚好5V，开始充电");
        } else if (voltage5.output5V() > 5) {
            System.out.println("电压超过5V，都闪开 我要变成note7了");
        }
    }

```

**测试代码：**

```
public class test {

    public static void main(String[] args) {
        
        System.out.println("===============类适配器==============");
        Mobile mobile = new Mobile();
        mobile.charging(new VoltageAdapter());
    }
}

```
**小结:**

Java这种单继承的机制，所有需要继承的我个人都不太喜欢。 所以类适配器需要继承src类这一点算是一个缺点， 因为这要求dst必须是接口，有一定局限性; 且src类的方法在Adapter中都会暴露出来，也增加了使用的成本。

但同样由于其继承了src类，所以它可以根据需求重写src类的方法，使得Adapter的灵活性增强了。


### 三、对象适配器模式（常用）:

基本思路和类的适配器模式相同，只是将Adapter类作修改，这次不继承src类，而是持有src类的实例，以解决`兼容性`的问题。 
即：`持有` src类，`实现` dst 类`接口`，完成src->dst的适配。 
（根据“合成复用原则”，在系统中尽量使用关联关系来替代继承关系，因此大部分结构型模式都是对象结构型模式。） 

**实例 1**

**Adapter类如下：**

```
/**
 * 介绍：对象适配器模式：
 * 持有 src类，实现 dst 类接口，完成src->dst的适配。 。以达到解决**兼容性**的问题。
 */

public class VoltageAdapter2 implements Voltage5 {
    private Voltage220 mVoltage220;

    public VoltageAdapter2(Voltage220 voltage220) {
        this.mVoltage220 = voltage220;
    }

    @Override
    public int output5V() {
        int dst = 0;
        if (null != mVoltage220) {
            int src = mVoltage220.output220V();
            System.out.println("对象适配器工作，开始适配电压");
            dst = src / 44;
            System.out.println("适配完成后输出电压：" + dst);
        }
        return dst;
    }
}
```
**测试代码：**

```
public class test {

    public static void main(String[] args) {
        
     System.out.println("\n===============对象适配器==============");
        VoltageAdapter2 voltageAdapter2 = new VoltageAdapter2(new Voltage220());
        Mobile mobile2 = new Mobile();
        mobile2.charging(voltageAdapter2);
    }
}

```

### 实战

首先想象这么一个场景，有一天，你的老板交给你这么一个任务，查看你们新上线的游戏中每个服的在线人数。对于相应的功能，相应的接口已经提供，只需要调用就行，例如你们的游戏有三个服务器，只要调用Utility.getOnlinePlayerCount(int)，传入相应的数值，就能得到在线人数，如果你传入了一个不存在的服，则会返回-1。然后你只要将得到的数据拼装成XML就好，具体的显示功能由你的老板来完成。于是，初步的代码便很快写出来:

首先定义一个用于统计在线人数的接口PlayerCount，如下：
```
public interface PlayerCount {
   String getServerName();
   int getPlayerCount();

}

```

接着定义三个实现了PlayerCount接口的类，分别对应三个不同的服务器：

```
public class ServerOne inplement PlayerCount {
    public String getServerName() {
        return "一服"；
    }
    
    public int getPlayerCount() {
    
       return Utility.getOnlinePlayerCount(1)
    }

}

public class ServerOne inplement PlayerCount {
    public String getServerName() {
        return "二服"；
    }
    
    public int getPlayerCount() {
    
       return Utility.getOnlinePlayerCount(2)
    }

}

public class ServerOne inplement PlayerCount {
    public String getServerName() {
        return "三服"；
    }
    
    public int getPlayerCount() {
    
       return Utility.getOnlinePlayerCount(3)
    }

}

```

然后定义一个封装XML的类：XMLBuilder

```
public class XMLBuilder {

    public static String buildXML(PlayerCount player) {
        StringBuilder builder = new StringBuilder();
        builder.append("<root>")
        builder.append("<server>").append(player.getServerName()).append("</server>")
        builder.append("<player_count>").append(player.getPlayerCount()).append("</player_count>")
        builder.append("</root>")
        return builder.toString;
    }

}

```
**测试类***

```
public class Test {
   public static void main(String[] args){
   
        XMLBuilder.buildXML(new ServerOne());
   
   }

}

```
相应的，二服，三服也比较简单，对自己所写的代码，自己也是很满意的。在你提交的时候，你的老板，突然微笑着告诉你，Utility.getOnlinePlayerCount(int)方法主要是针对二、三服的，是后来加上去的，而一服，一直没改过来，是用ServerFirst这个类。

咦，你脑子一动，那只要将Utility.getOnlinePlayerCount(int)方法修改下，将一服加进去就可以了，但是现在问题来了，Utility和ServerFirst这两个类都已经被打到Jar包里了，没法修改啊，咋办呢。分析一下，因为我们前期定义的PlayerCount接口，然后各个实现类的getPlayCount()方法里面，具体的去调用Utility.getOnlinePlayerCount(int)方法，然而，Utility.getOnlinePlayerCount(int)方法并没有实现Utility.getOnlinePlayerCount(1)，所以这样不行。对于一服，它自己的获取在线人数的方法是ServerFirst类里面的getOnlinePlayerCount()方法，也就是ServerFirst.getOnlinePlayerCount()方法。

那么我们怎么做呢，很简单，核心思想就是只要能让两个互不兼容的接口能正常对接就行了，上面的代码中，XMLBuilder中使用PlayerCount这个接口来拼装XML，而ServerFirst并没有实现PlayerCount这个接口，所以在XMLBuilder中，就没有办法使用ServerFirst这个类作为参数，并调用其中的获取在线人数的方法，这个时候就需要一个适配器类来为XMLBuilder和ServerFirst之间搭起一座桥梁，把ServerFirst传到PlayerCount中去，便可以调用ServerFirst中的方法了，毫无疑问，ServerOne就将充当适配器类的角色。修改ServerOne的代码，如下所示：

```
public class ServerAdaper implement PlayerCount {
    
    private ServerFirst mSeverFirst;
    public ServerAdaper() {
        mSeverFirst = new ServerFrist;
    }
    
   public String getServerName() {
        return "一服"；
    }
    
    public int getPlayerCount() {    
       return mSeverFirst.getOnlinePlayerCount(1); // 调用mSeverFirst中的getOnlinePlayerCount方法
    }

}

```

**测试类***

```
public class Test {
   public static void main(String[] args){
        XMLBuilder.buildXML(new ServerAdaper());
   
   }

}

```
#### 接口适配器

#### 应用场景：不想实现接口中的所有方法

**-创建接口**
```
/**
 * Created by 谭健 2017年7月2日 20:56:08
 * 定义端口接口，提供通信服务
 */
public interface Port {

    // 远程SSH端口22
    public void SSH();

    // 网络端口80
    public void NET();

    // Tomcat容器端口8080
    public void Tomcat();

    // Mysql数据库端口3306
    public void Mysql();

    // Oracle数据库端口1521
    public void Oracle();

    // 文件传输FTP端口21
    public void FTP();
}
```
**2 - 伪实现接口**

```
/**
 * 定义抽象类实现端口接口，但是什么事情都不做
 */
public abstract class Wrapper implements Port{

    @Override
    public void SSH(){};

    @Override
    public void NET(){};

    @Override
    public void Tomcat(){};

    @Override
    public void Mysql(){};

    @Override
    public void Oracle(){};

    @Override
    public void FTP(){};
}
```

**3 - 覆写伪接口实现一种功能**
```
/**
 * 提供聊天服务
 * 需要网络和文件传输功能
 */
public class Chat extends Wrapper{

    @Override
    public void NET(){ System.out.println("Hello world!"); };

    @Override
    public void FTP(){ System.out.println("File upload succeddful!"); };

}
```
**4 - 覆写伪接口实现另外一种功能**
```
/**
 * 网站服务器
 * 需要Tomcat容器，Mysql数据库，网络服务，远程服务
 */
public class Server extends Wrapper{

    @Override
    public void SSH(){ System.out.println("Connect success!"); };

    @Override
    public void NET(){ System.out.println("Hello WWW!"); };

    @Override
    public void Tomcat(){ System.out.println("Tomcat 9 is running!"); };

    @Override
    public void Mysql(){ System.out.println("Mysql is running!"); };
}
```
**5 - 运行服务**
```
/**
 * 运行器
 * 运行聊天服务和服务器
 */
public class Start {
    private static Port chatPort = new Chat();
    private static Port serverPort = new Server();
    public static void main(String[] args) {
    // 聊天服务
    chatPort.FTP();
    chatPort.NET();

    // 服务器
    serverPort.Mysql();
    serverPort.SSH();
    serverPort.Tomcat();
    serverPort.NET();
    }
}
```




#### 4、适配器模式应用场景

**类适配器与对象适配器的使用场景一致，仅仅是实现手段稍有区别，二者主要用于如下场景**：

（1）想要使用一个已经存在的类，但是它却不符合现有的接口规范，导致无法直接去访问，这时创建一个适配器就能间接去访问这个类中的方法。

（2）我们有一个类，想将其设计为可重用的类（可被多处访问），我们可以创建适配器来将这个类来适配其他没有提供合适接口的类。

以上两个场景其实就是从两个角度来描述一类问题，那就是要访问的方法不在合适的接口里，一个从接口出发（被访问），一个从访问出发（主动访问）。

**接口适配器使用场景**：

（1）想要使用接口中的某个或某些方法，但是接口中有太多方法，我们要使用时必须实现接口并实现其中的所有方法，可以使用抽象类来实现接口，并不对方法进行实现（仅置空），然后我们再继承这个抽象类来通过重写想用的方法的方式来实现。这个抽象类就是适配器。


**小结:**

对象适配器和类适配器其实算是同一种思想，只不过实现方式不同。 
根据合成复用原则，组合大于继承， 
所以它解决了类适配器必须继承src的局限性问题，也不再强求dst必须是接口。 
同样的它使用成本更低，更灵活。

（和装饰者模式初学时可能会弄混，这里要搞清，装饰者是对src的装饰，使用者毫无察觉到src已经被装饰了（使用者用法不变）。 这里对象适配以后，使用者的用法还是变的。 

**即，装饰者用法： setSrc->setSrc，对象适配器用法：setSrc->setAdapter.)**




