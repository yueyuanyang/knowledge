
## 《Effective Java》读书笔记

---

### 引言

**1 代码应该被重用 ，而不是被拷贝。**

**2 错误应该尽早被检测出来，最好是在编译时刻。**

**3 接口、类、数组是引用类型（对象）， 基本类型不是**

---

### 第二章 创建和销毁对象

#### 1. 考虑用静态工厂方法代替构造器

**优势：** 

1 有名称（多个 相同签名 的构造器） 

2 不必每次调用它们都创建一个新对象。（可控） 

3 可以返回原返回类型的任何子类型的对象。（灵活，可返回一个接口类型，强迫客户端面向接口编程）

4 静态工厂方法可以利用 类型推导 简化 参数化类型实例 的创建。 

**缺点：**

1 类不含public protected的构造器，不能被子类化。 
2 Javadoc API 文档中，没有特殊标明 静态工厂方法。 不方便查阅。 

**一些惯用名称：**

1. valueOf，实际上是类型转换方法 
2. of，valueOf的简洁替代。 
3. getInstance。 
4. newInstance。与3相比，确保返回的每个实例都与其他的所有实例不同。 
5. getXXXX。 
6. newXXXX。

```
class Child{
    int age = 10;
    int weight = 30;
    public static Child newChild(int age, int weight) {
        Child child = new Child();
        child.weight = weight;
        child.age = age;
        return child;
    }
    
    // 构造函数参数只识别类型，忽略参数名称，使用静态方法可以区分传入的参数
    
    public static Child newChildWithWeight(int weight) {
        Child child = new Child();
        child.weight = weight;
        return child;
    }
    public static Child newChildWithAge(int age) {
        Child child = new Child();
        child.age = age;
        return child;
    }
}

/// 可以减少对外暴露的属性
// 指定类型的传入
class Player {
    public static final int TYPE_RUNNER = 1;
    public static final int TYPE_SWIMMER = 2;
    public static final int TYPE_RACER = 3;
    int type;

    private Player(int type) {
        this.type = type;
    }

    public static Player newRunner() {
        return new Player(TYPE_RUNNER);
    }
    public static Player newSwimmer() {
        return new Player(TYPE_SWIMMER);
    }
    public static Player newRacer() {
        return new Player(TYPE_RACER);
    }
}

```

**总结：**

和构造器比，各有好处。优先使用静态工厂。

(重用对象的好处 可以用 == 替代 equals，性能更好。静态工厂参考Boolean.valueOf().)

---

### 2. 遇到多个构造器参数时，要考虑用Builder

静态工厂和构造器有个共同的局限性：它们都不能很好地扩展到大量的可选参数。 

Builder模式，既能保证像重叠构造器模式那样的安全性，也能保证像JavaBeans模式那么好的可读性。还可以有多个可变参数。

Builder模式构建时，被构建类的field一般都是final。

`如类的构造器或者静态工厂中具有多个参数，特别大多数参数时可选的时候，使用Builder模式。`

[Builder模式 请参考](https://github.com/yueyuanyang/knowledge/blob/master/java/designPattern/content/part2.md)

（最好一开始就用Builder，否则兼容以前的构造函数和静态工厂很烦。）


### 3.用私有构造器或者枚举类型强化Singleton属性

这种方法更加简洁，无偿的提供了序列化机制，绝对**防止多次实例化**（由Enum保证），即使是在面对复杂的序列化或者反序列化或者反射攻击的时候也可以保证唯一。这已成为实现 Singleton的最佳方法。

```
// 写成枚举类型
public enum EnumSingleton {
    INSTANCE;

    EnumSingleton() {
        System.out.println("我被创建了");
    }

    public void metho1() {
        System.out.println("单例的方法");
    }
}

```

### 4. 通过私有构造器强化不可实例化的能力

副作用是使得一个类不能被子类化。可用作常量定义工具类。

**场景**：

在**创建工具类**的时候，大部分是无需实例化的，实例化对它们没有意义。在这种情况下，创建的类，要确保它是不可以实例化的。
 
**存在问题**：

在创建不可实例化的类时，虽然没有定义构造器。但是，客户端在使用该类的时候，依然可以实例化它。客户端，可以继承该类，通过实例化其子类来实现实例化；客户端可以调用默认的构造器来实例化该类。
 
要避免这个问题，使用的方式是，定义一个私有的构造器：

```
public class UtilityClass {
    // Suppress default constructor for noninstantiability
    private UtilityClass() {
        throw new AssertionError();
    }
}

```

### 5. 避免创建不必要的对象

1) 一般来说，最好能重用对象而不是在每次需要的时候就创建一个相同功能的新对象。

**demo1**：
```
String s = new String("hello world");
```
需要改成
```
String s = "hello world";
```
**因为**第一种写法，每一次调用的时候都会创建一个新的String实例。但是第二种写法，在同一台虚拟机中运行的代码，只要他们包含相同的字符串字面常量，就会被重用。

2) 除了重用不可变的对象之外，也可以重用那些已经不会被修改的可变对象。

**错误方式**：

