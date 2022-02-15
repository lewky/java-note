<!--
date: 2022-02-12T22:46:12+08:00
lastmod: 2022-02-15T22:46:12+08:00
-->
## synchronized

synchronized是JVM内置的锁机制。

### 同步代码块

```java
public void func() {
    synchronized (this) {
        // ...
    }
}
```

同步代码块只会作用于同一个对象，上面代码是同步的this对象（也可以是同个类的其他对象或者其他类的对象）。当一个线程进入了该对象的同步代码块，另一个线程想进入该对象的同步代码块就必须等待。

### 同步一个类

```java
public void func() {
    synchronized (Test.class) {
        // ...
    }
}
```

作用于整个类，即多个线程调用同一个类的不同对象上的同步代码块，也会进行同步。

### 同步方法

```java
public synchronized void func() {
    // ...
}
```

和同步代码块一样作用于同一个对象，但同步代码块较为灵活，其同步的范围可以更小。

### 静态同步方法

```java
public synchronized static void func() {
    // ...
}
```

作用于整个类。

## synchronized的可重入性

synchronized是一个可重入锁，即一个线程可以在同步语句中调用同一个对象的其他同步方法。

线程如果已经持有一个对象的锁，当再次请求该对象锁时是不会被阻塞的，换言之，同一个线程可以多次请求同一个对象锁，即对象锁是可重入的。

对象锁关联了线程持有者id和计数器，当一个线程请求锁成功时，JVM会记录下持有锁的线程，并将计数器计为1。此时其他线程如果请求该对象锁则会进入等待，而持有该锁的线程再次请求该锁时，则可以拿到锁，并使计数器加1。当线程退出一个synchronized方法/块时，计数器会递减，当计数器为0时会释放该锁。

## JUC组件

JUC即java.util.concurrent包，提供了许多并发组件，而AQS（AbstractQueuedSynchronizer，抽象队列式同步器）被认为是JUC的核心，许多组件的实现都依赖于它。具体可以看看这部分[AQS笔记](/all/concurrency_04_内存模型?id=aqs（abstractqueuedsynchronizer）)。

### 可重入锁ReentrantLock

ReentrantLock是一个可重入的互斥锁，和synchronized相比：

1）synchronized是JVM层面实现的，ReentrantLock是JDK层面实现的。<br>
2）JDK1.6开始对synchronized进行了优化，因此两者性能大致相同。<br>
3）ReentrantLock是**等待可中断**的。当持有锁的线程长期不释放锁时，其他等待的线程可以放弃等待，改为处理其他事情。<br>
4）ReentrantLock**支持公平锁**。即多个线程在等待同一个锁时，必须按照申请锁的时间顺序来依次获得锁。synchronized的锁是非公平锁（随机获得锁的抢占机制），ReentrantLock默认也是使用非公平锁，但可以通过设置构造方法的参数来构造公平锁。<br>
5）ReentrantLock的**锁可以绑定多个条件**（ReentrantLock对象可以绑定多个Condition对象，通过多次调用`newCondition()`来绑定多个条件），而synchronized的锁对象只能借由wait和notify的通信机制来实现一个隐含的条件，每多一个条件就要多增加一个新的对象锁。<br>
6）ReentrantLock**必须手动释放锁**，否则会造成死锁。而synchronized无需手动释放锁，JVM会确保锁的释放。

因此，**除非需要使用到ReentrantLock的高级功能，否则优先使用synchronized**。

```java
public class LockExample {

    private Lock lock = new ReentrantLock();

    public void func() {
        // 在获取锁成功之前会一直阻塞
        lock.lock();
        try {
            for (int i = 0; i < 10; i++) {
                System.out.print(i + " ");
            }
        } finally {
			// 必须在finally块中手动释放锁，以防发生死锁
            lock.unlock();
        }
    }

	public static void main(String[] args) {
	    LockExample lockExample = new LockExample();
	    ExecutorService executorService = Executors.newCachedThreadPool();
	    executorService.execute(() -> lockExample.func());
	    executorService.execute(() -> lockExample.func());
	}
}
```

### 倒计时器CountDownLatch

通过计数器来用来控制线程间的通信，计数器归零时会唤醒等待中的线程。

