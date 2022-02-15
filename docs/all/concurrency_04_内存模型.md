<!--
date: 2022-02-13T22:46:12+08:00
lastmod: 2022-02-15T22:46:12+08:00
-->
## Java内存模型（Java Memory Model，JMM）

JVM中试图定义一种JMM来屏蔽各种硬件和操作系统的内存访问差异，以实现让Java程序在各平台下都能达到一致的内存访问效果。Java内存模型是一种虚拟机规范，跟JVM内存分区不是一回事。

## 计算机的存储器

在了解JMM之前先简单了解下计算机中的各种存储器概念。存储器是计算机的重要组成部分，用于存储程序与数据，可分为：计算机内部的存储器（内存储器，简称内存）、计算机外部的存储器（外存储器，简称外存）。

## 计算机的内存、主存

内存和外存是一对相对而言的概念，主存和辅存也是如此。

一般来说，主存指的是内存；但是在一些专业性较强的场合，主存与内存还是有一定区别的。内存储器还有其他形式。而CPU中的存储器和主存是两个概念。处理器需要自己的内存储器，它们以寄存器的形式存在。

内存是CPU能直接寻址的存储空间，它的特点是存取速率快。内存一般采用半导体存储单元，包括随机存储器（RAM）、只读存储器（ROM）和高级缓存（Cache）。

### 随机存储器RAM（Random Access Memory）

高速存取，支持读写数据，读写时间相等，且与地址无关，但是断电后其中的数据会丢失。

### 只读存储器ROM（Read Only Memory）

断电后信息不丢失，如计算机启动用的BIOS芯片。存取速度很低（较RAM而言），且不能改写。由于不能改写信息，不能升级，现已很少使用。

### 高速缓存Cache

介于CPU与内存之间，常用有一级缓存（L1）、二级缓存（L2）、三级缓存（L3）（一般存在于Intel系列）。

它的读写速度比内存还快，当CPU在内存中读取或写入数据时，数据会被保存在高速缓冲存储器中，当下次访问该数据时，CPU直接读取高速缓冲存储器，而不是更慢的内存。

## 计算机的外存、辅存

外存指的是辅存，指除计算机内存及CPU缓存以外的储存器，此类储存器一般断电后仍然能保存数据。外存需要通过I/O系统与之交换数据，又称为辅助存储器。常见的外储存器有硬盘、U盘、光盘及软盘等。

## JMM中的主内存与工作内存

由于处理器上的寄存器的读写速度比内存快几个数量级，为了解决这种速度矛盾，在CPU和主存间加入了高速缓存。

但是高速缓存也带来了新的问题：缓存一致性。如果多个缓存共享同一块主内存区域，那么多个缓存的数据可能会不一致，需要一些协议来解决这个问题。

JVM定义了一种Java内存模型，将所有的变量都存储在主内存中（计算机的内存，所有线程共享的），每个线程都有自己的工作内存（线程私有的）。而工作内存就存储在高速缓存或者寄存器中，用来保存该线程使用的变量的主内存副本拷贝，即变量副本。

线程只能直接操作工作内存中的变量，不同线程之间的变量值传递需要通过主内存来完成。

```java
Thread1 <--------> 工作内存1 <-------->   -------
                                        |  Load |
Thread2 <--------> 工作内存2 <-------->  |   &   | <--------> 主内存
                                        | Store |
Thread3 <--------> 工作内存3 <-------->   -------
```

线程间通信必须经过主内存：

1）线程A把本地内存A中更新过的共享变量刷新到主内存中<br>
2）线程B到主内存中去读取线程A之前已更新过的共享变量

## 内存间的交互操作

JMM定义了8个操作来完成主内存和工作内存的交互操作：

● **lock（锁定）**：作用于主内存的变量，把一个变量标识为一条线程独占状态。<br>
● **unlock（解锁）**：作用于主内存变量，把一个处于锁定状态的变量释放出来，释放后的变量才可以被其他线程锁定。<br>
● **read（读取）**：作用于主内存变量，把一个变量值从主内存传输到线程的工作内存中，以便随后的load动作使用。<br>
● **load（载入）**：作用于工作内存的变量，它把read操作从主内存中得到的变量值放入工作内存的变量副本中。<br>
● **use（使用）**：作用于工作内存的变量，把工作内存中的一个变量值传递给执行引擎，每当虚拟机遇到一个需要使用变量的值的字节码指令时将会执行这个操作。<br>
● **assign（赋值）**：作用于工作内存的变量，它把一个从执行引擎接收到的值赋值给工作内存的变量，每当虚拟机遇到一个给变量赋值的字节码指令时执行这个操作。<br>
● **store（存储）**：作用于工作内存的变量，把工作内存中的一个变量的值传送到主内存中，以便随后的write的操作。<br>
● **write（写入）**：作用于主内存的变量，它把store操作从工作内存中一个变量的值传送到主内存的变量中。

**JMM还规定了在执行上述八种基本操作时，必须满足如下规则：**

