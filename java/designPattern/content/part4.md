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

写完之后，你自己似乎没有当初那么自信了，你发现代码中已经存在相当多重复的代码，需要考虑重新设计架构了。于是你想，要不把每个技能都写成接口，有什么技能的角色实现什么接口，简单一想，觉得这想法高大尚啊，但是实现起来会发现，接口并不能实现代码的复用，每个实现接口的类，还是必须写自己写实现。

于是，we need change ! 遵循设计的原则，找出应用中可能需要变化的部分，把它们独立出来，不要和那些不需要变化的代码混在一起。我们发现，对于每个角色的display，attack，defend，run都是有可能变化的，于是我们必须把这写独立出来。再根据另一个设计原则：针对接口（超类型）编程，而不是针对实现编程，于是我们把代码改造成这样：

```  
public interface IAttackBehavior  
{  
    void attack();  
}  

```
``` 
public interface IDefendBehavior  
{  
    void defend();  
}  

```
``` 
public interface IDisplayBehavior  
{  
    void display();  
} 

```

**实现接口**

```
public class AttackJY implements IAttackBehavior  
{    
    @Override  
    public void attack()  
    {  
        System.out.println("九阳神功！");  
    }  
  
}

```
```
public class DefendTBS implements IDefendBehavior  
{  
    @Override  
    public void defend()  
    {  
        System.out.println("铁布衫");  
    }  
}  

```
```
public class RunJCTQ implements IRunBehavior  
{  
  
    @Override  
    public void run()  
    {  
        System.out.println("金蝉脱壳");  
    }  
}

```
**这时候需要对Role的代码做出改变：**

```
/** 
 * 游戏的角色超类 
 *  
 */  
public abstract class Role  
{  
    protected String name;  // 公共属性
  
    protected IDefendBehavior defendBehavior;         // 动态属性
    protected IDisplayBehavior displayBehavior;      // 动态属性
    protected IRunBehavior runBehavior;             // 动态属性
    protected IAttackBehavior attackBehavior;       // 动态属性
  
    public Role setDefendBehavior(IDefendBehavior defendBehavior)  
    {  
        this.defendBehavior = defendBehavior;  
        return this;  
    }  
  
    public Role setDisplayBehavior(IDisplayBehavior displayBehavior)  
    {  
        this.displayBehavior = displayBehavior;  
        return this;  
    }  
  
    public Role setRunBehavior(IRunBehavior runBehavior)  
    {  
        this.runBehavior = runBehavior;  
        return this;  
    }  
  
    public Role setAttackBehavior(IAttackBehavior attackBehavior)  
    {  
        this.attackBehavior = attackBehavior;  
        return this;  
    }  
  
    protected void display()  
    {  
        displayBehavior.display();  
    }  
  
    protected void run()  
    {  
        runBehavior.run();  
    }  
  
    protected void attack()  
    {  
        attackBehavior.attack();  
    }  
  
    protected void defend()  
    {  
        defendBehavior.defend();  
    }  
  
}  

```

**每个角色现在只需要一个name了：**

```
public class RoleA extends Role  
{  
    public RoleA(String name)  
    {  
        this.name = name;  
    }    
}  

```

#### 现在我们需要一个金蝉脱壳，降龙十八掌！，铁布衫，样子1的角色A只需要这样：

```
  
public class Test  
{  
    public static void main(String[] args)  
    {  
  
        Role roleA = new RoleA("A");  
  
        roleA.setAttackBehavior(new AttackXL())//  
                .setDefendBehavior(new DefendTBS())//  
                .setDisplayBehavior(new DisplayA())//  
                .setRunBehavior(new RunJCTQ());  
        System.out.println(roleA.name + ":");  
        roleA.run();  
        roleA.attack();  
        roleA.defend();  
        roleA.display();  
    }  
}  

```

经过我们的修改，现在所有的技能的实现做到了100%的复用，并且随便项目经理需要什么样的角色，对于我们来说只需要动态设置一下技能和展示方式，是不是很完美。恭喜你，现在你已经学会了策略模式，现在我们回到定义，定义上的算法族：其实就是上述例子的技能；定义上的客户：其实就是RoleA，RoleB...；我们已经定义了一个算法族（各种技能），且根据需求可以进行相互替换，算法（各种技能）的实现独立于客户（角色）。现在是不是很好理解策略模式的定义了。

> https://blog.csdn.net/lmj623565791/article/details/24116745







