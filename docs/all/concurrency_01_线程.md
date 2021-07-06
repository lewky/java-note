<!--
date: 2021-06-29T22:46:12+08:00
lastmod: 2021-07-06T22:46:12+08:00
-->
## 使用线程

线程有三种使用方式：

1）继承Thread类（Thread类本身实现了Runnable接口）<br>
2）实现Runnable接口<br>
3）实现Callable接口

实现接口的方式并不是真正定义了一个线程，而是定义了一个可以被Thread类运行的任务，通过Thread的构造器传递给Thread对象以调用。Thread对象通过调用`start()`方法来启动线程，并执行对应的任务（即调用重写的`run()`方法）。

```java
public class Thread implements Runnable {
    /* What will be run. */
    private Runnable target;

    public Thread(Runnable target) {
        init(null, target, "Thread-" + nextThreadNum(), 0);
    }
}
```

### Runnable和Callable区别

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

## 线程池

创建和销毁线程的开销较大，可以通过池化技术来管理、复用线程对象以降低开销。（类比于数据库连接和连接池）

好处：

1）降低资源消耗。通过复用已创建的线程对象来降低线程在创建和销毁时的开销。<br>
2）提高响应速度。当任务到达时，无需等待线程创建就能立即执行任务。<br>
3）提高线程的可管理性。可以对线程进行统一分配、调优和监控。

## Executor

JDK 1.5引入了Executor框架，其实现的正是线程池的功能。

### Executors



## 基础线程机制








## 参考链接

* [Java 并发](http://www.cyc2018.xyz/Java/Java%20%E5%B9%B6%E5%8F%91.html)