isBabyBoomer每次被调用，都会新建一个Calendar、一个TimeZone和两个Date实例。
```
import java.util.*;  
  
public class Person {  
    private final Date birthDate;  
  
    public Person(Date birthDate) {  
        // Defensive copy - see Item 39  
        this.birthDate = new Date(birthDate.getTime());  
    }  
  
    // Other fields, methods omitted  
  
    // DON'T DO THIS!  
    public boolean isBabyBoomer() {  
        // Unnecessary allocation of expensive object  
        Calendar gmtCal =  
            Calendar.getInstance(TimeZone.getTimeZone("GMT"));  
        gmtCal.set(1946, Calendar.JANUARY, 1, 0, 0, 0);  
        Date boomStart = gmtCal.getTime();  
        gmtCal.set(1965, Calendar.JANUARY, 1, 0, 0, 0);  
        Date boomEnd = gmtCal.getTime();  
        return birthDate.compareTo(boomStart) >= 0 &&  
               birthDate.compareTo(boomEnd)   <  0;  
    }  
}

```
**正确方式**:

```
import java.util.*;  
  
class Person {  
    private final Date birthDate;  
  
    public Person(Date birthDate) {  
        // Defensive copy - see Item 39  
        this.birthDate = new Date(birthDate.getTime());  
    }  
  
    // Other fields, methods  
  
    /** 
     * The starting and ending dates of the baby boom. 
     */  
    private static final Date BOOM_START;  
    private static final Date BOOM_END;  
  
    static {  
        Calendar gmtCal =  
            Calendar.getInstance(TimeZone.getTimeZone("GMT"));  
        gmtCal.set(1946, Calendar.JANUARY, 1, 0, 0, 0);  
        BOOM_START = gmtCal.getTime();  
        gmtCal.set(1965, Calendar.JANUARY, 1, 0, 0, 0);  
        BOOM_END = gmtCal.getTime();  
    }  
  
    public boolean isBabyBoomer() {  
        return birthDate.compareTo(BOOM_START) >= 0 &&  
               birthDate.compareTo(BOOM_END)   <  0;  
    }  
} 

```

**总结**：

- Map的keySet 每次返回的是同一个Set实例。 
- `优先使用基本类型而不是装箱基本类型`，当心无意识的自动装箱,例如：在求和时，将sum的类型从Long改为long，运行速度提成了7倍。。 
- 并不代表我们要尽可能的避免创建对象，对于小对象而言，它的创建和回收动作是非常廉价的。 反之，通过维护自己的`对象池`来避免创建对象并不是一种好的做法。 除非池中的对象是非常`重量级`的。

### 6. 消除过期的对象引用

- 避免内存泄漏。

- 只要类是自己管理内存（数组存储对象引用，及时置位null），程序员就应该警惕内存泄漏问题。

清空对象引用应该是一种例外，而不是一种规范行为，不要求也不建议程序员对于每个对象引用，一旦程序不再用，就把它清空，这通常会把代码弄的很混乱。消除过期引用的最好办法是在最紧凑的作用域范围内定义每一个变量，当作用域被执行完，GC会自动把过期作用域的变量回收掉。

什么时候应该自己清空引用？

一般而言，只要是自己管理内存，就应该警惕内存泄漏问题。假如你开辟了一段内存空间，并一直持有这段空间的引用，就有责任管理它，因为GC无法自动完成对你承诺管理的内存的回收，除非你告诉它（显式地清空引用）。

### 7. 避免使用终结方法

终结方法finalizer通常是不可预测的，也是很危险的，一般情况下是不必要的。 它不保证何时执行，是否执行。且会带来性能损失。

如果需要，提供一个`显式的终止方法`,类似InputStream。 显式的终止方法通常与try-finally结合起来使用，以确保及时终止。`在finally子句内调用显式的终止方法`，可以保证即使在使用对象的时候，有异常抛出，该终止方法也会执行。

**终结方法的合法用途**： 

`1. 使用者忘记调用显式终止方法，终结方法可以充当 “安全网”，迟一点释放比不释放要好`。（希望尽可能不要这样） 。如果终结方法发现资源还未被终止，`应该在日志中记录一条警告`。因为这是客户端的一个bug。但是要考虑这种额外的保护是否值得。 

**四个示例**：FileInputStream、FileOutputStream、Timer、Connection。

```
FileInputStream fileInputStream = new FileInputStream();

try{
    //Do something about fileInputStream;
}finally{
    fileInputStream.close();
}

```

2. 释放native对象的资源。

还有一点，终结方法链不会自动执行。如果类有终结方法，并且子类覆盖了终结方法，要手动调用超类的终结方法。应该在try中终结子类，在finally中调用超类的finalize().

```
 protected void finalize() throws Throwable {
        try {
            //finalize自己
        } finally {
            super.finalize();
        }
    }
    
```

为了防范子类没有手工调用超类的终结方法。可以采用匿名内部类实现一个**终结方法守卫者**。

```
//终结方法守卫者
private final Object finalizerGuardian = new Object() {
    @Override
    protected void finalize() throws Throwable {
        //在这里调用外围实例的finalize
        FinalizeTest.this.finalize();
    }
};
    
```

