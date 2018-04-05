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

### 类适配器模式

一句话描述：Adapter类，通过继承 src类，实现 dst 类接口，完成src->dst的适配。

别的文章都用生活中充电器的例子来讲解适配器,的确，这是个极佳的举例

充电器本身相当于Adapter，220V交流电相当于src，我们的目dst标是5V直流电。 
我们现有的src类：

**原来的类实现**

```
public class Source {
    public void method1() {
        System.out.println("我是方法一");
    }
}
```

**实现的接口**

```
/**
 * Created by dell on 2016/10/7.
 */
public interface AdapterInfc {
    void method1();
    void method2();
}

```

**适配器**

```
/**
 * Created by dell on 2016/10/7.
 * 类适配器是继承了原有的类
 */
public class AdapterInfcImpl extends Source implements AdapterInfc {

    public void method2() {
        System.out.println("第二期功能扩展，我要扩展方法");
    }
}
```

**测试类**

```
/**
 * 适配器模式
 * 类适配器
 * Created by dell on 2016/10/7.
 */
public class Test {
    public static void main(String[] agrs) {
        AdapterInft inft = new AdapterInftImpl();
        inft.method1();
        inft.method2();
    }
}
```



