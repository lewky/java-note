<!--
date: 2021-04-19T22:34:12+08:00
lastmod: 2021-05-07T22:34:12+08:00
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
* 方法引用本质也是为了简写匿名内部类，被它实现的接口中的抽象方法的形参列表和返回值类型，要与方法引用的方法的形参列表和返回值类型相同
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

        // 静态方法引用
        test = System.out::println;
        test = Test::staticMethod;

        // 特定类的任意对象的实例方法引用
        ArrayList<Test> list = new ArrayList<>();
        list.forEach(Test::notStaticMethod);

        // 特定对象的实例方法引用
        Test instance = new Test();
        test = instance::notStaticMethod;

        // 超类上的实例方法引用
        test = super::notStaticFatherMethod;

        // 编译报错，但奇怪的是Lambda表达式却不会报错
        test = super::staticFatherMethod;  // The method staticFatherMethod() from the type Father should be accessed in a static way
        test = () -> super.staticFatherMethod();

    }

}

class Father {

    protected void notStaticFatherMethod() {

    }

    protected static void staticFatherMethod() {

    }
}
```

## 默认方法（Default Methods）


## 集合的Stream API（Streams）


## 新的日期和时间API（Date and Time API）


## Optional容器类


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
* [Java Lambda 表达式](https://www.runoob.com/java/java8-lambda-expressions.html)