## 第三章 对于所有对象都通用的方法

### 8. 覆盖equals时请遵守通用约定

“值类（value class）”仅仅是一个表示值的类，例如Integer或Date。需要覆盖equals（）。 
有一种值类不需要覆盖，即第一条确保下的“每个值至多只存在一个对象”的类。枚举类型就属于这种类。 
equals()方法实现了等价关系。

1. **自反性**。对于任何非null的引用值x，x.equals(x)必须返回true.(对象必须等于其自身)
2. **对称性**。对于任何非null的引用值x和y，当且仅当y.equals(x)返回true时，x.equals(y)必须返回true。（警惕子类父类不同的equals实现时）
3. **传递性**。对于任何非null的引用只x、y和z，如果x.equals(y)返回true，并且y.equals(z)也返回true，那么x.equeals(z)也必须返回true。（优先使用组合而不是继承，可以避免违反传递性 对称性，否则没有完美解决方案。 只要超类不能直接创建实例，则不会违反原则）
4. **一致性**。对于任何非null的引用值x和y，只要equals的比较操作在对象中所用的信息没有被修改，多次调用x.equals(y)返回值一致。（）
5. **非空性**。对于任何非null的引用值x，x.equals(null)必须返回false。（不用重复判空，只需要用instance 即可）

实现`高质量equals方法`的诀窍：

1. **使用==操作符** 检查”参数是否为这个对象的引用”。如果是，返回true。这是一种性能优化，适用于比较操作昂贵的情况。
2. **使用instanceof操作符检查**“参数是否为正确的类型”。如果不是，返回false。
3. 把参数**强转**成正确的类型。
4. 对于该类中的每个“关键”域进行**匹配比较。（float使用Float.compare()比较，double使用Double.compare()比较**。 有些filed可能为null,且合法。 可以用 (field ==null? o.field==null: field.equals(0.field)) 。 如果field和o.field通常是相同的对象引用，那么下面的做法更快一些: ( field == o.field || field!=null && field.equlas(0.field))）
5. 编写完equals方法之后，应该去测试 对称性、传递性、一致性。
6. 覆盖equals时，同时覆盖hashCode。
7. 不要将**equals()方法声明中**的**Object对象**替换为其他的类型。(public boolean equals(MyClass o)){})

```
 //一个示例
    @Override
    public boolean equals(Object o) {
        if (o == this) {
            return true;
        }
        if (!(o instanceof EnumSingletonTest)) {
            return false;
        }
        EnumSingletonTest data = (EnumSingletonTest) o;
        return data.xxx == xxx
                && dta.bbb == bbb;
    }
```

### 9. 覆盖equals时 总要覆盖 hashCode

不覆盖该方法，会导致该类无法配合HashMap、HashSet和Hashtable一起使用。 
**因为equals()相等的话，hashCode（）一定要相等。**
一个好的散列函数通常倾向于“**为不相等的对象产生不相等的散列码”**。

### 10. 始终要覆盖 toString

### 11. 谨慎地覆盖clone

一般，`x.clone() != x ,x.clone().getClass() == x.getClass(), x.clone().equals(x) `（但也不强求） 
实际上，**clone方法**就是**另一个构造器**；你必须确保它不会伤害到原始的对象，并确保正确地创建被克隆对象中的约束条件。 
更好的办法是**类似集合类**那样，提供 **转换构造器 和 转换工厂**。且转换构造器和工厂可以**带一个接口类型的参数**。 例如你有一个HashSet，想得到TreeSet，clone方法无法提供这样的功能，但是用转换构造器很容易实现, new TreeSet(s)。 
专家级程序员不建议用clone。

### 12. 考虑实现 Comparable 接口

实现compareTo()方法 。它的约定和equals方法相似。（自反、传递、对称），但它不用进行类型检查。比较时，顺序很重要，**先比较重要的，如果产生了非零的结果，则结束比较**。

**小于返回负整数，等于返回0，大于返回正整数。墙裂建议，(x.compareTo(y)==0) == (x.equals(y))**

用于TreeSet TreeMap，以及工具类Collections、Arrays，内部包含搜索和排序算法。

如果一个域没有实现Comparable接口，或者需要使用一个非标准的排序关系。可以用Comparator来代替，或者使用已有的Comparator。例如：

```
public final class A implements Comparable<A>{
    private String s ;
    public int compareto(A a){
        return String.CASE_INSENSITIVE_ORDER_.compare(s,a.s);
    }
}

```
有些时候可以直接用 this.s - other.s 来作为返回值，但是要确保 **最小和最大的可能域值之差小于或等于INTEGER.MAX_VALUE**。否则计算结果超过int型溢出时， 结果将相反。

### 第四章 类和接口

### 13. 使类和成员的可访问性最小化

隐藏内部数据和实现细节。 
**尽可能**的使每个类或者成员**不被外界访问**。 
对于**顶层的（非嵌套的）**类和接口，只有**两种**可能的访问级别：**包级私有（缺省） 和 公有的**。 
如果一个包级私有的顶层类活接口 只是在某一个类的内部被用到，就应该考虑使他成为那个类的私有嵌套类。 
然而，降低不必要的共有类的可访问性，比上一点更重要。

