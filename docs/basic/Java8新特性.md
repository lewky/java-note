<!--
date: 2021-04-19T22:34:12+08:00
lastmod: 2021-05-11T22:34:12+08:00
-->
## Java8新特性

Java8新增了很多新特性，这里只介绍比较重要的几个新特性。

## Lambda表达式(Lambda Expressions)

Lambda表达式，也可称为闭包，是Java8最重要的特性。它允许把方法作为方法的参数来传递，使代码更加简洁紧凑。

Lambda表达式语法格式如下：

```java
(parameters) -> expression
或
(parameters) ->{ statements; }
```

* 可选类型声明：不需要声明参数类型，编译器可以统一识别参数值。
* 可选的参数圆括号：一个参数无需定义圆括号，但多个参数需要定义圆括号，没有参数时也要定义一个圆括号。
* 可选的大括号：如果主体包含了一个语句，就不需要使用大括号，此时不能在语句末尾使用分号（此时是作为表达式，而不是一个完整的语句）。反过来说，如果使用了大括号，则必须在语句末尾使用分号。
可选的返回关键字：如果主体只有一个表达式返回值则编译器会自动返回值，大括号需要指定明表达式返回了一个数值。

```java
public class Test {

    interface MathOperation {
        int operation(int a, int b);
    }

    private static int operate(int a, int b, MathOperation mathOperation) {
        return mathOperation.operation(a, b);
    }

    public static void main(String[] args) throws Exception {
        // 声明参数类型
        MathOperation addition = (int a, int b) -> a + b;

        // 不声明参数类型
        MathOperation subtraction = (a, b) -> a - b;

        // 使用大括号
        MathOperation subtmultiplication = (a, b) -> { return a * b; };

        System.out.println(Test.operate(3, 4, addition));           // 7
        System.out.println(Test.operate(3, 4, subtraction));        // -1
        System.out.println(Test.operate(3, 4, subtmultiplication)); // 12
    }
}
```

**注意：**
* Lambda表达式相当于简写了实现接口的匿名内部类，但是被实现的接口只能有一个抽象方法，这种接口被称为函数式接口（Functional Interface）
* 可以在接口上添加`@FunctionalInterface`来表明一个函数式接口，这样编译器会帮你自动检查当前接口是否只有一个抽象方法
* Lambda表达式如果需要访问**外部局部变量**，则该局部变量必须是final的或者effectively final的，因为Lambda表达式本质上就是一个只存在于内存中的匿名内部类。
	* [为什么局部内部类和匿名内部类只能访问局部final变量](/basic/关键字?id=为什么局部内部类和匿名内部类只能访问局部final变量)
* Lambda表达式当中不允许声明一个与局部变量同名的参数或者局部变量，原因同上。

## 方法引用（Method Reference）

方法引用就是通过方法的名字来指向一个方法，是对Lambda表达式的进一步简写。但不是所有的Lambda表达式都能简写为方法引用，需要满足三个条件：
* 方法引用本质也是为了简写匿名内部类，被它实现的**接口中的抽象方法的形参列表和返回值类型，要与方法引用的方法的形参列表和返回值类型相同**
* 方法参数不能被Lambda表达式改动，必须原封不动地作为参数被传递给另一个方法
* Lambda表达式只有一行语句

方法引用使用双冒号`::`表示，一共有以下几种用法：

```java
public class Test extends Father {

    interface TestMethodReference {
        void test();
    }

    void notStaticMethod() {

    }

    static void staticMethod() {

    }

    protected void test() {
        // 构造器引用
        TestMethodReference test = Test::new;
        test = ArrayList<Test>::new;

        // 数组构造器引用，Function是Java8提供的一个函数式接口
        Function<Integer, Test[]> function = Test[]::new;

        // 静态方法引用，只能用类名::方法名
        test = Test::staticMethod;

        // 特定类的任意对象的实例方法引用，Consumer是Java8提供的一个函数式接口
        Consumer<Test> conumer = Test::notStaticMethod;

        // 特定对象的实例方法引用
        Test instance = new Test();
        test = instance::notStaticMethod;

        // 超类上的实例方法引用
        test = super::notStaticFatherMethod;
    }

}

class Father {
    protected void notStaticFatherMethod() {

    }
}
```

## 默认方法（Default Methods）

### 语法

默认方法就是接口可以有实现方法，而且不需要实现类去实现其方法，在方法名前面加上`default`关键字即可实现默认方法。引入默认方法的目的是为了解决接口的修改与Java旧版本不兼容的问题，因为已经发布的版本的接口无法更改，而新版本又在接口中引入了新的方法。

```java
interface TestDefault {
    default void defaultMethod() {
        // do something
    }
}
```

### 多个默认方法