● 如果要把一个变量从主内存中复制到工作内存，就需要**按顺序执行read和load操作**，如果把变量从工作内存中同步回主内存中，就要**按顺序地执行store和write操作**。但JMM只要求上述操作必须按顺序执行，而**没有保证必须是连续执行**。<br>
● **不允许read和load、store和write操作之一单独出现**。<br>
● 不允许一个线程丢弃它的最近assign的操作，即**变量在工作内存中改变了之后必须同步到主内存中**。<br>
● 不允许一个线程无原因地（没有发生过任何assign操作）把数据从工作内存同步回主内存中。<br>
● **一个新的变量只能在主内存中诞生**，不允许在工作内存中直接使用一个未被初始化（load或assign）的变量。即就是对一个变量实施use和store操作之前，必须先执行过了assign和load操作。<br>
● 一个变量在同一时刻只允许一条线程对其进行lock操作，但lock操作可以被同一条线程重复执行多次，多次执行lock后，只有执行相同次数的unlock操作，变量才会被解锁。lock和unlock必须成对出现。<br>
● 如果对一个变量执行lock操作，将会清空工作内存中此变量的值，在执行引擎使用这个变量前需要重新执行load或assign操作初始化变量的值。<br>
● 如果一个变量事先没有被lock操作锁定，则不允许对它执行unlock操作；也不允许去unlock一个被其他线程锁定的变量。<br>
● **对一个变量执行unlock操作之前，必须先把此变量同步到主内存中**（执行store和write操作）。

## JMM三大特性

### 原子性

JMM保证了read、load、use、assign、store、write、lock、unlock这八个操作具有原子性，但JMM允许虚拟机将没有被volatile修饰的64位数据（long，double）的读写操作划分为两次32位的操作来进行，即load、store、read和write操作可以不具备原子性。

在单线程场景下，以上原子性操作是线程安全的，但到了多线程环境中则未必，如下对一个int进行自增：

```java
public class Test {

    static int cnt = 0;

    public static void main(final String[] args) {
        final Runnable runnable = () -> {
            for (int i = 0; i < 100; i++) {
                cnt++;
                try {
                    Thread.currentThread().sleep(10);
                } catch (final InterruptedException e) {
                    e.printStackTrace();
                }
            }
        };
        final Thread thread1 = new Thread(runnable);
        final Thread thread2 = new Thread(runnable);
        thread1.start();
        thread2.start();
        try {
            Thread.currentThread().sleep(3000);
        } catch (final InterruptedException e) {
            e.printStackTrace();
        }
        // 192，而不是200
        System.out.println(cnt);
    }

}
```

从上面代码实例可以看到，在多线程环境下，不同线程读取到的公共变量值不一定是最新的值，这说明即使单个操作是原子的，但一组读取赋值操作却并非原子的。

JDK提供的原子类可以避免这个问题，将上述代码中的cnt变量改为AtomicInteger类型，如下：

```java
public class Test {

    static AtomicInteger cnt = new AtomicInteger();

    public static void main(final String[] args) {
        final Runnable runnable = () -> {
            for (int i = 0; i < 100; i++) {
				// 使用原子类的自增方法来替代自增运算符
                cnt.incrementAndGet();
                try {
                    Thread.currentThread().sleep(10);
                } catch (final InterruptedException e) {
                    e.printStackTrace();
                }
            }
        };
        final Thread thread1 = new Thread(runnable);
        final Thread thread2 = new Thread(runnable);
        thread1.start();
        thread2.start();
        try {
            Thread.currentThread().sleep(3000);
        } catch (final InterruptedException e) {
            e.printStackTrace();
        }
        // 200
        System.out.println(cnt);
    }

}
```

除了使用原子类，也可以直接用synchronized关键字来进行同步，以保证操作的原子性。它对应的内存间交互操作为：lock和unlock，**在虚拟机实现上对应的字节码指令为monitorenter和monitorexit**。

```java
public class Test {

    static int cnt = 0;

    public static void main(final String[] args) {
        final Runnable runnable = () -> {
            for (int i = 0; i < 100; i++) {
				// 在同步块中进行自增操作
                synchronized (Test.class) {
                    cnt++;
                }
                try {
                    Thread.currentThread().sleep(10);
                } catch (final InterruptedException e) {
                    e.printStackTrace();
                }
            }
        };
        final Thread thread1 = new Thread(runnable);
        final Thread thread2 = new Thread(runnable);
        thread1.start();
        thread2.start();
        try {
            Thread.currentThread().sleep(3000);
        } catch (final InterruptedException e) {
            e.printStackTrace();
        }
        // 200
        System.out.println(cnt);
    }

}
```

### 可见性

可见性指当一个线程修改了共享变量的值，其它线程能够立即得知这个修改。JMM是通过在变量修改后将新值同步回主内存，在变量读取前从主内存刷新变量值来实现可见性的。

可见性主要有以下三种实现方式：

1）volatile<br>
2）synchronized<br>
3）final

### 有序性

## volatile



## AQS（AbstractQueuedSynchronizer）



## 参考链接

* [Java 并发](http://www.cyc2018.xyz/Java/Java%20%E5%B9%B6%E5%8F%91.html)
* [区分内存、外存、主存、辅存等](https://blog.51cto.com/u_10739931/1698313)
* [Java内存模型（JMM）总结](https://zhuanlan.zhihu.com/p/29881777)