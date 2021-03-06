## 设计模式 —— 单例模式

### 单例模式定义：

单例模式确保某个类只有一个实例，而且自行实例化并向整个系统提供这个实例。在计算机系统中，线程池、缓存、日志对象、对话框、打印机、显卡的驱动程序对象常被设计成单例。这些应用都或多或少具有资源管理器的功能。每台计算机可以有若干个打印机，但只能有一个Printer Spooler，以避免两个打印作业同时输出到打印机中。每台计算机可以有若干通信端口，系统应当集中管理这些通信端口，以避免一个通信端口同时被两个请求同时调用。总之，选择单例模式就是为了避免不一致状态，避免政出多头。

### 单例模式特点：

1、单例类只能有一个实例。

2、单例类必须自己创建自己的唯一实例。

3、单例类必须给所有其他对象提供这一实例。

单例模式保证了全局对象的唯一性，比如系统启动读取配置文件就需要单例保证配置的一致性。

### 线程安全的问题

一方面在获取单例的时候，要保证不能产生多个实例对象，后面会详细讲到五种实现方式；

另一方面，在使用单例对象的时候，要注意单例对象内的实例变量是会被多线程共享的，推荐使用无状态的对象，不会因为多个线程的交替调度而破坏自身状态导致线程安全问题，比如我们常用的VO，DTO等（局部变量是在用户栈中的，而且用户栈本身就是线程私有的内存区域，所以不存在线程安全问题）。

### 单例模式的选择

还记得我们最早使用的MVC框架Struts1中的action就是单例模式的，而到了Struts2就使用了多例。在Struts1里，当有多个请求访问，每个都会分配一个新线程，在这些线程，操作的都是同一个action对象，每个用户的数据都是不同的，而action却只有一个。到了Struts2， action对象为每一个请求产生一个实例，并不会带来线程安全问题（实际上servlet容器给每个请求产生许多可丢弃的对象，但是并没有影响到性能和垃圾回收问题，有时间会做下研究）。

### 实现单例模式的方式

**1.饿汉式单例（立即加载方式）**

```
// 饿汉式单例
public class Singleton {
    // 私有构造
    private Singleton() {}

    private static Singleton single = new Singleton1();

    // 静态工厂方法
    public static Singleton getInstance() {
        return single;
    }
}

```

饿汉式单例在类加载初始化时就创建好一个静态的对象供外部使用，除非系统重启，这个对象不会改变，所以本身就是线程安全的。Singleton通过将构造方法限定为private避免了类在外部被实例化，在同一个虚拟机范围内，Singleton的唯一实例只能通过getInstance()方法访问。（事实上，通过Java反射机制是能够实例化构造方法为private的类的，那基本上会使所有的Java单例实现失效。此问题在此处不做讨论，姑且闭着眼就认为反射机制不存在。）

**2.懒汉式单例（延迟加载方式）**

```
// 懒汉式单例
public class Singleton {

    // 私有构造
    private Singleton() {}

    private static Singleton single = null;

    public static Singleton getInstance() {
        if(single == null){
            single = new Singleton();
        }
        return single;
    }
}

```

该示例虽然用延迟加载方式实现了懒汉式单例，但在多线程环境下会产生多个single对象，如何改造请看以下方式:

**使用synchronized同步锁**

```
public class Singleton {

    // 私有构造
    private Singleton3() {}

    private static Singleton3 single = null;

    public static Singleton3 getInstance() {
        
        // 等同于 synchronized public static Singleton3 getInstance()
        synchronized(Singleton3.class){
          // 注意：里面的判断是一定要加的，否则出现线程安全问题
            if(single == null){
                single = new Singleton3();
            }
        }
        return single;
    }
}


```

在方法上加synchronized同步锁或是用同步代码块对类加同步锁，此种方式虽然解决了多个实例对象问题，但是该方式运行效率却很低下，下一个线程想要获取对象，就必须等待上一个线程释放锁之后，才可以继续运行。

```
public class Singleton {
    // 私有构造
    private Singleton() {}

    // 单例模式: volatile + 双重检测-> 禁止指令重排
    private volatile static Singleton single = null;

    // 双重检查
    public static Singleton getInstance() {
        if (single == null) { // 双重检测机制
            synchronized (Singleton.class) { // 同步所(修饰类)
                if (single == null) {
                    single = new Singleton4();
                }
            }
        }
        return single;
    }
}

```

**推荐使用**
```
/**
 * 枚举模式：最安全的
 */
public class SingletonExample {

    private SingletonExample() {

    }

    public static  SingletonExample getInstance() {
        return Singleton.INSTANCE.getSingleton();
    }
    private enum Singleton {
        INSTANCE;
        private SingletonExample singleton;
        Singleton() {
           singleton = new SingletonExample();
        }
        public  SingletonExample getSingleton() {
            return singleton;
        }
    }
}
```
使用双重检查进一步做了优化，可以避免整个方法被锁，只对需要锁的代码部分加锁，可以提高执行效率。**推荐使用**

