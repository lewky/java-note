<!--
date: 2021-04-15T23:31:12+08:00
lastmod: 2022-02-09T23:31:12+08:00
-->
## Object基类

Object是所有对象类型的基类（即父类），即使不在类中声明父类，也默认继承了Object类。

Object类中有以下方法：
```java
public final native Class<?> getClass();

public native int hashCode();

public boolean equals(Object obj)

protected native Object clone() throws CloneNotSupportedException;

public String toString()

public final native void notify();

public final native void notifyAll();

public final native void wait(long timeout) throws InterruptedException;

public final void wait(long timeout, int nanos) throws InterruptedException

public final void wait() throws InterruptedException

protected void finalize() throws Throwable { }
```

### getClass()

getClass()是本地方法（调用的是操作系统的库函数），获取当前类的类对象，一般用于反射。

### equals()

equals方法用以判断两个对象是否相等，需要满足以下特性：
```java
//自反性
a.equals(a);  //true

//对称性
a.equals(b) == b.equals(a);  //true

//传递性
if (a.equals(b) && b.equals(c))
    a.equals(c);  //true;

//一致性，即多次调用equals()方法结果不变
a.equals(b) == a.equals(b);  //true

//与null的比较为false
a.equals(null);  //false，a不能为null，否则抛出NullPointerException
```