对于成员，依次有：**private、包级私有（default）、protected（只有子类和包内可以访问）、public**
protected应该少用，它和public都作为公开API的一部分。

如果方法覆盖了超类中的一个方法，**子类中的访问级别就不允许低于超类中的访问级别**。这样可以确保任何可使用超类的地方也可以使用子类。

**实例域绝不能是公有的**。如果域是非final的，则更不应该。如果final域包含可变对象的引用，它引用的对象是可以被修改的，这会导致灾难性的后果。

长度非零的数组总是可变的。所以，**类具有公有的静态final数组域，或者返回这种域的访问方法，几乎总是错误的**。因为客户端能**修改数组**中的内容，这是一个安全漏洞。 

修正这个问题有两种方法：

访问权限设为私有，同时增加一个公有的不可变列表。

```
private static final A[] PRIVATE_VALUES = {.....};
public static final List<A> VALUES = 
    Collections.unmodifiableList(Arrays.asList(PRIVATE_VALUES ));
    
```

2. 访问权限设为私有，增加一个公有方法，返回私有数组的一个备份：

```
private static final A[] PRIVATE_VALUES = {.....};
public static final A[] values(){
    return PRIVATE_VALUES .clone();
}

```

### 14. 在公有类中使用访问方法而非公有域

**如果类可以在它所在的包的外部进行访问，就提供访问方法。 
如果类是包级私有的，或者是私有的嵌套类，直接暴漏它的数据域并没有本质的错误。 
公有类**永远都**不应该暴露可变的域**。偶尔会**暴漏不可变域**。 
有时候会**暴露包级私有的或者私有的嵌套类的域**，无论这个类可变还是不可变。

### 15. 使可变性最小化

**不可变类只是其实例不能被修改的类**。 
每个实例中包含的所有信息都必须在创建该实例的时候就提供，并在对象的整个生命周期内固定不变。例如String、基本类型的包装类、BigInteger、BigDecimal。

不可变类遵循下面五条规则：

1. **不要提供任何会修改对象状态的方法（改变对象属性的方法）**。
2. 保证类**不会被扩展**。（final或者 私有、包级私有构造器，提供公有静态工厂）
3. 使所有的**域都是final**的。
4. 使所有的**域都成为私有**的。
5. 确保对于任何可变组件的互斥访问。**确保客户端无法获取指向 可变对象的域 的引用。并且永远不要用客户端提供的对象引用来初始化这样的域，也不要从任何访问方法中返回该对象的引用**。

大多数不可变类在修改域时，都是通过返回一个新的对象做的。只对操作数进行运算但不修改它。 
这被成为 函数的 functional做法，与之对应的是过程的procedural 或者 命令式的 imperative，这些方式会导致状态改变。 

```
public class ImmutableExampleTest {
    private final int value;

    public ImmutableExampleTest(int value) {
        this.value = value;
    }

    public int getValue() {
        return value;
    }

    public ImmutableExampleTest add(ImmutableExampleTest b) {
        return new ImmutableExampleTest(value + b.value);
    }
}
```

不可变对象可以只有一种状态，即被创建时的状态。 
不可变对象**本质上是线程安全的**，它们不要求同步。 
不可变对象可以被自由地共享。鼓励客户端尽可能地**重用现有的实例**。可以**对频繁用到的值**，提供公有的静态final常量。 
不可变**对象可以被自由地共享。鼓励客户端尽可能地重用现有的实例。可以对频繁用到的值，提供公有的静态final常量**。 
不可变对象永远不需要保护性拷贝。不应该提供clone方法或者拷贝构造器。 
不仅可以共享不可变对象，甚至也可以**共享它们的内部信息**。例如BigInteger类的negate()方法，返回的BigInteger对象中的数组指向原始实例中的同一个内部数组。 
不可变对象为其他对象提供了大量的构件（building blocks）。 
不强求所有的域都是final的，但必须保证 ：没有一个方法能够对对象的状态产生 外部可见 的改变。可以用非final域去缓存一些昂贵开销的计算的结果（配合lazy initialization）。 
如果不可变类实现了Serializable接口，并且包含一个或多个指向可变对象的域，需要特殊处理。

不可变对象**真正唯一的缺点是，对于每个不同的值都需要一个单独的对象**。 
如果你执行一个多步骤的操作，并且每个步骤都会产生一个新的对象，除了最后的结果之外其他的对象最终都会被丢弃，此时**性能问题**就会暴露出来。

两种解决办法：

1. 先猜测一下经常用到哪些多步骤操作，然后将它们作为基本类型提供。
2. 如果无法预测，最好的办法就是**提供一个公有的可变配套类**。String类是不可变类，它的可变配套类是StringBuilder。BigInteger对应BigSet。

**总结**：
**尽量将小的值对象，成为不可变的。认真考虑将较大的值对象做成不可变的，例如String、BigInteger。只有当性能确实需要优化时，才为不可变的类提供公有的可变配套类。 
如果类不能被做成不可变的，仍然应该尽可能地限制它的可变性。除非有令人信服的理由，否则每个域都是final的**
不应该提供“重新初始化”方法，它会增加复杂性。