内部维护了一个计数器，每次调用`countDown() `都会使计数器的值减1，减到0时会自动唤醒那些因为调用了`await()`而进入等待的线程。

```java
public class CountdownLatchExample {

    public static void main(String[] args) throws InterruptedException {
		// 倒计时器初始值为10
        final int totalThread = 10;
        CountDownLatch countDownLatch = new CountDownLatch(totalThread);
        ExecutorService executorService = Executors.newCachedThreadPool();
        for (int i = 0; i < totalThread; i++) {
            executorService.execute(() -> {
                System.out.print("run..");
				// 新线程使计数器减一
                countDownLatch.countDown();
            });
        }
		// 线程等待被唤醒
        countDownLatch.await();
        System.out.println("end");
        executorService.shutdown();
    }
}
```

CountDownLatch不能重复使用，调用`await()`的线程被阻塞，如果某个子线程被中断则其他等待的线程会永远都等待下去，永远达不到被唤醒的界限。

### 循环屏障CyclicBarrier

通过屏障点来控制线程间的通信，线程到达屏障点时将被阻塞，直到所有线程都到达同一个屏障点时，才能继续一起执行。

CyclicBarrier和CountDownLatch类似，内部维护了一个计数器，当线程调用`await()`后计数器值会减一，并进行等待。直到计数器归零，所有调用了`await()`进入等待的线程才能继续执行。由于其计数器可以通过`reset()`来重复使用，因此才名为循环屏障。

CyclicBarrier有两个构造方法，parties参数是计数器的初始值，barrierAction参数是所有线程都到达屏障点后会被执行一次的任务：

```java
public CyclicBarrier(int parties, Runnable barrierAction) {
    if (parties <= 0) throw new IllegalArgumentException();
    this.parties = parties;
    this.count = parties;
    this.barrierCommand = barrierAction;
}

public CyclicBarrier(int parties) {
    this(parties, null);
}
```

Demo如下：

```java
public class CyclicBarrierExample {

    static class MyRunnable implements Runnable {

        @Override
        public void run() {
            System.out.println();
            System.out.println("run barrierAction..");
        }

    }

    public static void main(final String[] args) {
        // 计数器初始值
        final int totalThread = 10;
        // 全部线程到达屏障点时自动执行一次的barrierAction
        final Runnable runnable = new MyRunnable();
        final CyclicBarrier cyclicBarrier = new CyclicBarrier(totalThread, runnable);
        final ExecutorService executorService = Executors.newCachedThreadPool();
        for (int i = 0; i < totalThread; i++) {
            executorService.execute(() -> {
                System.out.print("before..");
                try {
                    // 等待其他线程到达屏障点
                    cyclicBarrier.await();
                } catch (InterruptedException | BrokenBarrierException e) {
                    e.printStackTrace();
                }
                System.out.print("after..");
            });
        }
        executorService.shutdown();
    }
}
```

输出如下：

```java
before..before..before..before..before..before..before..before..before..before..
run barrierAction..
after..after..after..after..after..after..after..after..after..after..
```

### 信号量Semaphore

类似于操作系统的信号量，可以控制访问互斥资源的线程数量，常用于限流场景。

Semaphore底层维护了令牌数量，线程通过调用`acquire()`获取一个令牌，在成功获取之前会一直处于阻塞状态（除非被其他线程中断）。有个带令牌数量参数的重载方法`acquire(int permits)`，可以一次获取多个令牌，获取成功后可用的令牌数量也会减少指定的数量。

`tryAcquire()`同样用于获取一个令牌，但它不会阻塞，只会获取一次并返回获取结果是否成功；同样有个一次获取多个令牌的重载方法`tryAcquire(int permits)`，此外该方法还有其他带有超时等待参数的重载方法，允许在超时之前一直尝试获取令牌。

`release()`用于释放令牌。Semaphore默认是非公平锁，可以通过构造方法来改变。

