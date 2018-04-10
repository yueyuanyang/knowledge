
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

**总结：**

和构造器比，各有好处。优先使用静态工厂。

（重用对象的好处 可以用 == 替代 equals，性能更好。静态工厂参考Boolean.valueOf().）

---

### 2. 遇到多个构造器参数时，要考虑用Builder

静态工厂和构造器有个共同的局限性：它们都不能很好地扩展到大量的可选参数。 

Builder模式，既能保证像重叠构造器模式那样的安全性，也能保证像JavaBeans模式那么好的可读性。还可以有多个可变参数。

Builder模式构建时，被构建类的field一般都是final。

`如类的构造器或者静态工厂中具有多个参数，特别大多数参数时可选的时候，使用Builder模式。`

（最好一开始就用Builder，否则兼容以前的构造函数和静态工厂很烦。）

### 3.用私有构造器或者枚举类型强化Singleton属性

```
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

副作用是 使得一个类不能被子类化。可用作常量定义工具类。

### 5. 避免创建不必要的对象

Map的keySet 每次返回的是同一个Set实例。 
`优先使用基本类型而不是装箱基本类型`，当心无意识的自动装箱。 
并不代表我们要尽可能的避免创建对象，对于小对象而言，它的创建和回收动作是非常廉价的。 
反之，通过维护自己的`对象池`来避免创建对象并不是一种好的做法。 
除非池中的对象是非常`重量级`的。


### 6. 消除过期的对象引用

避免内存泄漏。

只要类是自己管理内存（数组存储对象引用，及时置位null），程序员就应该警惕内存泄漏问题。

### 7. 避免使用终结方法

终结方法finalizer通常是不可预测的，也是很危险的，一般情况下是不必要的。 它不保证何时执行，是否执行。且会带来性能损失。

如果需要，提供一个`显式的终止方法`。类似InputStream。 
显式的终止方法通常与try-finally结合起来使用，以确保及时终止。`在finally子句内调用显式的终止方法`，可以保证即使在使用对象的时候，有异常抛出，该终止方法也会执行。

终结方法的合法用途： 

`1 使用者忘记调用显式终止方法，终结方法可以充当 “安全网”，迟一点释放比不释放要好`。（希望尽可能不要这样） 。如果终结方法发现资源还未被终止，`应该在日志中记录一条警告`。因为这是客户端的一个bug。但是要考虑这种额外的保护是否值得。 

四个示例：FileInputStream、FileOutputStream、Timer、Connection。

2 释放native对象的资源。

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


















