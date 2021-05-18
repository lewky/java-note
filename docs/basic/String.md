<!--
date: 2021-03-27T23:48:12+08:00
lastmod: 2021-05-18T23:48:12+08:00
-->
## String的不可变

String和包装类一样，都是final的，都实现了Serializable接口。

String是不可变的，原因是内部用于存储字符的`value`数组变量是final的，一旦被初始化就不可重新赋值；且String没有提供修改value这个数组变量的方法。

String还有一个重要的变量`hash`，即所谓的哈希：
```java
/** Cache the hash code for the string */
private int hash; // Default to 0
```

在jdk1.8中，用char数组存储字符串：
```java
/** The value is used for character storage. */
private final char value[];
```

在jdk1.9之后，用byte数组存储字符串，并用`coder`标识字节的编码：
```java
/** The value is used for character storage. */
private final byte[] value;

/** The identifier of the encoding used to encode the bytes in {@code value}. */
private final byte coder;
```

### 为什么jdk1.9改用byte数组来存储字符串

主要是为了节省空间。

char占据两个字节，byte占据一个字节。对于char，如果存储的字符串是LATIN1编码(即ISO-8859-1)，那么就只需要一个字节，高位的空间其实浪费了。所以在jdk1.9中，通过byte数组和coder编码来存储字符串，可以在使用单字节编码时(WINDOWS-1252、ISO-8859-1、UTF-8)起到节省空间的作用。

coder变量只有两个值：0和1。

● 0 代表Latin-1（单字节编码）<br>
● 1 代表 UTF-16 编码（双字节编码）

