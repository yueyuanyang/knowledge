## 设计模式 —— 构建者(Builder)

我们先看设计模式的分类：


| 范围 | 创建型 | 结构型 | 行为型 |
| - | - | - |  - |
| 类 | Factory Method（工厂方法）| Adapter(类)(适配器) | Interpreter（解释器）<br>Template Method（模版方法）
| 对象 | Abstract Factory（抽象工厂）<br> Builder（建造者）<br>Prototype（原型）<br>Singleton（单例） | Adapter(对象)（适配器）<br>Bridge（桥接）<br>Composite（组合）<br>Decorator（装饰者）<br>Facade（外观）<br>Flyweight（享元）<br>Proxy（代理） | Chain of Responsibility（职责链）<br>Command（命令）<br>Iterator（迭代器）<br>Mediator（中介者）<br>Memento（备忘录）<br>Observer（观察者）<br>State（状体）<br>Strategy（策略）<br>Visitor（访问者）


### 再细点分类

| 范围 | 创建型 | 结构型 | 行为型 |
| - | - | - |  - |
| 对象创建 | Singleton（单例）<br>Prototype（原型）<br>Factory Method（工厂方法）<br>Abstract Factory（抽象工厂）<br>Builder（建造者）| | 
| 接口适配 | Factory Method（工厂方法）| Adapter（适配器）<br>Bridge（桥接）<br>Façade（外观） | Interpreter（解释器）<br>Template Method（模版方法）
| 对象去耦 | | | Mediator（中介者）<br>Observer（观察者）
| 抽象集合 | | Composite（组合） | IIterator（迭代器）
| 行为扩展 | | Decorator（装饰） | Visitor（访问者）<br>Chain of Responsibility（职责链）
| 算法封装 | |  | Template Method（模板方法）<br>Strategy（策略）<br>Command
| 性能与对象访问 | | Flyweight（享元）<br>Proxy（代理） | 
| 对象状态 || Adapter(类)(适配器) | Interpreter（解释器）
| 其他 | || Interpreter（解释器）

