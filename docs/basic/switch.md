<!--
date: 2021-03-29T23:31:12+08:00
lastmod: 2021-03-29T23:31:12+08:00
-->
## switch

### 为什么switch不支持long

在 Java 语言规范里中，有说明 switch 支持的类型有：char、byte、short、int、Character、Byte、Short、Integer、String、enum。

为什么只支持上面几种？int、String 都可以，为什么不支持 long ？

原因就是 switch 对应的 JVM 字节码 `lookupswitch`、`tableswitch` 指令**只支持 int 类型**。

下面是 JVM 规范中的说明（https://docs.oracle.com/javase/specs/jvms/se8/html/jvms-3.html#jvms-3.10）：

>The Java Virtual Machine's `tableswitch` and `lookupswitch` instructions operate only on `int` data. Because operations on `byte`, `char`, or `short` values are internally promoted to `int`, a `switch` whose expression evaluates to one of those types is compiled as though it evaluated to type `int`. If the `chooseNear` method had been written using type `short`, the same Java Virtual Machine instructions would have been generated as when using type `int`. Other numeric types must be narrowed to type `int` for use in a `switch`.

byte、char、short 类型在编译期默认提升为 int，并使用 int 类型的字节码指令。所以对这些类型使用 switch，其实跟 int 类型是一样的。

### 为什么可以支持 String？

switch 支持 String 其实就是语法糖。编译器会根据字符串的 hashCode 来处理。

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

### 为什么用两个 switch ？

是为了减少编译器的工作。

比如 switch 内有的 case 不写 break 等复杂情况，如果想直接根据 hashCode + equals 来只生成一个 switch，编译器就需要考虑各种情况。

所以目前编译器只做位置映射，第二部分直接按原 switch 来生成了。

## 参考链接

* [为什么switch不支持long](https://www.cnblogs.com/eycuii/p/11470950.html)