### 16. 复合优先于继承（组合大于继承）

继承是实现代码重用的有力手段，但它并非永远是最佳工具。 
在**包内部使用继承**非常安全，因为它属于同一个程序员的**控制之下**。 
对于**专门为继承而设计、并具有很好的文档说明的类，继承也非常安全。 
（这里的继承代表 实现继承 ，指的是一个类继承另一个类。 对于类实现接口、或者接口继承接口都不属于这种情况）**

**与方法调用不同的是，继承打破了封装性**。 
子类依赖于其超类中特定功能的实现细节。超类的实现有可能随着发行版本的不同而有所变化，如果发生了变化，子类可能遭到破坏，即使子类的代码完全没有改变。

**只有当子类和超类之间确实存在 “is-a”关系时，才应该用继承**。 
继承机制会把超类API中的所有缺陷传播到子类中，而复合（装饰、代理）则允许设计新的API 来隐藏这些缺陷。

### 17. 要么为继承而设计，并提供文档说明，要么就禁止继承。

构造器决**不能调用可被覆盖**的方法。不管是直接的还是间接的调用。 
如果类是为了继承而被设计的，无论实现`Cloneable`或`Serializable`都不是好主意。因为它把一些实质性的负担转嫁给扩展这个类的程序员的身上。因为`clone()`或`readObject()`**方法行为上类似于构造器，所以也不可以调用可覆盖的方法，不管以直接还是间接的形式**。

如果必须从这种类继承，一种办法是 确保这个类永远不会调用它的任何可覆盖的方法。

### 18. 接口优于抽象类

现有的类可以很容易被更新，以实现新的接口。 
接口是定义mixin（混合类型）的理想选择。 
接口允许我们构造非层次（竖向）结构的类型框架。

虽然接口不允许包含方法的实现。**但通过对每个重要接口都提供一个抽象的骨架实现（skeletal implementation）类，把接口和抽象类的优点结合起来**。接口的作用仍然是定义类型，但是骨架实现类接管了所有与接口实现相关的工作。 
按照惯例，骨架实现类称为AbstractInterface，例如AbstractCollection、AbstractSet等。

骨架实现上有个小小的不同，就是**简单实现**。AbstractMap.SimpleEntry就是个例子，简单实现类似于骨架实现类，因为它实现了接口，并且也是为了继承而设计的，区别在于它不是抽象的：它是最简单的可能的有效实现。你可以原封不动地使用，也可以看情况将它子类化。

抽象类比接口有一个**明显的优势**：抽象类的**演变**比接口的演变要**容易的多**。 
如果在后续的版本中，希望在抽象类中增加新的方法，始终可以增加具体方法，包含合理的默认实现即可。 
一般来说，想在公有接口中增加方法，而不破坏实现这个接口的所有现有的类，这是不可能的。可以通过在为接口增加新方法的同时，也为骨架实现类增加同样的新方法，来一定程度上减小破坏。

**因此，一个接口一旦公开发行，再修改基本不可能**。

**总结**：

接口通常是定义允许多个实现（多态）的最佳途径。 
如果你想要演变的容易性比功能、灵活性更重要，而且你接受抽象类的局限性，就用抽象类。 
接口->骨架实现类->简单实现类。

### 19. 接口只用于定义类型

除此之外，为了任何其他的目的而定义接口是不恰当的。 
说白了，定义接口是为了接口的行为。 
有一种接口是常量接口，这是对接口的不良使用。因为常量是实现细节，不应该泄漏出去。对类的用户来说没有价值，反而会干扰。而且如果非final类实现了常量接口，它的所有子类的命名空间也会被接口中的常量所“污染”。 

**定义常量应该用枚举或者不可实例化的工具类**。

### 20. 类层次优于标签类

想到了自己项目里 群聊创建模块。 原本就是一个标签类，改造成了类层次。 
类层次好处： 
简单清晰，没有样板代码，不受无关数据域的拖累，多个程序员可独立扩展层次结构。而且类层次可以反映类型之间本质上的层次关系，有助于增强灵活性。

标签类有很多缺点：

1. 可读性差
2. 内存占用增加因为实例承担着属于其他风格的不相关的域。
3. 域不能是final的。 
总之它过于冗长，容易出错，效率低下。
其实标签类是对类层次的一种简单的效仿。

将标签类转化成类层次：

1. 为标签类的每个方法都定义一个包含抽象方法的抽象类，这里的每个方法的行为都依赖于 标签值。
2. 如果有行为不依赖于标签值的方法，就将这些方法放在抽象类里。
3. 如果所有的方法都用到了某些域，则将域也放在抽象类里。
4. 为每种原始标签书写相应的子类。

### 21. 用函数对象表示策略

如果一个类，它只有一个方法，这个方法执行其他对象（通过参数传入）上的操作。 
这个类的实例实际上等同于一个指向该方法的指针。 
这样的实例称之为**函数对象**。也是一个具体策略类。 
我们在设计具体的策略类时，还需要定义一个策略接口。

**总结**：

