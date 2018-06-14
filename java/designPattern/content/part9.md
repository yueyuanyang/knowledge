## Java设计模式----责任链模式

### 简述：

前端时间再看一些类库的源码，发现责任链模式的强大之处，尤其是和建造者模式的结合后强大的动态可扩展性更是牛逼的一塌糊涂。接下来赶紧了解一下吧！

**我们先来了解一下什么是责任链模式**：

职责链模式（Chain of Responsibility）：使多个对象都有机会处理同一个请求，从而避免请求的发送者和接收者之间的耦合关系。将这些对象连成一条链，并沿着这条链传递该请求，直到有一个对象处理它为止。

**应用场景**：

为完成同一个请求，如果存在多个请求处理器以及未知请求处理器个数或者请求处理器可动态配置的情况下，可以考虑使用责任链模式。如OKHttp的拦截器就是使用的责任链模式。

### 实例UML图：

### 实例：

#### 1、实例场景

在公司内部员工请假一般情况是这样的：员工在OA系统中提交一封请假邮件，该邮件会自动转发到你的直接上级领导邮箱里，如果你的请假的情况特殊的话，该邮件也会转发到你上级的上级的邮箱，根据请假的情况天数多少，系统会自动转发相应的责任人的邮箱。我们就以这样一种场景为例完成一个责任链模式的代码。为了更清晰的描述这种场景我们规定如下：

- GroupLeader（组长 ）：他能批准的假期为2天，如果请假天数超过2天就将请假邮件自动转发到组长和经理邮箱。
- Manager（经理）：他能批准的假期为4天以内，如果请假天数大于4天将该邮件转发到自动转发到组长、经理和部门领导的邮箱。
- DepartmentHeader（部门领导）：他能批准的假期为7天以内，如果大于7天就只批准7天。

#### 2、实例代码

我们清楚了上面的场景以后就开始定义模型：

根据面向对象的思想我们得定义需要用到的对象。OK，为了更加清楚的说明“责任链模式的可扩展性”问题我这里采用了建造者模式构造Request对象，“请假”对象Request如下：
```
/**  
 * 类描述：请假对象  
 *  
 * @author lzy  
 */  
public class Request {  
     private String name;  
  
     private String reason;  
  
     private int days;  
  
     private String groupLeaderInfo;  
  
     private String managerInfo;  
  
     private String departmentHeaderInfo;  
  
     private String customInfo;  
  
     public Request(Builder builder) {  
          super();  
          this.name = builder.name;  
          this.reason = builder.reason;  
          this.days = builder.days;  
          this.groupLeaderInfo = builder.groupLeaderInfo;  
          this.managerInfo = builder.managerInfo;  
          this.departmentHeaderInfo = builder.departmentHeaderInfo;  
          this.customInfo = builder.customInfo;  
     }  
  
     public static class Builder {  
          public String name;  
  
          public String reason;  
  
          public int days;  
  
          public String groupLeaderInfo;  
  
          public String managerInfo;  
  
          public String departmentHeaderInfo;  
  
          public String customInfo;  
  
          public Builder() {  
  
          }  
  
          public Builder setName(String name) {  
              this.name = name;  
              return this;  
          }  
  
          public Builder setReason(String reason) {  
              this.reason = reason;  
              return this;  
          }  
  
          public Builder setDays(int days) {  
              this.days = days;  
              return this;  
          }  
  
          public Builder setGroupLeaderInfo(String groupLeaderInfo) {  
              this.groupLeaderInfo = groupLeaderInfo;  
              return this;  
          }  
  
          public Builder setManagerInfo(String managerInfo) {  
              this.managerInfo = managerInfo;  
              return this;  
          }  
  
          public Builder setDepartmentHeaderInfo(String departmentHeaderInfo) {  
              this.departmentHeaderInfo = departmentHeaderInfo;  
              return this;  
          }  
  
          public Builder setCustomInfo(String customInfo) {  
              this.customInfo = customInfo;  
              return this;  
          }  
  
          public Builder newRequest(Request request) {  
              this.name = request.name;  
              this.days = request.days;  
              this.reason = request.reason;  
              if (request.groupLeaderInfo != null  
                        && !request.groupLeaderInfo.equals("")) {  
                   this.groupLeaderInfo = request.groupLeaderInfo;  
              }  
  
              if (request.managerInfo != null && !request.managerInfo.equals("")) {  
                   this.managerInfo = request.managerInfo;  
              }  
  
              if (request.departmentHeaderInfo != null  
                        && !request.departmentHeaderInfo.equals("")) {  
                   this.departmentHeaderInfo = request.departmentHeaderInfo;  
              }  
  
              if (request.customInfo != null && !request.customInfo.equals("")) {  
                   this.customInfo = request.customInfo;  
              }  
                
  
              return this;  
          }  
  
          public Request build() {  
              return new Request(this);  
          }  
     }  
  
     public String name() {  
          return name;  
     }  
  
     public String reason() {  
          return reason;  
     }  
  
     public int days() {  
          return days;  
     }  
  
     public String groupLeaderInfo() {  
          return groupLeaderInfo;  
     }  
  
     public String managerInfo() {  
          return managerInfo;  
     }  
  
     public String departmentHeaderInfo() {  
          return departmentHeaderInfo;  
     }  
  
     public String customInfo() {  
          return customInfo;  
     }  
  
     @Override  
     public String toString() {  
          return "Request [name=" + name + ", reason=" + reason + ", days="  
                   + days + ",customInfo=" + customInfo + ", groupLeaderInfo="  
                   + groupLeaderInfo + ", managerInfo=" + managerInfo  
                   + ", departmentHeaderInfo=" + departmentHeaderInfo + "]";  
     }  
  
}  
```