如果一个类实现了多个不同接口，这些接口又有相同的默认方法，如下：

```java
interface TestDefault1 {
    default void defaultMethod() {
        System.out.println(TestDefault1.class.getSimpleName());
    }
}

interface TestDefault2 {
    default void defaultMethod() {
        System.out.println(TestDefault2.class.getSimpleName());
    }
}

public class Test implements TestDefault1, TestDefault2 {
    @Override
    public void defaultMethod() {
        TestDefault1.super.defaultMethod();
    }
}
```

此时实现类必须重写接口中重复的默认方法，只能使用public修饰符，default方法只能在接口中使用。重写时可以通过`Class.super.xxx()`来指定调用哪个接口中的默认方法。

### 静态默认方法

Java8的接口中还可以定义静态的默认方法，但此时只能使用static关键字，不能和default一起使用，且必须要有对应的方法体：

```java
interface TestDefault {
    static void staticMethod() {
        // do something
    }
}
```

## 集合的Stream API（Streams）

集合框架提供了一种新的处理元素的Stream API，将需要被处理的集合看作一种流，流在管道中传输，并可以在管道的节点上进行处理，比如过滤、排序、聚合等。

元素流在管道中经过中间操作（intermediate operation）的处理，最后由最终操作(terminal operation)得到前面处理的结果。中间操作是惰性的（lazy），每个中间操作都会返回一个新的流。中间操作不会真的去遍历集合，只有遇到最终操作才会开始真正的遍历集合执行计算。一个流可以经过若干个中间操作，但只能有一个最终操作，经过最终操作后流中的元素就被用完了，会返回计算的结果（也可能返回的是void）。

注意，Stream流属于管道流，只能被消费一次。一个Stream流经过一次中间操作后，数据就被转移到下一个流，原本的流就会关闭。一个流只能被操作一次，重复操作同一个流会抛异常`java.lang.IllegalStateException: stream has already been operated upon or closed`。这也是为什么每个中间操作都会返回新的流对象，同时也便于链式调用。

Stream的数据源除了可以是集合外，也可以是数组，I/O channel，产生器generator等。平时集合用的比较多，这里只简单介绍下集合的Stream API。

集合在Java8之前通过Iterator或者For-Each的方式，显示地在集合外部进行迭代。而Stream提供了内部迭代的方式，通过访问者模式（Visitor）实现。

### 生成流

有两种方式生成流：
* `stream()`生成串行流，用于串行处理操作
* `parallelStream()`生成并行流，用于并行处理操作

### forEach

集合的Stream提供了新的内部迭代方式forEach，这是一个最终操作：

```java
public static void main(final String[] args) throws CloneNotSupportedException {
    final List<String> list = Arrays.asList("a", "b", "c");
    Consumer<String> consumer = System.out::println;
    consumer = consumer.andThen(string -> { string = string + string; System.out.println(string); });
    list.stream().forEach(consumer);
}
```

输出如下：

```java
a
aa
b
bb
c
cc
```

### map、flatMap

map将Stream中元素映射为另一个元素，原本有多少个元素，返回的流中就有多少个元素。比如说原本元素是`Stream<T>`，那么map会返回`Stream<R>`。

flatMap则是将Stream的元素压缩到同一个Stream中，并且只有当Stream中元素是一个Stream时才能使用。原本的元素都是Stream，Stream中又有元素，flatMap会把里面的元素都拿出来，一起装到一个新的Stream并返回。比如说原本元素是`Stream<Stream<T>>`，那么flatMap会返回`Stream<R>`。如果原本元素是`Stream<Stream<Stream<T>>>`，那么flatMap会返回`Stream<Stream<R>>`。

```java
public static void main(final String[] args) throws CloneNotSupportedException {
    final List<String> list = Arrays.asList("a", "b", "c");
    // map
    list.stream()
        .map(each -> each + each)
        .forEach(System.out::println);

    // flatMap
    list.stream()
        .map(each -> Stream.of(each))
        .flatMap(each -> each)
        .forEach(System.out::println);
}
```

结果如下：

```java
aa
bb
cc
a
b
c
```

### filter

用于过滤元素，当条件为true时才能被保留下来。

```java
public static void main(final String[] args) throws CloneNotSupportedException {
    final List<String> list = Arrays.asList("a", "b", "c");
    list.stream()
        .filter(each -> "a".equals(each))
        .forEach(System.out::println);
    // result: a
}
```

### limit

用于获取指定数量的流：

```java
new Random().ints().limit(3).forEach(System.out::println);

List<String> list = Arrays.asList("a", "b", "c");
list.stream()
    .limit(2)
    .forEach(System.out::print);
// result: ab
```

### sorted

对流元素进行顺序排序：

