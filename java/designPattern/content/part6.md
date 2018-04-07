## 设计模式 —— 工厂模式

### 设计模式之三种工厂模式

### 简单工厂模式

简单工厂模式其实不是一个设计模式，反而比较像一种编程习惯。主要我们定义一个非常简单的类主要负责帮我们生产不同的产品。
类图如下：

![类图1](https://github.com/yueyuanyang/knowledge/edit/master/java/designPattern/img/factory.png)

客户端通过简单工厂可以生产出具体产品A，具体产品B，具体产品C。

**简单工厂：**

```
public class SimplePizzaFactory {

    /**
     * 根据传入的type参数，返回相应的pizza
     * @param type
     * @return
     */
    public Pizza createPizza(String type) {  //一般这个方法为static
        Pizza pizza = null;

        if (type.equals("cheese")) {
            pizza = new CheesePizza();
        } else if (type.equals("pepperoni")) {
            pizza = new PepperoniPizza();
        } else if (type.equals("clam")) {
            pizza = new ClamPizza();
        } else if (type.equals("veggie")) {
            pizza = new VeggiePizza();
        }
        return pizza;
    }
}

```

**客户端**, 想要建立一个pizza store，这个pizza store里有一个简单工厂，当我们需要什么pizza的时候，告诉简单工厂，它会为我们生产

```
public class PizzaStore {
    //通过组合的使用，加上一个简单工厂SimplePizzaFactory的引用，用于创建pizza
    SimplePizzaFactory factory;
 
    public PizzaStore(SimplePizzaFactory factory) { 
        this.factory = factory;
    }
 
    public Pizza orderPizza(String type) {
        Pizza pizza;
        //调用简单工厂SimplePizzaFactory的createPizza(type)方法创建pizza
        pizza = factory.createPizza(type);
 
        pizza.prepare();
        pizza.bake();
        pizza.cut();
        pizza.box();

        return pizza;
    }
}
```

### 工厂方法模式

这个和简单工厂有区别，简单工厂模式只有一个工厂，工厂方法模式对每一个产品都有相应的工厂。

![类图2](https://github.com/yueyuanyang/knowledge/edit/master/java/designPattern/img/factory1.png)

构建一个工厂的时候，实际上是构建一个具体的子类对象，让子类决定去生产什么产品。

构建`两个`工厂，`一个芝加哥pizza工厂`，`一个纽约pizza工厂`。

- 去生产芝加哥风味的pizza

```
public class ChicagoPizzaStore extends PizzaStore {

    Pizza createPizza(String item) {
            if (item.equals("cheese")) {
                    return new ChicagoStyleCheesePizza();
            } else if (item.equals("veggie")) {
                    return new ChicagoStyleVeggiePizza();
            } else if (item.equals("clam")) {
                    return new ChicagoStyleClamPizza();
            } else if (item.equals("pepperoni")) {
                    return new ChicagoStylePepperoniPizza();
            } else return null;
    }
}
```
- 纽约风味的pizza。

```
public class ChicagoPizzaStore extends PizzaStore {

    Pizza createPizza(String item) {
            if (item.equals("cheese")) {
                    return new ChicagoStyleCheesePizza();
            } else if (item.equals("veggie")) {
                    return new ChicagoStyleVeggiePizza();
            } else if (item.equals("clam")) {
                    return new ChicagoStyleClamPizza();
            } else if (item.equals("pepperoni")) {
                    return new ChicagoStylePepperoniPizza();
            } else return null;
    }
}

```

**客户端：**

```
public class PizzaTestDrive {
 
    public static void main(String[] args) {
        PizzaStore nyStore = new NYPizzaStore();
        PizzaStore chicagoStore = new ChicagoPizzaStore();
 
        Pizza pizza = nyStore.orderPizza("cheese");
        System.out.println("Ethan ordered a " + pizza.getName() + "\n");
 
        pizza = chicagoStore.orderPizza("cheese");
        System.out.println("Joel ordered a " + pizza.getName() + "\n");

        pizza = nyStore.orderPizza("clam");
        System.out.println("Ethan ordered a " + pizza.getName() + "\n");
 
        pizza = chicagoStore.orderPizza("clam");
        System.out.println("Joel ordered a " + pizza.getName() + "\n");
    }
}

```

### 抽象工厂模式：

**定义**：为创建一组相关或相互依赖的对象提供一个接口，而且无需指定他们的具体类。

**类型**：创建类模式

**类图**：

![类图3](https://github.com/yueyuanyang/knowledge/edit/master/java/designPattern/img/factory2.png)

**抽象工厂模式与工厂方法模式的区别**

抽象工厂模式是工厂方法模式的升级版本，他用来创建一组相关或者相互依赖的对象。他与工厂方法模式的区别就在于，工厂方法模式针对的`是一个产品等级结构；而抽象工厂模式则是针对的多个产品等级结构`。在编程中，通常一个产品结构，表现为一个接口或者抽象类，也就是说，工厂方法模式提供的所有产品都是衍生自同一个接口或抽象类，而抽象工厂模式所提供的产品则是衍生自不同的接口或抽象类。

在抽象工厂模式中，有一个产品族的概念：`所谓的产品族，是指位于不同产品等级结构中功能相关联的产品组成的家族`。抽象工厂模式所提供的一系列产品就组成一个产品族；而工厂方法提供的一系列产品称为一个等级结构。我们依然拿生产汽车的例子来说明他们之间的区别。

![类图4](https://github.com/yueyuanyang/knowledge/edit/master/java/designPattern/img/factory3.png)

在上面的类图中，两厢车和三厢车称为两个不同的等级结构；而2.0排量车和2.4排量车则称为两个不同的产品族。再具体一点，2.0排量两厢车和2.4排量两厢车属于同一个等级结构，2.0排量三厢车和2.4排量三厢车属于另一个等级结构；而2.0排量两厢车和2.0排量三厢车属于同一个产品族，2.4排量两厢车和2.4排量三厢车属于另一个产品族。

明白了等级结构和产品族的概念，就理解工厂方法模式和抽象工厂模式的区别了，如果工厂的产品全部属于同一个等级结构，则属于工厂方法模式；如果工厂的产品来自多个等级结构，则属于抽象工厂模式。在本例中，如果一个工厂模式提供2.0排量两厢车和2.4排量两厢车，那么他属于工厂方法模式；如果一个工厂模式是提供2.4排量两厢车和2.4排量三厢车两个产品，那么这个工厂模式就是抽象工厂模式，因为他提供的产品是分属两个不同的等级结构。当然，如果一个工厂提供全部四种车型的产品，因为产品分属两个等级结构，他当然也属于抽象工厂模式了。

```
interface IProduct1 {
    public void show();
}
interface IProduct2 {
    public void show();
}

class Product1 implements IProduct1 {
    public void show() {
        System.out.println("这是1型产品");
    }
}
class Product2 implements IProduct2 {
    public void show() {
        System.out.println("这是2型产品");
    }
}

interface IFactory {
    public IProduct1 createProduct1();
    public IProduct2 createProduct2();
}
class Factory implements IFactory{
    public IProduct1 createProduct1() {
        return new Product1();
    }
    public IProduct2 createProduct2() {
        return new Product2();
    }
}

public class Client {
    public static void main(String[] args){
        IFactory factory = new Factory();
        factory.createProduct1().show();
        factory.createProduct2().show();
    }
}

```