接下来再定义"批准结果" 对象Result：

```
/**  
 * 类描述：结果对象  
 *  
 * @author lzy  
 *  
 */  
public class Result {  
     public boolean isRatify;  
     public String info;  
  
     public Result() {  
  
     }  
  
     public Result(boolean isRatify, String info) {  
          super();  
          this.isRatify = isRatify;  
          this.info = info;  
     }  
  
     public boolean isRatify() {  
          return isRatify;  
     }  
  
     public void setRatify(boolean isRatify) {  
          this.isRatify = isRatify;  
     }  
  
     public String getReason() {  
          return info;  
     }  
  
     public void setReason(String info) {  
          this.info = info;  
     }  
  
     @Override  
     public String toString() {  
          return "Result [isRatify=" + isRatify + ", info=" + info + "]";  
     }  
}
```

2) 我们接下来再来定义一个接口，这个接口用于处理Request和获取请求结果Result。

```
/**  
 * 接口描述：处理请求  
 *  
 * @author lzy  
 *  
 */  
public interface Ratify {  
     // 处理请求  
     public Result deal(Chain chain);  
  
     /**  
      * 接口描述：对request和Result封装，用来转发  
      */  
     interface Chain {  
          // 获取当前request  
          Request request();  
  
          // 转发request  
          Result proceed(Request request);  
     }  
}

```

看到上面的接口，可能会有人迷惑：在接口Ratify中为什么又定义一个Chain接口呢？其实这个接口是单独定义还是内部接口没有太大关系，但是考虑到Chain接口与Ratify接口的关系为提高内聚性就定义为内部接口了。定义Ratify接口是为了处理Request那为什么还要定义Chain接口呢？这正是责任链接口的精髓之处：转发功能及可动态扩展“责任人”，这个接口中定义了两个方法一个是request（）就是为了获取request，如果当前Ratify的实现类获取到request之后发现自己不能处理或者说自己只能处理部分请求，那么他将自己的那部分能处理的就处理掉，然后重新构建一个或者直接转发Request给下一个责任人。可能这点说的不容易理解，我举个例子，在Android与后台交互中如果使用了Http协议，当然我们可能使用各种Http框架如HttpClient、OKHttp等，我们只需要发送要请求的参数就直接等待结果了，这个过程中你可能并没有构建请求头，那么框架帮你把这部分工作给做了，它做的工程中如果使用了责任链模式的话，它肯定会将Request进行包装（也就是添加请求头）成新的Request，我们姑且加他为Request1，如果你又希望Http做本地缓存，那么Request1又会被转发到并且重新进一步包装为Request2。总之Chain这个接口就是起到对Request进行重新包装的并将包装后的Request进行下一步转发的作用。如果还不是很明白也没关系，本实例会演示这一功能机制。