```java
public class SemaphoreExample {

    public static void main(String[] args) {
        // Semaphore令牌数量
        final int clientCount = 3;
        final int totalRequestCount = 10;
        Semaphore semaphore = new Semaphore(clientCount);
        ExecutorService executorService = Executors.newCachedThreadPool();
        for (int i = 0; i < totalRequestCount; i++) {
            executorService.execute(()->{
                try {
                    // 获取令牌，在成功获取之前会一直阻塞
                    semaphore.acquire();
                    System.out.print(semaphore.availablePermits() + " ");
                } catch (InterruptedException e) {
                    e.printStackTrace();
                } finally {
                    // 释放令牌
                    semaphore.release();
                }
            });
        }
        executorService.shutdown();
    }
}
```

### 异步任务FutureTask

JDK1.5新增了具备返回值和可撤销的任务Callable，其返回值由FutureTask封装。FutureTask中获取执行结果的`get()`方法是一个阻塞的方法，直到任务执行完毕才能获取到结果。

```java
public class FutureTask<V> implements RunnableFuture<V>

public interface RunnableFuture<V> extends Runnable, Future<V>
```

FutureTask同样继承了Runnable接口，因此也可以像定义Runnable任务那样来使用，也可用于异步获取执行结果或取消执行任务的场景。

当一个计算任务需要执行很长时间，那么就可以用FutureTask来封装这个任务，等主线程完成自己的任务后再去获取结果。

```java
public static void main(String[] args) throws Exception {
    FutureTask<String> task = new FutureTask<>(() -> "Done.");
    new Thread(task).start();
    // 在成功获取结果之前会阻塞
    System.out.println(task.get());
}
```

### 阻塞队列接口BlockingQueue

BlockingQueue接口继承了Queue接口，主要提供了两个新的阻塞方法：put()和take()，这两个接口的方法可以看这个[queue笔记](/all/container_01_数据结构概览?id=queue)。

BlockingQueue接口主要有以下实现：

1）**FIFO队列**：LinkedBlockingQueue（线性）、ArrayBlockingQueue（数组，固定长度）<br>
2）**优先级队列** ：PriorityBlockingQueue

下面是用ArrayBlockingQueue实现的一个简单生产者消费者demo：

```java
public class ProducerConsumer {

    private static BlockingQueue<String> queue = new ArrayBlockingQueue<>(5);

    private static class Producer extends Thread {
        @Override
        public void run() {
            try {
                // 向阻塞队列中新增一个元素
                queue.put("product");
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.print("produce..");
        }
    }

    private static class Consumer extends Thread {

        @Override
        public void run() {
            try {
                // 从阻塞队列中取出一个元素
                String product = queue.take();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.print("consume..");
        }
    }
    
    public static void main(String[] args) {
        // 新增2个生产者
        for (int i = 0; i < 2; i++) {
            Producer producer = new Producer();
            producer.start();
        }
        // 新增5个消费者
        for (int i = 0; i < 5; i++) {
            Consumer consumer = new Consumer();
            consumer.start();
        }
        // 新增3个生产者
        for (int i = 0; i < 3; i++) {
            Producer producer = new Producer();
            producer.start();
        }
    }

}
```

### 并行执行框架ForkJoin

JDK1.7引入了并行执行框架ForkJoin和ForkJoinPool，和MapReduce原理类似，都是把大的计算任务拆分成多个小任务并行计算，最终汇总每个小任务结果后得到大任务结果。

ForkJoin使用ForkJoinPool来启动，它是一个特殊的线程池，线程数量取决于逻辑处理器数量。

ForkJoinPool实现了**工作窃取算法**来提高CPU的利用率。每个线程都维护了一个双端队列，用来存储需要执行的任务。工作窃取算法允许空闲的线程从其它线程的双端队列中窃取一个任务来执行。窃取的任务必须是最晚的任务，避免和队列所属线程发生竞争，但是如果队列中只有一个任务时还是会发生竞争。

JDK1.8新增的并行流parallelStream底层就使用了ForkJoin。关于parallelStream和ForkJoin的其他细节可以看[这部分笔记](/all/basic_11_Java8新特性?id=parallelstream%e5%92%8cforkjoin)。

## 参考链接

* [Java 并发](http://www.cyc2018.xyz/Java/Java%20%E5%B9%B6%E5%8F%91.html)
* [Java多线程：synchronized的可重入性](https://www.cnblogs.com/cielosun/p/6684775.html)