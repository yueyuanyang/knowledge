## Java 动态代理

### 作用

主要用来做方法的增强，让你可以在不修改源码的情况下，增强一些方法，在方法执行前后做任何你想做的事情（甚至根本不去执行这个方法），因为在InvocationHandler的invoke方法中，你可以直接获取正在调用方法对应的Method对象，具体应用的话，比如可以添加调用日志，做事务控制等。
动态代理是设计模式当中代理模式的一种。

### 简介

Java编程的目标是实现现实不能完成的，优化现实能够完成的，是一种虚拟技术。生活中的方方面面都可以虚拟到代码中。代理模式所讲的就是现实生活中的这么一个概念：中介。

代理模式的定义：给某一个对象提供一个代理，并由代理对象控制对原对象的引用。

### 代理模式包含如下角色：

- ISubject：抽象主题角色，是一个接口。该接口是对象和它的代理共用的接口。
- RealSubject：真实主题角色，是实现抽象主题接口的类。
- Proxy：代理角色，内部含有对真实对象RealSubject的引用，从而可以操作真实对象。代理对象提供与真实对象相同的接口，以便在任何时刻都能代替真实对象。同时，代理对象可以在执行真实对象操作时，附加其他的操作，相当于对真实对象进行封装。

实现动态代理的关键技术是反射。

## 静态代理
代理模式有几种，虚拟代理，计数代理，远程代理，动态代理。主要分为两类，静态代理和动态代理。静态代理比较简单，是由程序员编写的代理类，并在程序运行前就编译好的，而不是由程序动态产生代理类，这就是所谓的静态。

考虑这样的场景，管理员在网站上执行操作，在生成操作结果的同时需要记录操作日志，这是很常见的。此时就可以使用代理模式，代理模式可以通过聚合和继承两种方式实现：

```
/**方式一：聚合式静态代理 
 * @author Goser    (mailto:goskalrie@163.com) 
 * @Since 2016年9月7日 
 */  
//1.抽象主题接口  
public interface Manager {  
    void doSomething();  
}  
//2.真实主题类  
public class Admin implements Manager {  
    public void doSomething() {  
        System.out.println("Admin do something.");  
    }  
}  
//3.以聚合方式实现的代理主题  
public class AdminPoly implements Manager{  
    private Admin admin;  
     
    public AdminPoly(Admin admin) {  
        super();  
        this.admin = admin;  
    }  
   
    public void doSomething() {  
        System.out.println("Log:admin操作开始");  
        admin.doSomething();  
        System.out.println("Log:admin操作结束");  
    }  
}  
//4.测试代码  
        Admin admin = new Admin();  
        Manager m = new AdminPoly(admin);  
        m.doSomething();  
//方式二：继承式静态代理  
//与上面的方式仅代理类和测试代码不同  
//1.代理类  
public class AdminProxy extends Admin {  
    @Override  
    public void doSomething() {  
        System.out.println("Log:admin操作开始");  
        super.doSomething();  
        System.out.println("Log:admin操作开始");  
    }  
}  
//2.测试代码  
        AdminProxy proxy = new AdminProxy();  
        proxy.doSomething();  
```
