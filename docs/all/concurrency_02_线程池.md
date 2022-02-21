<!--
date: 2022-01-29T22:46:12+08:00
lastmod: 2022-02-21T22:46:12+08:00
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
3）SingleThreadExecutor：只有单例线程，即大小为1的FixedThreadPool<br>
4）ScheduledThreadPoolExecutor：用于执行定时任务，线程池无限大。

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

### corePoolSize

线程池核心线程数，即线程池中最小的线程数量，即使线程处于空闲状态也不会被销毁，除非设置了allowCoreThreadTimeOut。

### maximumPoolSize

线程池最大线程数。一个任务被提交后会被缓存到工作队列，等工作队列满了，则会创建一个新的线程，从工作队列中取出一个任务来处理。

### keepAliveTime

当线程数量大于corePoolSize时，空闲线程会在指定时间后被销毁。

### unit

keepAliveTime的计量单位。

### workQueue

线程池所使用的缓冲队列，即工作队列，新任务被提交后会先进入工作队列，任务调度时再从工作队列中取出任务来执行。

JDK主要提供了以下几种工作队列：

1）ArrayBlockingQueue：基于数组的有界阻塞队列，按FIFO排序。<br>
2）LinkedBlockingQuene：基于链表的无界阻塞队列（其实最大容量为Interger.MAX），按照FIFO排序。FixedThreadPool和SingleThreadExecutor使用的就是这个队列。<br>
3）SynchronousQueue：一个不存储元素的阻塞队列。每个插入操作必须等到另一个线程调用移除操作，否则插入操作一直处于阻塞状态，吞吐量通常要高于LinkedBlockingQueue。CachedThreadPool使用的就是这个队列。<br>
4）PriorityBlockingQueue：具有优先级的无界阻塞队列，优先级通过参数Comparator实现。<br>
5）DelayedWorkQueue：具有优先级的无界阻塞队列，会按照延时的先后顺序来排序，若时间相同则根据sequenceNumber排序。

### threadFactory

线程池创建线程使用的工厂，可以用来设定线程名、是否为daemon线程等。

### handler

线程池拒绝任务时的处理策略，JDK提供了以下拒绝策略：

1）CallerRunsPolicy：在调用者线程中直接执行被拒绝任务的run方法，除非线程池已经shutdown，此时直接抛弃任务。<br>
2）AbortPolicy：直接丢弃任务，并抛出RejectedExecutionException。<br>
3）DiscardPolicy：直接丢弃任务，什么都不做。<br>
4）DiscardOldestPolicy：抛弃最早进入队列的那个任务，然后尝试把这次拒绝的任务放入队列。

## 参考链接

* [Java 并发](http://www.cyc2018.xyz/Java/Java%20%E5%B9%B6%E5%8F%91.html)
* [ScheduledThreadPoolExecutor](https://www.cnblogs.com/yufeng218/p/13211010.html)
* [Java线程池七个参数](https://www.cnblogs.com/shijianchuzhenzhi/p/12964678.html)