函数指针的主要用途就是实现策略模式。 
为了在java中实现这个模式，要声明一个接口来表示该策略，并且为每个具体策略实现该接口。 
当具体策略只使用一次时，通常用匿名内部类来做。 
当重复使用时，通常实现为某个类私有的静态成员类，并通过 公有 静态 final 域 被导出，域类型是策略接口类型。

### 22. 优先考虑静态成员类

嵌套类指定义在另一个类内部的类。目的是 只为它的外围类服务。 
如果嵌套类将来可能用于其他的环境中，它就应该是顶层类。 
嵌套类四种:

1. 静态成员类（优先）
2. 非静态成员类
3. 匿名类（不能拥有静态成员，如果只有一处创建的地方）
4. 局部类（有多处创建的地方）

后三种被称为**内部类**。

静态成员类是最简单的一种嵌套类，最好把它看作是普通的类，只是碰巧被声明在另一个类的内部而已。 
如果声明内部类不要求访问外围实例，就要声明称 静态成员类。 
内部类会包含一个额外的指向外围对象的引用，保存这份引用要消耗时间和空间，也会影响GC.

匿名类的常见用法：

1. 动态创建函数对象。（21条）
2. 创建过程对象，比如Runnable、Thread。
3. 静态工厂方法的内部。


### 第五章 泛型

出错之后应该尽快发现，最好是在编译时就发现。

### 23. 请不要在新代码中使用原生态类型

`List<T>`对应的原生态类型就是`List`,如果使用原生态类型，就丢掉了泛型在安全性和表述性方面的所有优势。它只是为了兼容遗留代码的。 
无限制的通配符类型：`List<?>`，等`于List`,它们和`List<Object>`都不相等。 
在类文字 中必须使用原生态类型。`List.class String[].class ini.class`都合法，`List<String>.class`和 `List<?>.class` 都不合法。

### 24. 消除非受检警告

尽可能地消除每一个非受检警告。 如果消除所有警告，则可以确保代码是类型安全的。意味着在运行时不会出现ClassCastException异常。

如果无法消除警告，同时可以证明引起警告的代码是类型安全的，只有在这种情况之下，才可以用一个@SuppressWarnings(“unchecked”)注解来禁止这条警告。 
应该在尽可能小的范围里使用该注解。并添加注释解释为什么这么做是安全的。

### 25. 列表优先于数组

数组与泛型相比，有两个重要的不同： 
**一 数组是协变的，泛型是不可变的**。 
如果Sub是Super的子类型，那么Sub[]就是Super[]的子类型。 
然而泛型则不是。对于任意两个不同的类型`Type1,Type2,List<Type1>和List<Type>`不可能是子类型或父类型的关系。 
实际上，这不是泛型的缺陷，反而是数组的缺陷：

```
 //fails at runtime!
        Object[] objectArray = new Long[1];
        objectArray[0] = "I don't fit in";//Throws ArrayStoreException

        //Won't compile
        List<Object> listObject = new ArrayList<Long>();//Incompatible types
```
上述代码，无论哪种方法，都不能将String放入Long容器中。但是利用列表，可以在编译时发现错误.

**二 数组是具体化的 在运行时才知道自己的类型，泛型通过擦除，在运行时丢弃类型信息**

一般来说，数组和泛型不能很好地混合使用。 
如果混合使用出现了编译时错误，应该用列表替代数组。

### 26. 优先考虑泛型

不能直接创建 `new T[16]`,可以通过创建 `(T[]) new Object[16]`;强转。也可以在取出数据时强转。

有些泛型如ArrayList，必须在数组上实现。 

为了提升性能，其他泛型如HashMap也在数组上实现。

### 27. 优先考虑泛型方法

静态工具方法尤其适用于泛型化。 
泛型方法一个显著特征，可以配合 类型推导，无需明确指定类型参数的值。

### 28 利用 有限制通配符 来提升API的灵活性

**PECS表示，producer（取）-extends，consumer（存）-super**。 
还要记住所有的**comparable和comparator都是消费者**。

为了获得最大限度的灵活性，要在表示生产者或者消费者的**输入参数**上**使用通配符类型**。 
如果某个**输入参数** 即是生产者又是消费者，则通配符类型对你没有好处，需要严格的类型匹配。 
**不要用通配符作为返回类型**。

如果类的用户必须考虑通配符类型，类的API或许就会出错。

显式的类型参数：
```
Set<Number> numbers = Union.<Number>union(integers,doubles);
```

一个复杂的例子：

```
public static <T extends Comparable<? super T>> T max(List<? extends T> list)

```

类型参数和通配符之间具有双重性，许多方法都可以利用其中一个或者另一个进行声明。 
一般来说，如果**类型参数**只在**方法声明中出现一次**，就可以**用通配符取代它**。 
在公共API中，推荐第二种。 
例如：

```
    //类型参数和通配符之间具有双重性，许多方法都可以利用其中一个或者另一个进行声明
    //一般来说，如果类型参数只在方法声明中出现一次，就可以用通配符取代它。
    public static <E> void swap1(List<E> list, int i, int j) {
    }

    public static void swap2(List<?> list, int i, int j) {

    }

    public static <E> void swap3(E list, int i, int j) {

    }
    //这个是错误的， 因为E不是类型参数
  /*public static void swap4(? list, int i, int j) {

    }*/
    
```

