## 设计模式 —— 策略模式

### 概述

方法是类中最重要的组成部分，一个方法的方法体由一系列语句构成，也就是说一个方法的方法体是一个算法。在某些设计中，一个类的设计人员经常可能涉及这样的问题：由于用户需求的变化，导致经常需要修改类中某个方法的方法体，即需要不断地变化算法。在这样的情况下可以考虑使用策略模式。

策略模式是处理算法不同变体的一种成熟模式，策略模式通过接口或抽象类封装算法的标识，即在接口中定义一个抽象方法，实现该接口的类将实现接口中的抽象方法。策略模式把针对一个算法标识的一系列具体算法分别封装在不同的类中，使得各个类给出的具体算法可以相互替换。

在策略模式中，封装算法标识的接口称作策略，实现该接口的类称作具体策略。

### 模式的结构

策略模式的结构包括三种角色：

（1）策略（Strategy）：策略是一个接口，该接口定义若干个算法标识，即定义了若干个抽象方法。

（2）具体策略（ConcreteStrategy）：具体策略是实现策略接口的类。具体策略实现策略接口所定义的抽象方法，即给出算法标识的具体算法。

（3）上下文（Context）：上下文是依赖于策略接口的类，即上下文包含有策略声明的变量。上下文中提供了一个方法，该方法委托策略变量调用具体策略所实现的策略接口中的方法。

### 策略模式的优点

（1）上下文和具体策略是松耦合关系。因此上下文只知道它要使用某一个实现Strategy接口类的实例，但不需要知道具体是哪一个类。

（2）策略模式满足“开-闭原则”。当增加新的具体策略时，不需要修改上下文类的代码，上下文就可以引用新的具体策略的实例。

### 总结一下OO的原则：

1、封装变化（把可能变化的代码封装起来）

2、多用组合，少用继承（我们使用组合的方式，为客户设置了算法）

3、针对接口编程，不针对实现（对于Role类的设计完全的针对角色，和技能的实现没有关系）

### 开始实例

下面我就以角色游戏为背景，为大家介绍：假设公司需要做一款武侠游戏，我们就是负责游戏的角色模块，需求是这样的：每个角色对应一个名字，每类角色对应一种样子，每个角色拥有一个逃跑、攻击、防御的技能。

**初步的代码：**

```
package com.zhy.bean;  
  
/** 
 * 游戏的角色超类 
 *  
 * @author zhy 
 *  
 */  
public abstract class Role  
{  
    protected String name;  
  
    protected abstract void display();  
  
    protected abstract void run();  
  
    protected abstract void attack();  
  
    protected abstract void defend();  
  
}  

```

**创建角色**

```
package com.zhy.bean;  
  
public class RoleA extends Role  
{  
    public RoleA(String name)  
    {  
        this.name = name;  
    }  
  
    @Override  
    protected void display()  
    {  
        System.out.println("样子1");  
    }  
  
    @Override  
    protected void run()  
    {  
        System.out.println("金蝉脱壳");  
    }  
  
    @Override  
    protected void attack()  
    {  
        System.out.println("降龙十八掌");  
    }  
  
    @Override  
    protected void defend()  
    {  
        System.out.println("铁头功");  
    }  
  
}  

```

没几分钟，你写好了上面的代码，觉得已经充分发挥了OO的思想，正在窃喜，这时候项目经理说，再添加两个角色

RoleB(样子2 ，降龙十八掌，铁布衫，金蝉脱壳)。
RoleC(样子1，拥有九阳神功，铁布衫，烟雾弹)。

于是你觉得没问题，开始写代码，继续集成Role，写成下面的代码：
```
package com.zhy.bean;  
  
public class RoleB extends Role  
{  
    public RoleB(String name)  
    {  
        this.name = name;  
    }  
  
    @Override  
    protected void display()  
    {  
        System.out.println("样子2");  
    }  
  
    @Override  
    protected void run()  
    {  
        System.out.println("金蝉脱壳");//从RoleA中拷贝  
    }  
  
    @Override  
    protected void attack()  
    {  
        System.out.println("降龙十八掌");//从RoleA中拷贝  
    }  
  
    @Override  
    protected void defend()  
    {  
        System.out.println("铁布衫");  
    }  
  
}  

```
**创建角色**
```
package com.zhy.bean;  
  
public class RoleC extends Role  
{  
    public RoleC(String name)  
    {  
        this.name = name;  
    }  
  
    @Override  
    protected void display()  
    {  
        System.out.println("样子1");//从RoleA中拷贝  
    }  
  
    @Override  
    protected void run()  
    {  
        System.out.println("烟雾弹");  
    }  
  
    @Override  
    protected void attack()  
    {  
        System.out.println("九阳神功");  
    }  
  
    @Override  
    protected void defend()  
    {  
        System.out.println("铁布衫");//从B中拷贝  
    }  
  
}  

```