Object类中equals方法实现很简单：
```java
public boolean equals(Object obj) {
    return (this == obj);
}
````

`==`是比较两个对象的内存地址（对于基本数据类型是直接比较值是否相等），如果是两个不同的对象，则必然返回false。可以通过重写该方法来根据需要判断两个对象是否等价，比如Integer类就重写了该方法：
```java
public boolean equals(Object obj) {
    if (obj instanceof Integer) {
        return value == ((Integer)obj).intValue();
    }
    return false;
}
```

### hashCode()

hashCode()也是本地方法，用以计算哈希值，两个等价的对象其哈希值必定是相同的，但是哈希值相同的两个对象不一定等价。因为哈希算法的缘故，存在哈希冲突的可能性，两个不同的对象是有可能计算出相同哈希值的。

一般在重写equals方法时，要求一起重写hashCode方法，以保证等价的两个对象哈希值也相等。比如Integer类，就同时重写了equals和hashCode方法。

在使用HashSet和HashMap等容器时，这些容器类使用了hashCode方法来计算对象的存储位置，因此被添加进这些容器的对象需要根据实际需求来重写hashCode方法。比如两个等价的对象，由于没重写hashCode方法，就可能导致相同的对象可以出现在Set集合中，同时增加新元素的效率会大大下降（对于使用哈希存储的系统，如果哈希码频繁的冲突将会造成存取性能急剧下降）。

### toString()

```java
public String toString() {
    return getClass().getName() + "@" + Integer.toHexString(hashCode());
}
```

默认实现是返回类名、`@`以及哈希码的十六进制字符串拼接起来的String。

### clone()

Object类中只有两个projected方法，clone()是其中之一。一个类中如果不显示重写该方法，其他类就不能直接调用这个类实例的clone方法。只能在这个类内部或者跟Object类同包内才能调用这个类实例的clone方法（projected语义导致的）。

此外，如果一个类不实现`Cloneable`接口，在调用clone方法时会抛出`CloneNotSupportedException`。Cloneable接口只是一个标识接口，跟Serializable接口一样，是个空接口。**所有的数组默认实现了Cloneable接口，并显示重写了clone方法，其返回值为T[]的数组对象，T是任意的引用类型或原始类型。**

clone方法提供浅拷贝，对于对象类型的成员变量，只会拷贝引用。也就是说，拷贝对象和原始对象的引用类型成员变量指向同一个对象。与之相对的一个概念，则是深拷贝，即拷贝对象和原始对象的引用类型成员变量指向不同的对象。

>使用 clone() 方法来拷贝一个对象即复杂又有风险，它会抛出异常，并且还需要类型转换。Effective Java 书上讲到，最好不要去使用 clone()，可以使用拷贝构造函数或者拷贝工厂来拷贝一个对象。

实际工作中，可以用第三方库来深拷贝一个对象，也可以借助序列化来深拷贝对象：

● Apache的工具库commons-lang3.jar（Apache还有不少其他类型的工具库）提供的`SerializationUtils.clone(obj)`<br>
● Apache的BeanUtils和Spring的BeanUtils，用来快速copy一个bean<br>
● MapStruct映射框架，同样用以处理bean的拷贝和映射，性能更高

### wait()、notify()、notifyAll()

这三个方法和线程的知识点有关，具体可以看后面的[线程笔记](/all/concurrency_01_线程)。

### finalize()

垃圾收集器在销毁对象时会调用该方法，通过重写finalize()可以整理系统资源或者执行其他清理工作。

## 访问权限

|修饰符|当前类|同包|子类|其他包|
|:-|:-|:-|:-|:-|
|public|√|√|√|√|
|protected|√|√|√|×|
|default|√|√|×|×|
|private|√|×|×|×|

类的成员不写访问修饰时默认为default。外部类的修饰符只能是public或默认，类的成员（包括内部类）的修饰符可以是以上四种。

## 抽象类（abstract class）和接口（interface）

抽象类和抽象方法都使用`abstract`关键字声明。如果一个类包含抽象方法，则必须声明为抽象类。但是抽象类中可以不存在抽象方法。

抽象类不可以通过new关键字来实例化，但它可以有自己的构造方法，可以通过子类的实例化来间接实例化抽象类。

接口比抽象类更加抽象，同样不能实例化（但是可以通过new一个接口的匿名内部类来间接实例化）。

接口中不能定义构造方法，且其中的方法全部都是公开的抽象方法。不过从java 8开始，接口允许有默认实现，即用default修饰的方法。这样可以降低维护接口成本，在接口中新增一个新的默认方法，可以不需要修改接口的所有实现类。并且从java 9开始，为了复用代码且不对外暴露方法，允许将方法声明为private。

接口的字段默认都是`public static final`的，即默认都是公共的静态常量；抽象类的字段则没有这种限制。一般不建议在接口中定义常量。

Java是单继承的，所以可以通过实现多个接口来间接实现多重继承的效果。

接口可以继承接口，而且支持多重继承。抽象类可以实现接口，继承具体类或者抽象类。

## 抽象方法能否被static、native或者synchronized修饰？

都不能：

● 抽象方法需要子类重写，而静态的方法是无法被重写的，因此二者是矛盾的。<br>
● 本地方法是由本地代码（如C代码）实现的方法，而抽象方法是没有实现的，也是矛盾的。<br>
● synchronized方法必须持有一个锁对象，默认的锁对象是this，即抽象类自身；但是抽象类不能实例化，因此也是相互矛盾的。

## super

super关键字有两个作用：

● 调用父类的构造方法<br>
● 调用父类的成员变量

### 注意点

● this和super功能类似，但是this是调用自身对象，super是调用父类对象。<br>
● 作为构造器调用时，this()和super()只能出现在构造方法的第一行；且this()和super()不能同时出现在同一个构造方法。<br>
● this()和super()指的是对象，所以不能在static域使用。<br>
● 当方法形参和成员变量重名时，可以用this或者super来区分。

* [super 和 this的异同](https://www.runoob.com/w3cnote/the-different-this-super.html)

## 方法重载(Overload)和重写(Override)

### 什么是方法重载？为什么不能根据返回类型来区分重载？

方法的重载和重写都是实现多态的方式，区别在于前者实现的是编译时的多态性，而后者实现的是运行时的多态性。

>为了满足里式替换原则，重写有以下三个限制：
>
>● 子类方法的访问权限必须大于等于父类方法；<br>
>● 子类方法的返回类型必须是父类方法返回类型或为其子类型。<br>
>● 子类方法抛出的异常类型必须是父类抛出异常类型或为其子类型。
>
>使用 `@Override`注解，可以让编译器帮忙检查是否满足上面的三个限制条件。

重载发生在一个类中，同名的方法如果有不同的参数列表（参数类型不同、参数个数不同或者二者都不同）则视为重载。为什么这里不包括返回类型呢？很简单，如果只是返回类型不同，是无法区分开来的，如下：

```java
float max(int a, int b);
int max(int a, int b);
```

在调用上述两个方法的时候，可以不用返回值，那么你怎么区分调用的是哪个方法？

在《深入理解Java虚拟机》中，6.3.6章节有这样一段：
>在Java语言中，要重载一个方法，除了要与原方法具有相同的简单名称之外，还要求必须拥有一个与原方法不同的特征签名；
>
>特征签名就是一个方法中各个参数在常量池中的字段符号引用的集合，也就是因为返回值不会包含在特征签名之中，因此Java语言里面是无法仅仅依靠返回值的不同来对一个已有方法进行重载。
>
>但在Class文件格式之中，特征签名的范围更大一些，只要描述符不是完全一致的两个方法也可以共存。
>
>也就是说，如果两个方法有相同的名称和特征签名，但返回值不同，那么也是可以合法存于同一个Class文件中的。

### Class文件中同方法名、同参数、不同返回值可以，那为什么Java文件中不行呢？

因为Java语言规范的规定，所以编译时会出现错误。

那为什么Class文件可以呢？因为Java虚拟机规范和Java语言规范不同，两者是分开的。

如有更多兴趣，可以看看这篇文章：[Java语言层面和JVM层面方法特征签名的区别 及 实例分析](https://blog.csdn.net/tjiyu/article/details/53891813)

## 构造器（constructor）是否可被重写？

构造器不能被继承，因此不能被重写，但可以被重载。

## 参考链接

* [五、Object 通用方法](http://www.cyc2018.xyz/Java/Java%20%E5%9F%BA%E7%A1%80.html#五、object-通用方法)
* [六、继承](http://www.cyc2018.xyz/Java/Java%20%E5%9F%BA%E7%A1%80.html#六、继承)
* [深入 -- 为什么不能根据返回类型来区分重载？](https://blog.csdn.net/simba_cheng/article/details/80835646)