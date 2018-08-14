## 装饰者模式

### 意图

动态地给一个对象添加一些额外的职责。就增加功能来说， Decorator模式相比生成子类更为灵活。该模式以对客 户端透明的方式扩展对象的功能。

### 适用环境

（1）在不影响其他对象的情况下，以动态、透明的方式给单个对象添加职责。

（2）处理那些可以撤消的职责。

（3）当不能采用生成子类的方法进行扩充时。一种情况是，可能有大量独立的扩展，为支持每一种组合将产生大量的 子类，使得子类数目呈爆炸性增长。另一种情况可能是因为类定义被隐藏，或类定义不能用于生成子类。

![dp5](https://github.com/yueyuanyang/knowledge/blob/master/java/designPattern/img/dp5.png)

### 定义
装饰者模式(Decorator ['dekəreitə]  Pattren)，是在不改变原类文件和使用继承的情况下，动态地扩展一个对象的功能，它是通过创建一个包装对象，也就是装饰来包裹真实的对象。

### 内容

使用装饰者模式的时候需要注意以下几点内容：

（1）装饰对象和真实对象有相同的接口。这样客户端对象就可以以和真实对象相同的方式和装饰对象交互。

（2）装饰对象包含一个真实对象的引用。

（3）装饰对象接受所有来自客户端的请求，并把这些请求转发给真实的对象。

（4）装饰对象可以在转发这些请求以前或以后增加一些附加功能。这样就确保了在运行时，不用修改给定的对象的结构就可以在外部增加附加功能。

在面向对象的设计中，通常是通过继承来实现对给定的类的功能扩展，然而，`装饰者模式不需要子类，可以在应用程序运行时动态扩展功能`。

### 1、一般化分析

分析一下染色馒头的制作过程：

（1）需要生产一个正常馒头；

（2）为了节省成本（不使用玉米面），使用染色剂加入到正常馒头中；

（3）通过和面机搅拌，最后生产出“玉米馒头”。

![dp1](https://github.com/yueyuanyang/knowledge/blob/master/java/designPattern/img/dp1.jpg)

### 原始设计
```
/** 
 * 馒头加工接口 
 *  
 * @author 
 *  
 */  
public interface IBread {  
    // 准备材料  
    public void prepair();  
  
    // 和面  
    public void kneadFlour();  
  
    // 蒸馒头  
    public void steamed();  
  
    /** 
     * 加工馒头方法 
     */  
    public void process();  
}

***********************************************************************

/** 
 * 正常馒头的实现 
 *  
 * @author 
 *  
 */  
public class NormalBread implements IBread {  
    // 准备材料  
    public void prepair() {  
        System.out.println("准备面粉、水以及发酵粉...");  
    }  
  
    // 和面  
    public void kneadFlour() {  
        System.out.println("和面...");  
    }  
  
    // 蒸馒头  
    public void steamed() {  
        System.out.println("蒸馒头...香喷喷的馒头出炉了！");  
    }  
  
    /** 
     * 加工馒头方法 
     */  
    public void process() {  
        // 准备材料  
        prepair();  
        // 和面  
        kneadFlour();  
        // 蒸馒头  
        steamed();  
    }  
  
}  


*********************************************************************

/** 
 * 染色的玉米馒头 
 *  
 * @author 
 *  
 */  
public class CornBread extends NormalBread {  
    // 黑心商贩 开始染色了  
    public void paint() {  
        System.out.println("添加柠檬黄的着色剂...");  
    }  
  
    // 重载父类的和面方法  
    @Override  
    public void kneadFlour() {  
        // 在面粉中加入 染色剂 之后才开始和面  
        this.paint();  
        // 和面  
        super.kneadFlour();  
    }  
} 

*********************************************************************

/** 
 * 甜蜜素馒头 
 *  
 * @author 
 *  
 */  
public class SweetBread extends NormalBread {  
    // 黑心商贩 开始添加甜蜜素  
    public void paint() {  
        System.out.println("添加甜蜜素...");  
    }  
  
    // 重载父类的和面方法  
    @Override  
    public void kneadFlour() {  
        // 在面粉中加入 甜蜜素 之后才开始和面  
        this.paint();  
        // 和面  
        super.kneadFlour();  
    }  
}  

```

我们知道，馒头的种类是很多的，现在黑心商贩又想生产“甜玉米馒头"了，怎么办叱？你可能要说“使用继承”不就行了吗？再创建一个馒头，继承正常馒头类。不可否认，这样做是不错的，可以实现要求。然而，反映到类图上是又多了一个子类，如果还有其他的馒头种类，都要继承吗？那样的话类就要爆炸了，庞大的继承类关系图。

**继承方式存在这样两点不利因素**：

（1）父类的依赖程序过高，父类修改会影响到子类的行为。

（2）不能复用已有的类，造成子类过多。

### 装饰者模式重新实现染色馒头的实例

使用装饰者的静态类图，结构如图所示。

![dp2](https://github.com/yueyuanyang/knowledge/blob/master/java/designPattern/img/dp2.jpg)

####  （1）为了装饰正常馒头NormalBread，我们需要一个正常馒头一样的抽象装饰者:AbstractBread,该类和正常馒头类NormalBread一样实现IBread馒头接口，不同的是该抽象类含有一个IBread接口类型的私有属性bread,然后通过构造方法，将外部IBread接口类型对象传入。

```
/** 
 * 抽象装饰者 
 *  
 * @author 
 *  
 */  
public abstract class AbstractBread implements IBread {  
    // 存储传入的IBread对象  
    private final IBread bread;  
  
    public AbstractBread(IBread bread) {  
        this.bread = bread;  
    }  
  
    // 准备材料  
    public void prepair() {  
        this.bread.prepair();  
    }  
  
    // 和面  
    public void kneadFlour() {  
        this.bread.kneadFlour();  
    }  
  
    // 蒸馒头  
    public void steamed() {  
        this.bread.steamed();  
    }  
  
    // 加工馒头方法  
    public void process() {  
        prepair();  
        kneadFlour();  
        steamed();  
  
    }  
}  
```
AbstractBread类满足了装饰者的要求：和真实对象具有相同的接口；包含一个真实对象的引用；接受所有来自客户端的请求，并反这些请求转发给真实的对象；可以增加一些附加功能。

#### （2）创建装饰者。

**A、创建染色剂装饰者----CornDecorator.**

创建染色装饰者"CornDecorator"，继承AbstractBread, 含有修改行为：添加柠檬黄的着色剂。
```
/** 
 * 染色的玉米馒头 
 *  
 * @author 
 *  
 */  
public class CornDecorator extends AbstractBread {  
  
    // 构造方法  
    public CornDecorator(IBread bread) {  
        super(bread);  
    }  
  
    // 黑心商贩 开始染色了  
    public void paint() {  
        System.out.println("添加柠檬黄的着色剂...");  
    }  
  
    // 重载父类的和面方法  
    @Override  
    public void kneadFlour() {  
        // 在面粉中加入 染色剂 之后才开始和面  
        this.paint();  
        // 和面  
        super.kneadFlour();  
    }  
}  

```

这和上面提到的CornBread类的内容是一样的，只是多了一个构造方法。

**B、创建甜蜜互装饰者-----SweetDecorator**

```
/** 
 * 甜蜜素馒头 
 *  
 * @author 
 *  
 */  
public class SweetDecorator extends AbstractBread {  
    // 构造方法  
    public SweetDecorator(IBread bread) {  
        super(bread);  
    }  
  
    // 黑心商贩 开始添加甜蜜素  
    public void paint() {  
        System.out.println("添加甜蜜素...");  
    }  
  
    // 重载父类的和面方法  
    @Override  
    public void kneadFlour() {  
        // 在面粉中加入 甜蜜素 之后才开始和面  
        this.paint();  
        // 和面  
        super.kneadFlour();  
    }  
}  
```

**C、生产甜玉米馒头**。

首先创建一个正常馒头，然后使用甜蜜素装饰馒头，之后再用柠檬黄的着色剂装饰馒头，最后加工馒头。

```
/** 
 * 客户端应用程序 
 *  
 * @author 
 *  
 */  
public class Client {  
  
    /** 
     * @param args 
     */  
    public static void main(String[] args) {  
        // 生产装饰馒头  
        System.out.println("\n====开始装饰馒头！！！");  
        // 创建普通的正常馒头实例  
        // 这是我们需要包装（装饰）的对象实例  
        IBread normalBread = new NormalBread();  
  
        // 下面就开始 对正常馒头进行装饰了！！！  
        // 使用甜蜜素装饰馒头  
        normalBread = new SweetDecorator(normalBread);  
        // 使用柠檬黄的着色剂装饰馒头  
        normalBread = new CornDecorator(normalBread);  
        // 生产馒头信息  
        // 另外一种写法(java IO 流实现)
       //  IBread normalBread = new CornDecorator(new SweetDecorator(new NormalBread())); 
        normalBread.process();  
        System.out.println("====装饰馒头结束！！！");  
  
    } 
}  
```

**运行结果：**

```
====开始装饰馒头！！！  
准备面粉、水以及发酵粉...  
添加柠檬黄的着色剂...  
添加甜蜜素...  
和面...  
蒸馒头...香喷喷的馒头出炉了！  
====装饰馒头结束！！！  
```

可能你会感觉比较奇怪，我们先使用的是甜蜜素，然后使用的是着色剂，应该先打印甜蜜素，然后再打印着色剂才对？不用奇怪，这也不矛盾，因为装饰者相当于对原有对象的包装，这就像一个礼品盒，最里面是一个最普通的纸盒，然后用一般的纸包装起来，最后使用比较漂亮的包装纸包装，我们最先看到的是最外漂亮的包装纸，其次才是里面的一般包装纸。

![dp3](https://github.com/yueyuanyang/knowledge/blob/master/java/designPattern/img/dp3.jpg)

### 3、设计原则

(1)封装变化部分

设计模式是封装变化的最好阐释，无论哪一种设计模式针对的都是软件中存在的“变化”部分，然后
用抽象对这些“变化”的部分进行封装。使用抽象的好处在于为软件的扩展提供了很大的方便性。
在装饰者模式中合理地利用了类继承和组合的方式，非常灵活地表达了对象之间的依赖关系。
装饰者模式应用中“变化”的部分是组件的扩展功能，装饰者和被装饰者完全隔离开来，这样我们就可以任意地改变装饰者和被装饰者。

（2）"开-闭"原则

我们在需要对组件进行扩展、增添新的功能行为时，只需要实现一个特定的装饰者即可，这完全是增量修改，

对原有软件功能结构没有影响，对客户端APP来说也是完全透明的，不必关心内部实现细节。

（3）面向抽象编程，不要面向实现编程

在装饰者模式中，装饰者角色就是抽象类实现，面向抽象编程的好处就在于起到了很好的接口隔离作用。在运用时，

我们具体操作的也是抽象类引用，这些显示了面向抽象编程。

（4）优先使用对象组合，而非类继承。

装饰者模式最成功在于合理地使用了对象组合方式，通过组合灵活地扩展了组件的功能，所有的扩展功能都是通过组合而非继承获得的，

这从根本上决定了是高内聚、低耦合的。

### 4、使用场合

（1）当我们需要为某个现有的对象动态地增加一个新的功能或职责时，可以考虑使用装饰者模式；

（2）当某个对象的职责经常发生变化或者经常需要动态地增加职责，避免为了适应这样的变化而增加继承子类扩展的方式，因为这种方式会造成子类膨胀的速度过快，难以控制，此时可以使用装饰者模式。

客户端不会觉得对象在装饰前和装饰后有什么不同，装饰者模式可以在不创建更多子类的情况下，将对象的功能加以扩展，装饰者模式使用原来被装饰的一个子类实例，

把客户端的调用委派到装饰者。

下面我们来看一下装饰者模式的静态类图.使我们对装饰者模式有一个更加清晰的认识

![dp4](https://github.com/yueyuanyang/knowledge/blob/master/java/designPattern/img/dp4.jpg)

A、被装饰者抽象Component:是一个接口或抽象类，是定义的核心对象。

在装饰者模式中，必然有一个被撮出来最核心、最原始的接口。这个类就是我们需要装饰类的基类。本例中是IBread接口。

B、被装饰者具体实现ConcreteComponent:这是Component类的一个实现类，我们要装饰的就是这个具体的实现类。本例中是NormalBread

C、装饰者Decorator:一般是一个抽象类。它里面有一个指向Component变量的引用。

D、装饰者实现ConcreteDecorator1和ConcreteDecorator2.




