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

**Adapter类如下：**

```
/**
 * 介绍：对象适配器模式：
 * 持有 src类，实现 dst 类接口，完成src->dst的适配。 。以达到解决**兼容性**的问题。
 */

public class VoltageAdapter2 implements Voltage5 {
    private Voltage220 mVoltage220;

    public VoltageAdapter2(Voltage220 voltage220) {
        mVoltage220 = voltage220;
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
**小结:**

对象适配器和类适配器其实算是同一种思想，只不过实现方式不同。 
根据合成复用原则，组合大于继承， 
所以它解决了类适配器必须继承src的局限性问题，也不再强求dst必须是接口。 
同样的它使用成本更低，更灵活。

（和装饰者模式初学时可能会弄混，这里要搞清，装饰者是对src的装饰，使用者毫无察觉到src已经被装饰了（使用者用法不变）。 这里对象适配以后，使用者的用法还是变的。 

**即，装饰者用法： setSrc->setSrc，对象适配器用法：setSrc->setAdapter.)**
