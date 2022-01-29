<!--
date: 2021-06-29T22:46:12+08:00
lastmod: 2022-01-29T22:46:12+08:00
-->
## 线程与进程

1）进程，每个进程都有独立的代码和数据空间，切换进程开销大<br>
2）线程，同一类的线程共享代码和数据空间，单个线程有独立的运行权和程序计数器(`program counter`)，切换线程开销小<br>
3）进程和线程都是描述CPU工作的时间段，线程是粒度更细小的时间段。

## java的线程

jvm启动时会创建一个`main`线程来执行`public static void main(String[] args)`方法里面的代码

java的线程机制是抢占式的，这表示调度机制会周期性中断线程，将上下文切换到另一个线程，从而为每个线程都提供处理器时间片来驱动

## 线程的四种使用方式

### 继承Thread类

```java
// 定义一个线程
public class MyThread extends Thread {
    public void run() {
        // ...
    }
}

// 执行该线程
public static void main(String[] args) {
    MyThread mt = new MyThread();
    mt.start();
}
```

### 实现Runnable接口

```java
// 定义一个可执行任务
public class MyRunnable implements Runnable {
    @Override
    public void run() {
        // ...
    }
}

// 执行该任务
public static void main(String[] args) {
    MyRunnable instance = new MyRunnable();
    Thread thread = new Thread(instance);
    thread.start();
}
```

### 实现Callable接口

```java
// 定义一个可执行任务
public class MyCallable implements Callable<String> {
    public String call() {
        return "callable";
    }
}

// 执行该任务
public static void main(String[] args) throws ExecutionException, InterruptedException {
    MyCallable mc = new MyCallable();
    FutureTask<Integer> ft = new FutureTask<>(mc);
    Thread thread = new Thread(ft);
    thread.start();
    System.out.println(ft.get());
}
```

### 使用ExecutorService

前三种方式都是手动一个个创建线程来执行定义的任务，ExecutorService则是JDK自身提供的线程池框架，通过该框架来使用多线程以达到复用线程的目的。

## Thread、Runnable和Callable区别

实现接口的方式并不是真正定义了一个线程，而是定义了一个可以被Thread类运行的任务，通过Thread的构造器传递给Thread对象以调用。Thread类本身实现了Runnable接口，Thread对象通过调用`start()`方法来启动线程，并执行对应的任务（即调用重写的`run()`方法）：

```java
public class Thread implements Runnable {
    /* What will be run. */
    private Runnable target;

    public Thread(Runnable target) {
        init(null, target, "Thread-" + nextThreadNum(), 0);
    }
}
```

1）实现Runnable接口需要重写run方法，实现Callable接口需要重写call方法。<br>
2）Callable可以有返回值，返回值通过FutureTask进行封装。获取执行结果的get方法是一个阻塞的方法，直到任务执行完毕才能获取到结果。<br>
3) Callable会抛出异常，而Runnable只能在内部处理异常，不能继续往外抛。

```java
public static void main(String[] args) throws Exception {
    FutureTask<String> task = new FutureTask<>(() -> "Done.");
    new Thread(task).start();
    System.out.println(task.get());
}
```

一般使用实现接口的方式来使用线程：

1）Java不支持多重继承，实现接口的方式更加灵活。<br>
2）通常只需要定义一个可执行任务即可，然后通过线程池来执行这些任务，而直接继承Thread的方式开销较大。

## 基础线程机制



## this逃逸

在构造器返回之前，即在一个对象还没初始化之前就返回当前对象的this引用，即为this逃逸。

在单线程情况下，this逃逸不会产生问题。但若在多线程场景下，其他线程可以通过this逃逸来提前获取到还未完全初始化的对象，此时可能造成意想不到的问题。比如其他线程访问到尚未初始化的成员变量，可能读取到错误的值，或者是产生NullPointerException等。

可能发生this逃逸的情况：

1）构造器内部对成员变量进行赋值，并显式抛出this引用，此时可能发生重排序；<br>
2）构造器内部创建了匿名内部类对象并抛出其引用，因为内部类天然持有着外部类对象的引用，此时相当于隐式地抛出了this引用。

* [this引用逃逸问题](https://blog.csdn.net/qq_33804730/article/details/79451670)


## 参考链接

* [Java 并发](http://www.cyc2018.xyz/Java/Java%20%E5%B9%B6%E5%8F%91.html)