```java
list.stream().sorted().forEach(System.out::print);
// result: 239

list.stream()
    .sorted((a, b) -> b - a)
    .forEach(System.out::print);
// result: 932
```

### 并行操作

通过`parallelStream()`进行并行处理操作：

```java
public class Test {

    int seq;

    public Test(int seq) {
        this.seq = seq;
    }

    @Override
    public String toString() {
        return "Test" + seq;
    }

    public static void main(String[] args) throws Exception {
        // 系统本身支持的处理器数量，并不一定等于物理CPU核数，而是逻辑CPU数
        // 比如我的系统CPU是6核12线程，这里得到的就是12
        System.out.println(Runtime.getRuntime().availableProcessors());  // 12

        // 手动设置并行处理线程数量
//        System.setProperty("java.util.concurrent.ForkJoinPool.common.parallelism", "13");
        // 获取手动设置的并行处理数量，不设置时则为null
        System.out.println(System.getProperty("java.util.concurrent.ForkJoinPool.common.parallelism"));  // null

        // 获取并行处理的数量，等于逻辑CPU数量-1，或者是手动设置的并行处理数量
        // ForkJoinPool允许并行执行的任务数量为该commonParallelism + 1，这里commonParallelism是11，即允许最大并行任务数量为12
        System.out.println(ForkJoinPool.getCommonPoolParallelism());  // 11

        List<Test> list = new ArrayList<Test>();
        for (int i = 0; i < 12; i++) {
            list.add(new Test(i));
        }
        // stream
        long start = System.currentTimeMillis();
        list.stream().forEach(each -> {
            try {
                Thread.sleep(100);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.println(each);
        });
        long end = System.currentTimeMillis();
        System.out.println("stream cost:" + (end - start));

        // parallelStream
        start = System.currentTimeMillis();
        list.parallelStream().forEach(each -> {
            try {
                Thread.sleep(100);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.println(each);
        });
        end = System.currentTimeMillis();
        System.out.println("parallelStream cost:" + (end - start));
    }

}
```

下面是执行结果：

```java
12
null
11
Test0
Test1
Test2
Test3
Test4
Test5
Test6
Test7
Test8
Test9
Test10
Test11
stream cost:1338
Test1
Test2
Test10
Test8
Test0
Test9
Test4
Test7
Test6
Test5
Test3
Test11
parallelStream cost:109
```

可以看出，parallelStream快了很多，输出的结果也不是串行的。

parallelStream使用了Java7引入的并行执行框架`ForkJoin`和`ForkJoinPool`，ForkJoinPool维护了一个静态的通用线程池`static final ForkJoinPool common`。该线程池有一个`parallelism`变量，如果不手动设置该值，则默认等于运行计算机上的逻辑处理器数量-1。比如系统CPU是6核12线程，那么默认值就是11，该数量可以用`Runtime.getRuntime().availableProcessors()`获取。

也可以通过设置启动参数`java.util.concurrent.ForkJoinPool.common.parallelism`来控制parallelism的值。ForkJoinPool的最大并行线程数量等于`parallelism + 1`。

>* 使用parallelStream可以简洁高效的写出并发代码。
>* parallelStream并行执行是无序的。
>* parallelStream提供了更简单的并发执行的实现，但并不意味着更高的性能，它是使用要根据具体的应用场景。如果cpu资源紧张parallelStream不会带来性能提升；如果存在频繁的线程切换反而会降低性能。
>* 任务之间最好是状态无关的，因为**parallelStream默认是非线程安全的**，可能带来结果的不确定性。

### Collectors

Collectors工具类提供了各种归约操作，可以将流转换为集合和聚合元素，这是一个最终操作，常用的有将流转换为集合、String等操作。

```java
List<String> list = Arrays.asList("a", "b", "cc");
// 返回ArrayList
List<String> collect = list.stream().filter(each -> each.length() > 1).collect(Collectors.toList());
System.out.println(collect instanceof ArrayList);   // true

// 返回一个String
String string = list.stream().collect(Collectors.joining());
System.out.println(string);  // abcc
```

### reduce

这里的reduce不是减少的意思，而是归纳、归约的意思，是一个最终操作，用于累计运算（迭代运算器）。

```java
List<Integer> list = Arrays.asList(1, 2, 3, 4, 5);
// 3 + (1 + 2 + 3 + 4 + 5) = 18
Integer result = list.stream().reduce(3, (a, b) -> a + b);
// 1 + 2 + 3 + 4 + 5 = 15
result = list.stream().reduce((a, b) -> a + b).get();
```

### peek

peek类似于forEach，但peek是一个中间操作。peek接收的是一个Consumer接口，用来对元素进行操作，且没有返回值，这意味着peek不会改变原本的元素，这是peek的map的区别。

