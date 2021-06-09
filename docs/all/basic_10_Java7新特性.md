<!--
date: 2021-04-19T22:34:12+08:00
lastmod: 2021-06-08T22:34:12+08:00
-->
## Java7新特性（New highlights）

## 二进制字面值（Binary Literals）

整型类型（byte、char、short、int、long）可以用二进制字面量表示，使用二进制字面量时要加上`ob`或者`oB`前缀。相比十六进制`0x`，二进制字面量会更加直观。

```java
byte a = 0b00000011;    // 3
byte a2 = 0B00000011;   // 3    
```

## 数值中支持_（Underscores in Numeric Literals）

允许在数值字面量中添加`_`来提供可读性，但是以下四种情况是不允许添加`_`的：

● 在数字的开头或结尾<br>
● 在小数点前后<br>
● 在F/f、L/l的前面<br>
● 在进制标识符之间<br>

```java
// 不可以
float pi1 = 3_.1415F;
float pi2 = 3._1415F;
long socialSecurityNumber1 = 999_99_9999_L;
int x1 = _52;
int x3 = 52_;
int x5 = 0_x52;
int x11 = 052_;
int x6 = 0x_52;
int x8 = 0x52_;

// 可以
int x2 = 5_2;
int x4 = 5_______2;
int x7 = 0x5_2;
int x9 = 0_52;
int x10 = 05_2;
```

##  switch支持String类型（Strings in Switch Statement）

Java7之前只支持char、byte、short、int、Character、Byte、Short、Integer、String、enum类型，从Java7开始，switch支持String，编译器会根据字符串的hashCode来处理。

* [switch为什么可以支持 String？](http://localhost:3000/#/basic/%E8%BF%90%E7%AE%97?id=%e4%b8%ba%e4%bb%80%e4%b9%88%e5%8f%af%e4%bb%a5%e6%94%af%e6%8c%81-string%ef%bc%9f)

## 推断泛型类型参数（Type Inference for Generic Instance Creation，也叫Diamond Syntax）

只要编译器可以从上下文中推断出类型参数，就可以用一对空着的尖括号`<>`来代替泛型参数。因为缩短的尖括号看起来像钻石，所以又叫钻石语法（Diamond Syntax）：

```java
//java7之后
Map<String, List<String>> myMap = new HashMap<>();
//java7之前
Map<String, List<String>> myMap = new HashMap<String, List<String>>();
```

>**The “diamond syntax” name**
>
>This form is called “diamond syntax” because, well, the shortened type information
looks like a diamond. The proper name in the proposal is “Improved Type Inference
for Generic Instance Creation,” which is a real mouthful and has ITIGIC as an acronym, which sounds stupid, so diamond syntax it is.

## try语句可以附带资源声明try-with-resources

一般在使用try-catch语句块时，会在finally块里释放资源，比如关闭数据库连接等。Java7提供了带资源声明的try语句，这样就无需在finally块里释放声明了的资源，无论是否发生异常，JVM会自动帮你释放资源。此外，在关闭资源时往往也需要再次try-catch，代码较为冗余。

任何实现了`java.lang.AutoCloseable`接口和`java.io.Closeable`接口的对象都可以使用try-with-resources。Closeable接口是jdk1.5引入的，AutoCloseable接口是jdk1.7引入的，Closeable继承了AutoCloseable。

try语句里可以声明多个资源对象，用分号隔开；如果只声明了一个资源对象时末尾可以不加分号。

```java
// 1.7之前
public String before(String path) {
    BufferedReader br = null;
    String result = "";
    try {
        br = new BufferedReader(new FileReader(path));
        result = br.readLine();
    } catch (Exception e) {
        // handle exception
    } finally {
        if (br != null) {
            try {
                br.close();
            } catch (IOException e) {
                // handle exception
            }
        }
    }
    return result;
}

// 1.7开始
public String after(String path) {
    String result = "";
    try (BufferedReader br = new BufferedReader(new FileReader(path))) {
        result = br.readLine();
    } catch (Exception e) {
        // handle exception
    }
    return result;
}
```

## 新增addSuppressed异常方法

有时候在finally块的语句会抛出异常，如果没有处理就会把原本发生的异常覆盖掉，在异常的顶级类`Throwable`中新增了`addSuppressed`方法，允许记录另一个异常，可以通过`getSuppressed`方法来获取这个被记录的异常。这两个方法都是线程安全的，且`addSuppressed`方法会自动被`try-with-resources`所调用。

这样做的好处是不会丢失任何异常，方便开发人员进行调试

```java
public String after(String path) {
    BufferedReader br = null;
    String result = "";
    Exception originalException = null;
    try {
        br = new BufferedReader(new FileReader(path));
        result = br.readLine();
    } catch (Exception e) {
        originalException = e;
    } finally {
        if (br != null) {
            try {
                br.close();
            } catch (IOException e) {
                if (originalException != null) {
                    // 这样就能记录finally块中发生的异常
                    originalException.addSuppressed(e);
                }
            }
        }
    }
    return result;
}
```

## 捕获多个异常（Catching Multiple Exception Types）

一个catch块可以捕获多个异常，多个异常之间用`|`分隔。

```java
// 1.7之前
public void before() {
    try {
        int a = 1/0;
        throw new IOException();
    } catch (RuntimeException e) {
        // handle exception
    } catch (IOException e) {
        // handle exception
        e = new IOException();
    }
}

// 1.7开始
public void after() {
    try {
        int a = 1/0;
        throw new IOException();
    } catch (RuntimeException | IOException e) {
        // handle exception
        if (e instanceof IOException) {
            // 编译报错
            e = new IOException();  // The parameter e of a multi-catch block cannot be assigned
        }
    }
}
```

**注意，如果一个catch处理了多个异常，那么这个catch的参数默认就是final的，不能在catch块里修改它的值。 另外，用一个catch处理多个异常，比用多个catch块处理异常生成的字节码要更小更高效。**

## 增加类型检查后的重抛 （Rethrowing Exceptions with Improved Type Checking）

`throws`声明异常抛出时可以抛出更加具体的异常类型，在Java1.7之前只能声明抛出`Excetpion`。

```java
// 1.7之前
public void before() throws Exception {
    try {
        int a = 1 / 0;
        throw new IOException();
    } catch (RuntimeException e) {
        throw e;
    } catch (IOException e) {
        throw e;
    }
}

// 1.7开始
public void after() throws RuntimeException, IOException {
    try {
        int a = 1 / 0;
        throw new IOException();
    } catch (RuntimeException | IOException e) {
        throw e;
    }
}
```

## 其他的一些新特性

● 支持在JVM上运行动态类型语言，在字节码层面支持了`InvokeDynamic`（Support for Dynamic Languages）<br>
● 逐渐去永久代化，字符串常量池和静态变量被转移到堆中。<br>
● 提供了ForkJoin框架，用于提供并行执行任务，是一个把大任务分割成若干个小任务，最终汇总每个小任务结果后得到大任务结果的框架。（Java8的parallelStream跟这个框架有关）<br>
● 提供了新的I/O API（Java nio Package）

* [nio](/io/nio)

## 参考链接

* [十一、特性](http://www.cyc2018.xyz/Java/Java%20%E5%9F%BA%E7%A1%80.html#十一、特性)
* [Java SE 7新增特效](https://blog.csdn.net/u012104435/article/details/50971374)
* [Java7的新特性](https://segmentfault.com/a/1190000004417830)
* [Java 避免异常屏蔽 - addSuppressed](https://www.cnblogs.com/hbbbs/articles/11868942.html)
