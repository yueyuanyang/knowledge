## 装饰者模式

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



