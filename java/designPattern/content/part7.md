## Proxy Pattern（代理模式）（建议使用 cglib 动态代理） 

### 作用

主要用来做方法的**增强**，让你可以**在不修改源码的情况下，增强一些方法，在方法执行前后做任何你想做的事情（甚至根本不去执行这个方法）**，因为在InvocationHandler的invoke方法中，你可以直接获取正在调用方法对应的Method对象，具体应用的话，比如可以添加调用日志，做事务控制等。
动态代理是设计模式当中代理模式的一种。

### 简介

Java编程的目标是实现现实不能完成的，优化现实能够完成的，是一种虚拟技术。生活中的方方面面都可以虚拟到代码中。代理模式所讲的就是现实生活中的这么一个概念：中介。

代理模式的定义：给某一个对象提供一个代理，并由代理对象控制对原对象的引用。

### 代理模式包含如下角色：

- ISubject：抽象主题角色，是一个接口。该接口是对象和它的代理共用的接口。
- RealSubject：真实主题角色，是实现抽象主题接口的类。
- Proxy：代理角色，内部含有对真实对象RealSubject的引用，从而可以操作真实对象。代理对象提供与真实对象相同的接口，以便在任何时刻都能代替真实对象。同时，代理对象可以在执行真实对象操作时，附加其他的操作，相当于对真实对象进行封装。

**实现动态代理的关键技术是反射**。

## 动态代理

有两种方法 一种是**java反射机制**，另一种效率比较好，**采用cglib**，后者也是spring AOP采用的技术。

### java反射机制 实现动态代理

```
接口
public interface StudentDao {  
    public void saveStudent();  
    public void queryStudent();  
}  

被代理类
public class StudentDaoImpl implements StudentDao {  
  
    @Override  
    public void saveStudent() {  
        System.out.println("保存学生资料。。。。");  
    }  
  
    @Override  
    public void queryStudent() {  
        System.out.println("查询学生资料。。。。");  
    }  
  
}  

代理类
public class DAOProxy implements InvocationHandler {  
  
    private Object originalObject;  
   
    public Object bind(Object obj) {  
        this.originalObject = obj;  
        return Proxy.newProxyInstance(obj.getClass().getClassLoader(), obj  
                .getClass().getInterfaces(), this);  
    }  
  
    void preMethod() {  
        System.out.println("----执行方法之前----");  
    }  
  
    void afterMethod() {  
        System.out.println("----执行方法之后----");  
    }  
  
    @Override  
    public Object invoke(Object proxy, Method method, Object[] args)  
            throws Throwable {  
        Object result = null;  
        preMethod();  
        result = method.invoke(this.originalObject, args);  
        afterMethod();  
        return result;  
    }  
  
}  

测试类
public class TestDaoProxy extends TestCase {  
    public void testDaoProxy(){  
        StudentDao studentDao = new StudentDaoImpl();  
        DAOProxy daoProxy=new DAOProxy();  
        studentDao = (StudentDao)daoProxy.bind(studentDao);  
        studentDao.queryStudent();  
    }  
} 



```

### cglib 动态代理
**maven**
```
<dependency>
    <groupId>cglib</groupId>
    <artifactId>cglib</artifactId>
    <version>3.2.6</version>
</dependency>
<dependency>
    <groupId>asm</groupId>
    <artifactId>asm</artifactId>
    <version>3.3.1</version>
</dependency>
```

**样例**

```
被代理类
public class StudentDao {  
    public void saveStudent() {  
        System.out.println("保存学生资料。。。。");  
    }  
  
    public void queryStudent() {  
        System.out.println("查询学生资料。。。。");  
    }  
}  

代理类
public class DAOCglibProxy implements MethodInterceptor {  
  
    private Object originalObject;  
  
    public Object bind(Object obj) {  
        this.originalObject = obj;  
        Enhancer enhancer = new Enhancer();  
        enhancer.setSuperclass(obj.getClass());  
        enhancer.setCallback(this);  
        return enhancer.create();  
    }  
  
  // 增强方法(原方法前)
    void preMethod() {  
        System.out.println("----执行方法之前----");  
    }  
    
  // 增强方法(原方法后)
    void afterMethod() {  
        System.out.println("----执行方法之后----");  
    }  
  
    @Override  
    public Object intercept(Object obj, Method method, Object[] args,  
            MethodProxy proxy) throws Throwable {  
        preMethod();  
        Object result = proxy.invokeSuper(obj, args);  
        afterMethod();  
        return result;  
    }  
  
}  

测试类
public void testDAOCglibProxy() {  
    StudentDao studentsDao = new StudentDao();  
    DAOCglibProxy proxy = new DAOCglibProxy();  
    studentsDao = (StudentDao) proxy.bind(studentsDao);  
    studentsDao.queryStudent();  
}  
    
```
**优点：打破了动态代理的时候，代理对象只能是接口，代理对象不能是类。** 

cglib实际上是通过继承出一个子类实现的代理。可以把Enhancer看做那个子类的生成器 ，所以在bind方法中，指定父类，然后绑定好代理对象就可以了

指定父类：enhancer.setSuperclass(obj.getClass());

绑定代理方法：enhancer.setCallback(this);

然后返回创建好的已经加了代理的目标对象：return enhancer.create();

在测试类中使用的时候很简单，只要绑定好，然后执行就行。

可见还是cglib这个方法更方便，而且据说效率最高。所以spring采用之。

**cglib和java反射的区别**

1) java反射 必须定义一个接口
2) 代理对象可以为类

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