3) 上面说Chain是用来对Request重新包装以及将包装后的Request进行下一步转发用的，那我们就具体实现一下：

```
/**  
 * 类描述：实现Chain的真正的包装Request和转发功能  
 *  
 * @author lzy  
 *  
 */  
public class RealChain implements Chain {  
     public Request request;  
     public List<Ratify> ratifyList;  
     public int index;  
  
     /**  
      * 构造方法  
      *  
      * @param ratifyList  
      *            Ratify接口的实现类集合  
      * @param request  
      *            具体的请求Request实例  
      * @param index  
      *            已经处理过该request的责任人数量  
      */  
     public RealChain(List<Ratify> ratifyList, Request request, int index) {  
          this.ratifyList = ratifyList;  
          this.request = request;  
          this.index = index;  
     }  
  
     /**  
      * 方法描述：具体转发功能  
      */  
     @Override  
     public Result proceed(Request request) {  
          Result proceed = null;  
          if (ratifyList.size() > index) {  
              RealChain realChain = new RealChain(ratifyList, request, index + 1);  
              Ratify ratify = ratifyList.get(index);  
              proceed = ratify.deal(realChain);  
          }  
  
          return proceed;  
     }  
  
     /**  
      * 方法描述：返回当前Request对象或者返回当前进行包装后的Request对象  
      */  
     @Override  
     public Request request() {  
          return request;  
     }  
}  
```
 
 4) 经过上面几步我们已经完成了责任链模式的核心功能，接下来我们定义几个相关责任对象：GroupLeader、Manager和DepartmentHeader，并让他们实现Ratify接口
 
 ```
 /**  
 * 组长  
 *  
 * @author lzy  
 *  
 */  
public class GroupLeader implements Ratify {  
  
     @Override  
     public Result deal(Chain chain) {  
          Request request = chain.request();  
          System.out.println("GroupLeader=====>request:" + request.toString());  
  
          if (request.days() > 1) {  
              // 包装新的Request对象  
              Request newRequest = new Request.Builder().newRequest(request)  
                        .setManagerInfo(request.name() + "平时表现不错，而且现在项目也不忙")  
                        .build();  
              return chain.proceed(newRequest);  
          }  
  
          return new Result(true, "GroupLeader：早去早回");  
     }  
}  
  
  
/**  
 * 经理  
 *  
 * @author lzy  
 *  
 */  
public class Manager implements Ratify {  
  
     @Override  
     public Result deal(Chain chain) {  
          Request request = chain.request();  
          System.out.println("Manager=====>request:" + request.toString());  
          if (request.days() > 3) {  
              // 构建新的Request  
              Request newRequest = new Request.Builder().newRequest(request)  
                        .setManagerInfo(request.name() + "每月的KPI考核还不错，可以批准")  
                        .build();  
              return chain.proceed(newRequest);  
  
          }  
          return new Result(true, "Manager：早点把事情办完，项目离不开你");  
     }  
  
}  
  
/**  
 * 部门领导  
 *  
 * @author lzy  
 *  
 */  
public class DepartmentHeader implements Ratify {  
  
     @Override  
     public Result deal(Chain chain) {  
          Request request = chain.request();  
          System.out.println("DepartmentHeader=====>request:"  
                   + request.toString());  
          if (request.days() > 7) {  
              return new Result(false, "你这个完全没必要");  
          }  
          return new Result(true, "DepartmentHeader：不要着急，把事情处理完再回来！");  
     }  
```

  到此，责任链模式的一个Demo就算是完成了，但为了方便调用，我们在写一个该责任链模式的客户端工具类ChainOfResponsibilityClient 如下
  ```
  /**  
 * 类描述：责任链模模式工具类  
 *  
 * @author lzy  
 *  
 */  
public class ChainOfResponsibilityClient {  
private ArrayList<Ratify> ratifies;  
  
     public ChainOfResponsibilityClient() {  
          ratifies = new ArrayList<Ratify>();  
     }  
  
     /**  
      * 方法描述：为了展示“责任链模式”的真正的迷人之处（可扩展性），在这里构造该方法以便添加自定义的“责任人”  
      *  
      * @param ratify  
      */  
     public void addRatifys(Ratify ratify) {  
          ratifies.add(ratify);  
     }  
  
     /**  
      * 方法描述：执行请求  
      *  
      * @param request  
      * @return  
      */  
     public Result execute(Request request) {  
          ArrayList<Ratify> arrayList = new ArrayList<Ratify>();  
          arrayList.addAll(ratifies);  
          arrayList.add(new GroupLeader());  
          arrayList.add(new Manager());  
          arrayList.add(new DepartmentHeader());  
  
          RealChain realChain = new RealChain(this, arrayList, request, 0);  
          return realChain.proceed(request);  
     }  
      
}  
```

