<!--
date: 2021-03-29T23:31:12+08:00
lastmod: 2021-07-04T23:31:12+08:00
-->
## 三元运算符

### 自动拆箱导致的空指针异常

看下面的代码：
```java
Integer age = user != null ? user.getAge() : 0;
```

咋看上去好像看不出什么问题，然而当user对象里的age变量为null时，就会触发NPE。原因是三元表达式的其中一条表达式的返回值为基础数据类型时，如果另一个表达式的返回值是包装类，则会自动拆箱。

比如这里的`user.getAge()`返回的是`Integer`，由于另一个表达式是0，当`user != null`是true的时候，左边的表达式就变成了`user.getAge().intValue()`，如果user对象里的age变量为null时，就会触发NPE。

如果想避免三元表达式由于拆箱导致的NullPointerException，可以改成如下：
```java
Integer age = user != null ? user.getAge() : new Integer(0);
```

### 三元表达式的隐式类型转换

三元表达式除了会根据另一个表达式的返回值来推断是否进行拆箱，还会和`+=`等操作符一样进行隐式类型转换，数值类型的返回值会自动向高精度转换，如下：
```java
char a = 'a';
int i = 96;

System.out.println(3 >= 2 ? a : 9.0);
System.out.println(3 >= 2 ? i : 9.0);
System.out.println(3 >= 2 ? a : i);
System.out.println(3 >= 2 ? i : a);
System.out.println(3 >= 2 ? 98 : a);
System.out.println(3 >= 2 ? 98 : i);
```

上述的结果如下：
```java
97.0
96.0
a
`
b
98
```

以上结果基于jdk8，另外对于char和int的加法等运算，运算结果会自行转为int类型；**但是在三元表达式中两个表达式返回值分别为char和int时，若返回int类型返回值会转型为char。**

## switch

### 为什么switch不支持long

在 Java 语言规范里中，有说明 switch 支持的类型有：char、byte、short、int、Character、Byte、Short、Integer、String、enum。（jdk1.7之后switch才支持String类型）

为什么只支持上面几种？int、String 都可以，为什么不支持 long ？

原因就是 switch 对应的 JVM 字节码 `lookupswitch`、`tableswitch` 指令**只支持 int 类型**。

下面是 JVM 规范中的说明（https://docs.oracle.com/javase/specs/jvms/se8/html/jvms-3.html#jvms-3.10）：

>The Java Virtual Machine's `tableswitch` and `lookupswitch` instructions operate only on `int` data. Because operations on `byte`, `char`, or `short` values are internally promoted to `int`, a `switch` whose expression evaluates to one of those types is compiled as though it evaluated to type `int`. If the `chooseNear` method had been written using type `short`, the same Java Virtual Machine instructions would have been generated as when using type `int`. Other numeric types must be narrowed to type `int` for use in a `switch`.

byte、char、short 类型在编译期默认提升为 int，并使用 int 类型的字节码指令。所以对这些类型使用 switch，其实跟 int 类型是一样的。

### 为什么可以支持String

switch支持String其实就是语法糖，编译器会根据字符串的hashCode来处理。

例：
```java
String a = "aa";
switch (a) {
  case "aa":
    System.out.println("111");
    break;
  case "AaAa":
    System.out.println("222");
    break;
  case "AaBB":
    System.out.println("333");
    break;
}
```

反编译后：
```java
String var1 = "aa";
byte var3 = -1;
switch(var1.hashCode()) { // 第一个switch，根据hashCode计算第二个switch内的位置
  case 3104:
    if (var1.equals("aa")) {
      var3 = 0;
    }
    break;
  case 2031744:
    if (var1.equals("AaBB")) {
      var3 = 2;
    } else if (var1.equals("AaAa")) {
      var3 = 1;
    }
}

switch(var3) { // 第二个switch，执行原switch的逻辑
  case 0:
    System.out.println("111");
    break;
  case 1:
    System.out.println("222");
    break;
  case 2:
    System.out.println("333");
}
```

可以发现，会先根据 hashCode 找出原始 switch 内的位置，再执行原代码逻辑。

对于第一个switch，可以看到，即使是用hashcode来直接定位到对应的case，也还是要判断一次`equals`。这样是不是会显得多此一举呢？

答案是否定的。因为hashcode是存在碰撞的可能性的。比如上述中的`"AaBB"`和`"AaAa"`的hashcode就是一样的，所以还需要用equals来区分判断。

先使用了hashcode来判断的好处，就是可以减少equals的次数，如果不用hashcode，那么就需要从第一个case开始一直往下一个个equals，显然效率较低。

### 为什么用两个switch

是为了减少编译器的工作。

比如 switch 内有的 case 不写 break 等复杂情况，如果想直接根据 hashCode + equals 来只生成一个 switch，编译器就需要考虑各种情况。

所以目前编译器只做位置映射，第二部分直接按原 switch 来生成了。

## &和&&的区别

&有两种用法：按位与，逻辑与。&&是短路与。短路与只要&&左边表达式的运算结果是false，就不会再继续运算&&右边的表达式。

|和||也是类似的。

## ++i与i++的区别

● i++：先赋值，后计算<br>
● ++i：先计算，后赋值

## Math.round(11.5) 等于多少？Math.round(-11.5)等于多少？

Math.round(11.5)的返回值是12，Math.round(-11.5)的返回值是-11。**四舍五入的原理是在参数上加0.5然后进行下取整。**

## 用最有效率的方法计算2乘以8？

2 << 3（左移3位相当于乘以2的3次方，右移3位相当于除以2的3次方）。

## Comparable和Comparator区别

**Comparable是排序接口，实现该接口需要重写`compareTo`方法。**实现该接口的对象，可以通过`compareTo`方法进行比较大小。

```java
public int compareTo(T o);
```

使用`Collections.sort`或`Arrays.sort`对Comparable的实现类进行排序时，或者将Comparable的实现类作为SortedMap、SortedSet的key时，无需指定Comparator比较器就能完成排序。

**Comparator是比较器接口，实现该接口需要重写`compare`方法，`equals`方法可以不重写。**

```java
int compare(T o1, T o2);

boolean equals(Object obj);
```

如果一个对象没有实现Comparable接口，则可以通过指定一个Comparator实现类作为比较器来为其进行排序。从Collections的两个sort方法的泛型和参数就可以看出来差别：

```java
public static <T extends Comparable<? super T>> void sort(List<T> list) {
    list.sort(null);
}

public static <T> void sort(List<T> list, Comparator<? super T> c) {
    list.sort(c);
}
```

## 参考链接

* [为什么switch不支持long](https://www.cnblogs.com/eycuii/p/11470950.html)
* [java三元运算符详解](https://www.cnblogs.com/itmlt1029/p/4756331.html)
* [Java中三元运算符值得注意的地方](https://blog.csdn.net/shikaiwencn/article/details/50443495)