### 29. 优先考虑 类型安全 的 异构容器

类型安全：当你向它请求String时，它从来不会返回一个Integer给你 
异构容器：不像普通的map，它的所有key都是不同类型的。

泛型最常用于集合。 
`当一个类的字面文字(Class<T>)被用在方法中，来传达编译时和运行时的类型信息时，就被称作type token`。

注解API广泛利用了有限制的类型令牌。 被注解的元素，本质上是个类型安全的异构容器。

### 第六章 枚举和注解

1.5中增加了 两个新的引用类型：**枚举类型、注解类型**。

### 30. 用enum代替 int 常量。

枚举易读，**提供了编译时的类型安全**。 
**允许添加任意的方法和域（本质上是个类）**。

如果枚举具有普适性，应该成为一个顶层类。如果用在特定顶层类中，应该成为该顶层类的一个成员类。

但枚举有小小的性能缺点，装载和初始化枚举时会有空间和时间的成本。

### 31. 用实例域 代替序数（ordinal（））
永远不要根据枚举的序数导出与它关联的值，而是要将它保存在一个实例域中。 
最好完全避免ordinal方法的使用。

### 32. 用EnumSet 代替位域

位域有更多缺点，打印时翻译位域更困难，遍历位域也困难。

EnumSet内部用单个long保存，（枚举类型小于等于64个）

枚举类型要用在集合中，所以没有理由用位域来表示它。 
用EnumSet，简洁、性能也有优势。

```
public static void main(String[] args) {
     EnumSet<Style> bold = EnumSet.of(Style.BOLD, Style.ITALIC);
     System.out.println(bold);
 }

 enum Style {
     BOLD, ITALIC, UNDERLINE;
 }
 //[BOLD, ITALIC]
 
```

### 33. 用EnumMap 代替序数（ordinal（））索引

按Enum分组索引，key是Enum。 
最好不要用序数来索引数组，而要使用EnumMap。

### 34. 用接口模拟可伸缩的枚举

用接口模拟可伸缩枚举有个小小的不足，无法从一个枚举类型继承到另一个枚举类型。 
虽然无法便携可扩展的枚举类型，却可以通过编写接口以及实现该接口的基础枚举类型，对它进行模拟

### 35. 注解优先于 命名模式

命名模式缺点：

1. 文字拼写错误会导致失败，且没有任何提示。
2. 无法确保它们只用于相应的程序元素上。
3. 没有提供将参数值与程序元素关联起来的好办法。

### 36. 坚持使用Override注解

### 37. 用标记接口定义类型

标记接口是没有包含方法声明的接口，而只是知名一个类实现了具有某种属性的接口。 
例如Serializable。

**如果想要定义类型，一定要使用接口**

### 第七章方法

### 38. 检查参数的有效性

应该在文档中清楚的指明参数的限制 以及 抛出的异常。

**并在方法体的开头检查参数**。 

这是“应该在发生错误之后尽快检测出错误”原则的具体情形。

检查构造器参数的有效性是非常重要的。

### 39. 必要时进行保护性拷贝

编写一些面对客户的不良行为时仍能保持健壮性的类，是非常值得的。

**对构造器的每个可变参数进行保护性拷贝是必要的**。

**保护性拷贝是在检查参数的有效性（上一条38）之前进行的，并且有效性检查是针对拷贝之后的对象，而不是针对原始的对象**。 这样做可以避免在“危险阶段”（从检查参数开始，直到拷贝参数之间的时间段）期间从另一个线程改变类的参数。

**对于参数类型可以被不可信任方子类化的参数，请不要使用clone方法进行保护性拷贝**。

返回可变内部域的保护性拷贝

**总结**： 

如果拷贝的成本受到限制，并且类信任它的客户端不会不恰当的修改组件，就可以在文档中说明客户端的职责，代替保护性拷贝。

### 40. 谨慎设计方法签名

本条目是若干API设计技巧的总结。

- 谨慎地选择方法的名称。易于理解、风格一致、大众认可，可参考Java类库。
- 不要过于追求提供便利的方法。方法太多会使类难以学习、使用、文档化、测试和维护。为每个动作提供一个功能齐全的方法，只有当一项操作被经常用到的时候，才- - 考虑为它提供快捷方式，如果不确定，还是不提供快捷为好。
- 避免过长的参数列表。目标是四个参数或更少。 相同类型的长参数序列格外有害。（拆分、辅助类、Builder）
- 对于参数类型，要优先使用接口而不是类。
- 对于boolean参数，要优先使用两个元素的枚举类型。易于阅读和编写，更易于后期添加更多的选项。

### 41. 慎用重载

重载是发生在编译时的，所以严格的说，它并不是多态。 
**要调用哪个重载方法是在编译时做出决定的。 
对于重载方法的选择是静态（编译时的对象类型决定）的，对于覆盖的方法的选择则是动态（运行时的对象类型决定）的**。

