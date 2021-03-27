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

## 参考链接

* [编译时常量和运行时常量](https://blog.csdn.net/Honeyhanyu/article/details/77878120?locationNum=6&fps=1)
* [java谜题--java运行时修改引用类的静态常量](https://blog.csdn.net/feiyu8607/article/details/7064751)
