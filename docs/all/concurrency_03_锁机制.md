<!--
date: 2022-02-12T22:46:12+08:00
lastmod: 2022-02-13T22:46:12+08:00
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



## JUC组件

JUC即java.util.concurrent包，提供了许多并发组件，而AQS（AbstractQueuedSynchronizer，抽象队列式同步器）被认为是JUC的核心，许多组件的实现都依赖于它。

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

    public static void main(String[] args) {
        final int totalThread = 10;
        CyclicBarrier cyclicBarrier = new CyclicBarrier(totalThread);
        ExecutorService executorService = Executors.newCachedThreadPool();
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

### 信号标Semaphore



### AQS（AbstractQueuedSynchronizer）



## 参考链接

* [Java 并发](http://www.cyc2018.xyz/Java/Java%20%E5%B9%B6%E5%8F%91.html)