<!--
date: 2021-03-27T23:48:12+08:00
lastmod: 2021-03-27T23:48:12+08:00
-->
## 编译时常量和运行时常量

被`static final`修饰的String是常量，其值一旦确定下来就不可再变化，即不可重新赋值。并且根据编译器的不同行为，可分为编译时常量和运行时常量。

编译时常量会在编译成class文件时被替换为对应的值（即字面量），也就是说，编译时常量在编译阶段就已经可以确定值，该常量的引用在当前class文件里是找不到的。基本数据类型、String字面量以及只涉及到这二者的常量表达式，都是编译时常量。

运行时常量是在运行阶段才能确定值的常量，除了基本数据类型和String字面量（即非nwe出来的String）以外的常量，都是运行时常量。

运行时常量涉及到常量类的初始化，而编译时常量则不会，因为编译时常量在编译阶段就被替换为字面量，其引用被优化掉了，所以不需要初始化常量类。

### 样例一

看下面的代码：

```java
public class TestA1 {

    public static void main(final String[] args) {
        System.out.println(TestA2.FIRST + " "  + TestA2.SENCOND + " "  + TestA2.THIRD);
    }
}

public class TestA2 {

    public static final String FIRST = "Hello";
    public static final String SENCOND = "World";
    public static final int THIRD = 233;
}
```

在`TestA1`中引用了`TestA2`的常量，在编译后会发现两个类的字节码文件如下：
```java
public class TestA1 {
  public static void main(String[] paramArrayOfString) {
    System.out.println("Hello World 233");
  }
}

public class TestA2 {
  public static final String FIRST = "Hello";
  
  public static final String SENCOND = "World";
  
  public static final int THIRD = 233;
}
```

可以看到，`TestA2`的3个常量就是编译时常量，在`TestA1.class`中这三个常量相加的表达式被优化成了一个字符串。这意味着，当`TestA2`重新修改了常量的值，如果不重新编译`TestA1`，就会导致两边常量值不一致的问题。

### 样例二

修改`TestA2`代码如下：
```java
public class TestA2 {

    public static final String FIRST = "Hello";
    public static final String SENCOND = null;
    public static final int THIRD = "233".length();
}
```

编译之后的字节码文件如下：
```java
public class TestA1 {
  public static void main(String[] paramArrayOfString) {
    System.out.println("Hello " + TestA2.SENCOND + " " + TestA2.THIRD);
  }
}

public class TestA2 {
  public static final String FIRST = "Hello";
  
  public static final String SENCOND = null;
  
  public static final int THIRD = "233".length();
}
```

可以看到，后面两个常量保留了引用，从之前编译时常量变成了运行时常量。

原因是null值表示该常量未初始化，需要等到运行期才能最终确定这个常量的值。而`length()`需要在运行期才能确定，所以在`TestA1.class`中保留了这两个常量的引用，只有在初始化了`TestA2`之后才能得到这两个常量的值。

### 样例三

修改`TestA2`代码如下：
```java
public class TestA2 {

    public static final String FIRST = "He";
    public static final String SENCOND = FIRST + "llo World" + 1;
    public static final Integer THIRD = 233;
}
```

编译之后的字节码文件如下：
```java
public class TestA1 {
  public static void main(String[] paramArrayOfString) {
    System.out.println("He Hello World1 " + TestA2.THIRD);
  }
}

public class TestA2 {
  public static final String FIRST = "He";
  
  public static final String SENCOND = "Hello World1";
  
  public static final Integer THIRD = Integer.valueOf(233);
}
```

可以看到，只涉及到基本数据类型和字符串字面量的常量表达式依然是编译时常量，而包装类则是运行时常量，对于new出来的String，也是运行时常量（有兴趣的可以试试）。

可以认为，Java对于基本数据类型和字符串字面量做了特殊的处理，因为二者可以放入常量池，不依赖于类，在编译期即可确定值。

而其他的对象类型：包括包装类、枚举类、new出来的String对象等，都是存放于堆内存中的，依赖于类，必须在运行期才能确定值。

## String的intern方法

String#intern()是一个本地方法，用来将字符串放入常量池，在不同的jdk有不同的实现区别：

* 在jdk1.6及以前：当字符串在常量池存在时，则返回常量池中的字符串；当字符串在常量池不存在时，则**在常量池中拷贝一份**，然后再返回常量池中的字符串。
* 在jdk1.6以后（不包括jdk1.6）：当字符串在常量池存在时，则返回常量池中的字符串；当字符串在常量池不存在时，则**把堆内存中此对象的引用添加到常量池中**，然后再返回此引用。

从jdk不同版本的源码注释即可看出差别，如果是在jdk1.6及以前的版本，频繁调用intern方法创建不同字符串常量时，会出现常量池不断创建新的字符串，进而引发永久代内存溢出。因此，jdk1.6以后版本的intern方法可以有效的减少内存的占用，提高运行时的性能。

如果是jdk1.7及之后的版本，对于下面的例子：
```java
final String s1 = new StringBuilder("go").append("od").toString();
final String s2 = new StringBuilder("ja").append("va").toString();
System.out.println(s1.intern() == s1);
System.out.println(s2.intern() == s2);
```

你会发现，上述代码的输出结果如下：
```java
true
false
```

第一个是`true`可以理解，下面是分析过程：
1. 首先在创建s1字符串的时候，会先在常量池里添加两个字面量：`go`和`od`；最后s1指向的是堆中new出来的字符串对象`good`。
2. 调用`s1.intern()`时，会去常量池查找是否存在`good`这个字符串。查找的时候是通过`equals()`来比较的，此时常量池里不存在`good`这个字符串，所以会把s1这个引用放入常量池，然后返回该引用。
3. 此时`s1.intern() == s1`的结果自然就是true了。

第二个的结果却是`false`，这个就很让人困惑了。按道理应该结果也是`true`才对，既然是false，那就说明常量池原本就已经存在`java`这个字符串了。事实也是如此，这个字符串是在加载`sun.misc.Version`这个类时被放入常量池的，如下：
```java
public class Version
{
  private static final String launcher_name = "java";
  private static final String java_version = "1.8.0_144";
  private static final String java_runtime_name = "Java(TM) SE Runtime Environment";
  private static final String java_profile_name = "";
  private static final String java_runtime_version = "1.8.0_144-b01";
  
  public Version() {}
  ...
}
```

可以看到这里实际上存入了好几个字符串到常量池里，当然可能还有其他地方也存入了别的字符串。想更深入地了解intern()的底层实现，可以看看这篇文章：[深入分析String.intern和String常量的实现原理](https://www.jianshu.com/p/c14364f72b7e)。

## 参考链接

* [编译时常量和运行时常量](https://blog.csdn.net/Honeyhanyu/article/details/77878120?locationNum=6&fps=1)
* [java谜题--java运行时修改引用类的静态常量](https://blog.csdn.net/feiyu8607/article/details/7064751)
* [不同JDK版本中String.intern()方法的区别](https://blog.csdn.net/Game_Zmh/article/details/101701708)
* [JAVA中intern()方法的详解](https://blog.csdn.net/tangyaya8/article/details/79240071)
* [Java不同版本的intern()方法区别](https://blog.csdn.net/sunlihuo/article/details/105275431)
* [深入分析String.intern和String常量的实现原理](https://www.jianshu.com/p/c14364f72b7e)