OK，我们测试一下见证奇迹吧：

```
/**  
 * 类描述：责任链模式测试类  
 *  
 * @author lzy  
 *  
 */  
public class Main {  
  
     public static void main(String[] args) {  
  
          Request request = new Request.Builder().setName("张三").setDays(5)  
                   .setReason("事假").build();  
          ChainOfResponsibilityClient client = new ChainOfResponsibilityClient();  
          Result result = client.execute(request);  
  
          System.out.println("结果：" + result.toString());  
     }   
}  
```

这个请求是张三请事假5天，按照我们的约定应该请求会到达部门领导手里，且他看到请求的样式为：“ [name=张三, reason=事假, days=5customInfo=null, groupLeaderInfo=张三平时表现不错，而且现在项目也不忙, managerInfo=张三每月的KPI考核还不错，可以批准, departmentHeaderInfo=null]”我们看一下打印的日志
 
```
GroupLeader=====>request:Request [name=张三, reason=事假, days=5customInfo=null, groupLeaderInfo=null, managerInfo=null, departmentHeaderInfo=null]  
Manager=====>request:Request [name=张三, reason=事假, days=5customInfo=null, groupLeaderInfo=张三平时表现不错，而且现在项目也不忙, managerInfo=null, departmentHeaderInfo=null]  
DepartmentHeader=====>request:Request [name=张三, reason=事假, days=5customInfo=null, groupLeaderInfo=张三平时表现不错，而且现在项目也不忙, managerInfo=张三每月的KPI考核还不错，可以批准, departmentHeaderInfo=null]  
结果：Result [isRatify=true, info=DepartmentHeader：不要着急，把事情处理完再回来!]  
```
OK，和预期一样完美。刚开始就提到这个责任链模式是可以“动态扩展的”，我们验证一下，首先自定义一个“责任人”（其实也可以叫拦截器）：

```
/**  
 * 类描述：自定义“责任人”  
 *  
 * @author lzy  
 *  
 */  
public class CustomInterceptor implements Ratify {  
  
     @Override  
     public Result deal(Chain chain) {  
          Request request = chain.request();  
          System.out.println("CustomInterceptor=>" + request.toString());  
          String reason = request.reason();  
          if (reason != null && reason.equals("事假")) {  
              Request newRequest = new Request.Builder().newRequest(request)  
                        .setCustomInfo(request.name() + "请的是事假，而且很着急，请领导重视一下")  
                        .build();  
              System.out.println("CustomInterceptor=>转发请求");  
              return chain.proceed(newRequest);  
          }  
          return new Result(true, "同意请假");  
     }
}  
```

然后在测试类Main.java中调用如下：

```
/**  
 * 类描述：责任链模式测试类  
 *  
 * @author lzy  
 *  
 */  
public class Main {  
  
     public static void main(String[] args) {  
  
          Request request = new Request.Builder().setName("张三").setDays(5)  
                   .setReason("事假").build();  
          ChainOfResponsibilityClient client = new ChainOfResponsibilityClient();  
          client.addRatifys(new CustomInterceptor());  
          Result result = client.execute(request);  
  
          System.out.println("结果：" + result.toString());  
     }  
  
} 
```
 
    
    