所以peek可以用于debug，用来输出元素的信息，当然也可以直接改变元素内部的属性。

```java
public class Test {

    int seq;

    public Test(int seq) {
        this.seq = seq;
    }

    public void setSeq(int seq) {
        this.seq = seq;
    }

    public int getSeq() {
        return seq;
    }

    public static void main(String[] args) throws Exception {
        List<String> list = Arrays.asList("a", "b", "c");
        // result: abc
        list.stream().peek(each -> each.toUpperCase()).forEach(System.out::print);

        List<Test> list2 = Arrays.asList(new Test(1));
        // result: 10
        list2.stream().peek(each -> each.setSeq(10)).map(Test::getSeq).forEach(System.out::print);
    }

}
```

### 统计

除了常规的Stream外，还提供了各种包装类的Stream：IntStream、LongStream、DoubleStream。它们和Stream一样，都继承自BaseStream接口，但是额外提供了普通Stream没有的统计聚合功能。

```java
List<Integer> list = Arrays.asList(5, 29, 1, 42, 87, 19);

IntSummaryStatistics stat = list.stream().mapToInt(each -> each).summaryStatistics();
System.out.println(stat.getCount());    // 6
System.out.println(stat.getAverage());  // 30.5
System.out.println(stat.getMax());      // 87
System.out.println(stat.getMin());      // 1
System.out.println(stat.getSum());      // 183
```

### 用Stream实现斐波那契数列

`iterate`和`generate`都可以生成无限流，这里用的前者：

```java
String result = Stream.iterate(new int[] {0, 1}, array -> new int[] {array[1], array[0] + array[1]})
    .limit(10)
    .map(each -> String.valueOf(each[0]))
    .reduce((a, b) -> a + ", " + b).orElse("");
System.out.println(result);
// result: 0, 1, 1, 2, 3, 5, 8, 13, 21, 34
```

## 新的日期和时间API（Date and Time API）

旧版本的日期API存在以下问题：
>* **非线程安全。**所有日期类都是可变的，这是Java日期类最大的问题之一。
>* **设计很差。**在`java.util`和`java.sql`的包中都有Date类，前者同时包含日期和时间，后者仅包含日期。用于格式化和解析的类则定义在`java.text`包中。
>* **时区处理麻烦。**日期类并不提供国际化，没有时区支持，因此Java引入了`java.util.Calendar`和`java.util.TimeZone`类，但他们同样存在上述所有的问题。

Java8在`java.time`包提供了新的日期API：
>* **Local本地日期**：简化了日期时间的处理，没有时区的问题。
>* **Zoned时区日期**：通过指定的时区处理日期时间。

### 本地日期API

在不需要处理时区时使用：LocalDate、LocalTime、LocalDateTime

```java

```

### 时区日期API

```java

```

## Optional容器类

Optional是一个容器对象，内部最多存放一个元素，元素允许为null。Optional类的目的是尽可能减少系统中的NullPointerException。

```java
// of方法不允许参数为null，否则抛出NPE
Optional<String> optional = Optional.of("test");
// ofNullable允许参数为null，此时返回一个空的Optional对象
optional = Optional.ofNullable(null);
// 获取一个值为null的空容器
optional = Optional.empty();

// 是否存在值
System.out.println(optional.isPresent());   // false
// 返回容器中的值，如果值是null则返回指定的值
System.out.println(optional.orElse("test"));    // test
// 返回容器中的值，如果值是null则抛出NoSuchElementException
System.out.println(optional.get());
// 返回容器中的值，如果值是null则抛出指定的异常
System.out.println(optional.orElseThrow(RuntimeException::new));
```

## 类型注解（Type Annotations）

新增类型注解，类型注解无法作用于返回值是void的方法。

* [类型注解](/basic/%E6%B3%A8%E8%A7%A3?id=%e7%b1%bb%e5%9e%8b%e6%b3%a8%e8%a7%a3)

## 并行操作（Parallel operations）


## 提升HashMaps的性能



## 其他的一些新特性

* Nashhorn JavaScript引擎（Nashhorn JavaScript Engine）
	* [Java 8 Nashorn JavaScript](https://www.runoob.com/java/java8-nashorn-javascript.html) 
* 新的并发工具（Concurrent Accumulators）
* 取消永久代，但是新增了本地内存的元空间Metaspace，用以保存类型信息、字段、方法、常量

## 参考链接

* [Java 8 新特性](https://www.runoob.com/java/java8-new-features.html)
* [Java8的新特性](https://segmentfault.com/a/1190000004419611)
* [Java8 parallelStream浅析](https://zhuanlan.zhihu.com/p/43039062)
* [Stream流实现斐波那契数列](https://www.cnblogs.com/lishuaiqi/p/12006292.html)