<!--
date: 2021-04-19T22:34:12+08:00
lastmod: 2021-06-08T22:34:12+08:00
-->
## 异常

异常的顶级类是`Throwable`，它有两个子类：`Error`和`Exception`。其中Error用来表示系统级的错误，不能通过程序来进行处理。Exception分为两种：

● 受检异常（CheckedException），即编译时异常。Java编译器会检查它，要么通过throws进行声明抛出，用么通过`try...catch...`进行捕获并处理，并且可以从异常中恢复。捕获异常语句通常和`finally`搭配使用，无论程序正常执行还是发生异常，finally代码块只要JVM不关闭都能执行，可以将释放外部资源的代码写在finally块中。如果`try{}`里有一个return语句，则会在return之前执行finally块。<br>
● 非受检异常，即运行时异常（RuntimeException）。运行时异常无需用`try...catch...`进行处理，但是一旦发生该类异常，此时程序崩溃并且无法恢复。

## 一些常见的运行时异常

● ArithmeticException（算术异常）<br>
● ClassCastException （类转换异常）<br>
● IllegalArgumentException （非法参数异常）<br>
● IndexOutOfBoundsException （下标越界异常）<br>
● NullPointerException （空指针异常）<br>
● SecurityException （安全异常）

## throw和throws

● throws声明异常抛出，当一个方法声明会抛出异常时，该方法的调用方需要对其进行异常处理，或者继续往外抛出该异常。<br>
● throw是手动抛出一个异常对象，抛出该异常的方法需要进行try-catch处理或者声明异常抛出。<br>
● 二者都是消极处理异常的方式，真正的异常处理交由调用方来完成。

## 参考链接

* [八、异常](http://www.cyc2018.xyz/Java/Java%20%E5%9F%BA%E7%A1%80.html#八、异常)
