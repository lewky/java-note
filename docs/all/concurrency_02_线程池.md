<!--
date: 2022-01-29T22:46:12+08:00
lastmod: 2022-02-20T22:46:12+08:00
-->
## 线程池

创建和销毁线程的开销较大，可以通过池化技术来管理、复用线程对象以降低开销。（类比于数据库连接和连接池，是享元模式在Java中的应用实例）

好处：

1）降低资源消耗。通过复用已创建的线程对象来降低线程在创建和销毁时的开销。<br>
2）提高响应速度。当任务到达时，无需等待线程创建就能立即执行任务。<br>
3）提高线程的可管理性。可以对线程进行统一分配、调优和监控。

## Executor

JDK 1.5引入了Executor框架，其实现的正是线程池的功能。Java里线程池的顶级接口是Executor，但是严格意义上讲Executor并不是一个线程池，而只是一个执行线程的工具，真正的线程池接口是ExecutorService。

主要有如下三种线程池：

1）CachedThreadPool：一个任务创建一个线程，线程池无限大，会自动回收空闲线程<br>
2）FixedThreadPool：无论任务多少，只能重用固定数量的线程<br>
3）SingleThreadExecutor：只有单例线程，即大小为1的FixedThreadPool

这三种线程池实际上都是通过`new ThreadPoolExecutor()`来创建的，还有一种用于执行定时任务的线程池ScheduledThreadPoolExecutor，其继承自ThreadPoolExecutor。

可以手动创建不同的线程池对象，也可以通过Executors这个静态工厂类来创建，不过阿里规范不推荐使用该类来创建线程池，有可能导致线上OOM：[【不要使用Java Executors 提供的默认线程池】](https://www.cnblogs.com/ylty/p/11842727.html)

## ThreadPoolExecutor参数含义

ThreadPoolExecutor提供了多个构造方法，参数最齐全的如下：

```java
public ThreadPoolExecutor(int corePoolSize,
            　　　　　　　  int maximumPoolSize,
                           long keepAliveTime,
                           TimeUnit unit,
                           BlockingQueue<Runnable> workQueue,　　　　　　　　　　　　　　　　　　
						   ThreadFactory threadFactory,
                           RejectedExecutionHandler handler) {

}
```

这些参数的含义如下：

1）corePoolSize： 线程池核心线程数<br>
2）maximumPoolSize：线程池最大数<br>
3）keepAliveTime： 空闲线程存活时间<br>
4）unit： 时间单位<br>
5）workQueue： 线程池所使用的缓冲队列<br>
6）threadFactory：线程池创建线程使用的工厂<br>
7）handler： 线程池对拒绝任务的处理策略<br>

## 参考链接

* [Java 并发](http://www.cyc2018.xyz/Java/Java%20%E5%B9%B6%E5%8F%91.html)