* [Java9 后String 为什么使用byte[]而不是char?](https://www.jianshu.com/p/9043243df546)

### 为什么String不可变

Ⅰ. 可以存储于字符串常量池（String Pool）。

只有当String是不可变的，才能确保从字符串常量池中获取到的字符串引用不会指向一个错误的字符串对象（防止String的值被修改）。

Ⅱ. 方便缓存hashcode。

因为String的不可变，可以保证hash也不会改变，只需要计算一次hash即可。虽然hash可以标识一个String对象，但是并不能单纯拿hash来对比两个String是否相等。hash值存在碰撞的可能，但是可以快速定位数据，缩小需要查找的数据范围。比如从jdk1.7开始switch支持String类型，就是用了hash + equals来实现的。

Ⅲ. 辅助其它对象的使用。

比如在一个HashSet中存储String对象，只有String是不可变的，才能确保不会违反Set中元素不重复的设计。

Ⅳ. 安全性。

String经常作为方法参数来传递，String的不可变性可以保证参数的不可变。比如说作为网络连接、账号密码等参数，如果String的值是可变的，就可能导致上下文值不相同的情况，可能导致系统安全性的问题。

Ⅴ. 线程安全。

不可变对象是天然的线程安全的，可以在多线程中安全地使用。

* [Why String is immutable in Java?](https://www.programcreek.com/2013/04/why-string-is-immutable-in-java/)

### hashCode()为什么使用31作为乘子

String重写了Object类的hashCode方法来计算hash值，其实现如下：
```java
public int hashCode() {
    int h = hash;
    if (h == 0 && value.length > 0) {
        char val[] = value;

        for (int i = 0; i < value.length; i++) {
            h = 31 * h + val[i];
        }
        hash = h;
    }
    return h;
}
```

● 31是一个奇质数，使用质数作为乘子可以得到比较好的均匀分布区间，更好地降低哈希算法的冲突率。这是学者们经过大量计算得出的一个结论。<br>
● 31占据5个bit，使用比31小的质数，得出的hash值较小，较容易产生冲突。使用比31大的质数，则容易发生运算溢出。<br>
● 31可以被jvm优化，`31*i`可以优化为`(i << 5) - i`。此时乘法运算被转为移位和减法运算。

此外还有个说法，因为31是一个奇素数，如果是偶数的话，当出现乘法溢出，信息就会丢失，因为与 2 相乘相当于向左移一位，最左边的位丢失。 

但是个人并不认可这种说法，对于素数（即质数），2是唯一的偶质数，作为乘子太小；而乘子通常都是选择质数，也就是说乘子一般都是奇质数。当出现乘法溢出时，不管乘子是不是偶数，必然会丢失精度。如下两个31相乘的结果，对于byte来说会溢出导致：
```java
byte a = (byte) (31*31);  //-63
```

* [哈希算法之 hashCode 为什么选择数字31作为优质乘子？](https://blog.csdn.net/weixin_37186559/article/details/84255170)

## 编译时常量和运行时常量

被`final`修饰的String属于常量，其值一旦确定下来就不可再变化，即不可重新赋值。按照常量的作用域，可以分为静态常量、成员常量和局部常量。根据编译器的不同行为，也可分为编译时常量和运行时常量。

编译时常量会在编译成class文件时被替换为对应的值（常量值，即字面量literal，也叫字面值。常量和常量值不是一个东西。），也就是说，编译时常量在编译阶段就已经可以确定值，该常量的引用在当前class文件里是找不到的。如果一个静态常量的值是基本数据类型字面量、String字面量或者是只涉及到这二者的常量表达式，则该常量是编译时常量。编译时常量必须是`static final`的。

运行时常量是在运行阶段才能确定值的常量，除了基本数据类型字面量和String字面量（即非new出来的String）以外的常量，都是运行时常量。

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

可以看到，只涉及到基本数据类型字面量和字符串字面量的常量表达式依然是编译时常量，而包装类常量则是运行时常量。对于new出来的String，在赋值给常量后也是运行时常量（有兴趣的可以试试）。

Java对于基本数据类型字面量和字符串字面量做了特殊的处理，因为二者可以放入常量池，不依赖于类，在编译期即可确定值。**基本数据类型的字面量存放于运行时常量池，字符串字面量存放于字符串常量池。**

而其他的对象类型：包括包装类、枚举类、new出来的String对象等，都是存放于堆内存中的，依赖于类，必须在运行期才能确定值。

## 字符串常量池String Pool(也叫String Table)

字符串常量池存放的是字符串的字面量，在jdk1.7之前，字符串常量池存在于运行时常量池中，运行时常量池在方法区中（Hotspot里叫永久代PermGen）。

jdk1.7开始逐渐去永久代化，字符串常量池和静态变量被转移到堆中。

jdk1.8及之后，取消永久代，但是新增了本地内存的元空间Metaspace，用以保存类型信息、字段、方法、常量。

## String的intern方法

String#intern()是一个本地方法，用来将字符串放入常量池，在不同的jdk有不同的实现区别：

● jdk1.6及以前：当字符串在常量池存在时，则返回常量池中的字符串；当字符串在常量池不存在时，则**在常量池中拷贝一份**，然后再返回常量池中的字符串。<br>
● jdk1.7：当字符串在常量池存在时，则返回常量池中的字符串；当字符串在常量池不存在时，则**把堆内存中此对象的引用添加到常量池中**，然后再返回此引用。

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

● 首先在创建s1字符串的时候，会先在常量池里添加两个字面量：`go`和`od`；最后s1指向的是堆中new出来的字符串对象`good`。<br>
● 调用`s1.intern()`时，会去常量池查找是否存在`good`这个字符串。查找的时候是通过`equals()`来比较的，此时常量池里不存在`good`这个字符串，所以会把s1这个引用放入常量池，然后返回该引用。<br>
● 此时`s1.intern() == s1`的结果自然就是true了。

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

## String，StringBuffer，StringBuilder

### 可变性

● String是不可变的<br>
● StringBuffer和StringBuilder是可变的

### 线程安全

● String是线程安全的，由String的不可变保证的<br>
● StringBuffer是线程安全的，由synchronize保证<br>
● StringBuilder是线程不安全的，但是性能比StringBuffer更加高效。在没有线程安全问题时使用StringBuilder就够了。

## 二进制安全字符串（Binary-safe strings）

>在 C 语言中，字符串里面不能包含空字符'\0'，否则这个空字符会被当做是字符串结尾，换句话说，C 语言的字符串默认是以 '\0' 结尾的，这不是二进制安全的，因为图片、音频等二进制数据里面会有 '\0' 这一字符，C 字符串会忽略 '\0' 这一字符后面的数据。

换言之，二进制安全字符就是不管输入什么字节都能正确处理，即使包含了零值字节。

Redis的字符串就是二进制安全的。

* [什么是二进制安全？](https://www.zhihu.com/question/24214241/answer/262778202)

## 一些相关的题目

### 怎样将GB2312编码的字符串转换为ISO-8859-1编码的字符串？

```java
String s1 = "你好";
String s2 = new String(s1.getBytes("GB2312"), "ISO-8859-1");
```

### 如何实现字符串的反转及替换？

方法有很多，这里提供3种：

Ⅰ. 使用jdk自带的reverse()

```java
String str="abc";
String result = new StringBuilder(str).reverse().toString();
```

Ⅱ. 递归实现

```java
public static String reverse(String originStr) {
    if(originStr == null || originStr.length() <= 1) 
        return originStr;
    return reverse(originStr.substring(1)) + originStr.charAt(0);
}
```

Ⅲ. 非递归实现

```java
String str="abc";
StringBuilder temp = new StringBuilder();
for (int i = str.length() - 1; i >= 0; i--) {
	temp.append(str.charAt(i));
}
String result = temp.toString();
```

## 参考链接

* [二、String](http://cyc2018.gitee.io/cs-notes/#/notes/Java%20基础?id=%e4%ba%8c%e3%80%81string)
* [编译时常量和运行时常量](https://blog.csdn.net/Honeyhanyu/article/details/77878120?locationNum=6&fps=1)
* [java谜题--java运行时修改引用类的静态常量](https://blog.csdn.net/feiyu8607/article/details/7064751)
* [不同JDK版本中String.intern()方法的区别](https://blog.csdn.net/Game_Zmh/article/details/101701708)
* [JAVA中intern()方法的详解](https://blog.csdn.net/tangyaya8/article/details/79240071)
* [Java不同版本的intern()方法区别](https://blog.csdn.net/sunlihuo/article/details/105275431)
* [深入分析String.intern和String常量的实现原理](https://www.jianshu.com/p/c14364f72b7e)