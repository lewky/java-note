<!--
date: 2022-02-17T22:34:12+08:00
lastmod: 2022-02-17T22:34:12+08:00
-->
## 运行时数据区域

JVM，Java Virtual Machine，即Java虚拟机，是Java能够跨平台执行的关键，它不是真实的物理机，其运行时数据区域（即内存分区）如下图所示：

![jvm1.6](https://cdn.jsdelivr.net/gh/lewky/java-note@main/docs/static/jvm1.6.png)

程序计数器、虚拟机栈、本地方法栈这三块内存区域是线程私有的，即每个线程都会开辟私有的内存区域给这三个使用，而堆、方法区则是所有线程共享的内存区域。

### 程序计数器

记录正在执行的虚拟机字节码指令的地址（如果正在执行的是本地方法则为空）。

### Java虚拟机栈

也叫线程栈，方法栈。每个Java方法在执行的同时会创建一个栈帧用于存储局部变量表（Local Variable Array）、操作数栈（Operand Stack）、常量池引用（Reference to Constant Pool）等信息。从方法调用直至执行完成的过程，对应着**一个栈帧在Java虚拟机栈中入栈和出栈**的过程。

## 参考链接

* [Java 虚拟机](http://www.cyc2018.xyz/Java/Java%20%E8%99%9A%E6%8B%9F%E6%9C%BA.html)