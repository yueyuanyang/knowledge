## 设计模式 —— 工厂模式

### 设计模式之三种工厂模式

### 简单工厂模式

简单工厂模式其实不是一个设计模式，反而比较像一种编程习惯。主要我们定义一个非常简单的类主要负责帮我们生产不同的产品。类图如下：

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

客户端，想要建立一个pizza store，这个pizza store里有一个简单工厂，当我们需要什么pizza的时候，告诉简单工厂，它会为我们生产

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