避免胡乱地使用重载机制。 
**安全而保守的策略是，永远不要导出两个具有相同参数数目的重载方法。 
如果方法使用可变参数，保守的策略是根本不要重载它**。

例如，考虑ObjectOutputStream类，对于不同类型，提供诸如writeBoolean(boolean)、writeInt(int)方法，好处还可以提供对应的readBoolean()readInt()方法。

导致混乱主要是由于 参数是 **有继承关系的类型**。应该避免 同一组参数只许经过类型转换就可以被传递给不同的重载方法。 
对于每一对重载方法，至少有一个对应的参数在**两个重载方法中具有“根本不同”的类型**。 
但是当**自动装箱和泛型成了Java语言的一部分之后，谨慎重载显得更加必要了**。 

例如List的remove()方法。如果配合Integer,会搞不清是要按下标删除还是删除对应的元素。

**如果被迫违反本规则，要保证让更具体化的重载方法把调用转发给更一般化的重载方法**。 

例如：
```
 public boolean contentEquals(StringBuffer sb) {
        return contentEquals((CharSequence)sb);
    }
    
```
### 42. 慎用可变参数

可变参数方法，接受，`0个或多个`指定类型的参数。

可变参数也会影响性能，每次方法调用都会导致数组进行一次分配和初始化。

### 43. 返回零长度的数组或者集合，而不是null

除非分析表明这个方法正是造成性能问题的真正源头。 
且返回同一零长度数组是可能的，因为零长度数组不可变，可以被自由共享（15条）。 
不可变的空集合：`Collections.emptySet(),Collections.emptyList(),Collections.emptyMap()`,

### 44. 为所有导出的API元素编写文档注释

## 第八章 通用程序设计

### 45. 将局部变量的作用域最小化

本条目与13条（使类和成员的可访问性最小化）本质上是类似的。

- **在第一次使用它的地方声明**。
- 如果在循环终止之后不再需要循环变量的内容，**for循环优先于while循环**
- **使方法小而集中**。

### 46. for-each循环优先于传统的for循环

集合数组，以及任意实现Iterable接口的对象，都能用for-each。 
嵌套迭代时，优势更明显。 

三种常见情况无法使用for-each：

1. 过滤—-遍历集合时要删除选定的元素，需要用迭代器。
2. 转换—-遍历列表或数组时，需要取代它部分或者全部的元素值。需要用迭代器或者数组索引。
3. 平行迭代—–故意的嵌套迭代（很少使用）

### 47. 了解和使用类库

### 48. 如果需要精确的答案，请 避免使用 float和double

它们尤其不适合于货币就算。 
使用**BigDecimal、int、long**进行货币计算

### 49. 基本类型优先于装箱基本类型

它们的区别：

1. 基本类型只有值，装箱基本类型是**引用**
2. 基本类型只有值，而装箱基本类型还有**null**
3. 基本类型通常更**节省时间、空间**

**对装箱基本类型运用 == 操作符，几乎总是错误的。 
当在一项操作中混合使用基本类型和装箱基本类型时，装箱基本类型就会 自动拆箱。 null被拆箱就抛出NPE**。

合理使用装箱基本类型的地方：

1. 集合中的元素、键、值。因为基本类型不能放在集合中。
2. 参数化类型中
3. 反射的方法调用时


### 50. 如果其他类型更适合，则尽量避免使用字符串。

- 字符串不适合代替其他的值类型。（从网络接受一段数据）
- 字符串不适合代替枚举类型
- 字符串不适合代替聚集类型。例如String key = className + "#" + i.next();应该用一个类来描述这个数据集，通常是一个私有的静态成员类。
- 字符串也不适合代替能力表。

### 51. 当心字符串连接的性能

为连接n个字符串而重复地使用字符串连接操作符，需要n的平方级的时间。

### 52. 通过接口引用对象

如果有合适的接口类型存在，对于 参数、返回值、变量、域来说，都应该使用接口类型进行声明。 
如果没有，则使用类层次结构中提供了必要功能的最基础的类。 
使得程序更灵活。

### 53. 接口优先于反射机制

缺点：

1. 丧失了编译时类型检查的好处
2. 反射的代码长
3. 性能损失

### 54. 谨慎使用本地方法

**使用本地方法来提高性能的做法不值得提倡**。 
如果本地代码只是做少量的工作，本地方法就可能降低性能。

### 55. 谨慎地进行优化
不要因为性能而牺牲合理的架构。**要努力编写好的程序而不是快的程序**。 
好的程序体现了**信息隐藏**，只要有可能可以改变单个决策，而不会影响到系统其他部分。

**努力避免那些限制性能的设计决策**。

**为了获得好的性能而对API进行包装，这是一种非常不好的想法**。（性能因素可能随着版本时间而被解决，但是API会一直留在那里限制自己）

构建完系统之后，要测量性能。利用各种工具，找出瓶颈。优化前、优化后也要测量。

### 56. 遵守普遍接受的命名惯例
类型参数（泛型），通常是 T 任意类型，E 集合元素，KV键值，X异常。

方法常用动词、动词短语。

## 第9章 异常

### 57. 只针对异常的情况才使用异常
























