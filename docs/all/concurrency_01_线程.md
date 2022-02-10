<!--
date: 2021-06-29T22:46:12+08:00
lastmod: 2022-02-10T22:46:12+08:00
-->
## 线程与进程

1）进程，每个进程都有独立的代码和数据空间，切换进程开销大<br>
2）线程，同一类的线程共享代码和数据空间，单个线程有独立的运行权和程序计数器(`program counter`)，切换线程开销小<br>
3）进程和线程都是描述CPU工作的时间段，线程是粒度更细小的时间段。

## java的线程

jvm启动时会创建一个`main`线程来执行`public static void main(String[] args)`方法里面的代码

java的线程机制是抢占式的，这表示调度机制会周期性中断线程，将上下文切换到另一个线程，从而为每个线程都提供处理器时间片来驱动

## this逃逸

在构造器返回之前，即在一个对象还没初始化之前就返回当前对象的this引用，即为this逃逸。

在单线程情况下，this逃逸不会产生问题。但若在多线程场景下，其他线程可以通过this逃逸来提前获取到还未完全初始化的对象，此时可能造成意想不到的问题。比如其他线程访问到尚未初始化的成员变量，可能读取到错误的值，或者是产生NullPointerException等。

可能发生this逃逸的情况：

1）构造器内部对成员变量进行赋值，并显式抛出this引用，此时可能发生重排序；<br>
2）构造器内部创建了匿名内部类对象并抛出其引用，因为内部类天然持有着外部类对象的引用，此时相当于隐式地抛出了this引用。

* [this引用逃逸问题](https://blog.csdn.net/qq_33804730/article/details/79451670)

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

### 使用Executor

前三种方式都是手动一个个创建线程来执行定义的任务，Executor则是JDK自身提供的线程池框架，通过该框架来使用多线程以达到复用线程的目的。

* [线程池](/all/concurrency_02_线程池)

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
3）Callable会抛出异常，而Runnable只能在内部处理异常，不能继续往外抛。

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

### 休眠sleep()

静态方法`Thread.sleep(long millis)`会令当前正在执行的线程休眠指定的毫秒时间。

在线程休眠过程中，可能会有其他线程尝试中断当前线程，此时sleep()会抛出InterruptedException。在通过Thread或Runnable来使用线程时，由于异常无法跨线程传播回main线程中，所以需要在本地进行异常处理，包括其他线程抛出的异常。如果通过Callable来使用线程则没有这个问题。

```java
// Thread or Runnable
public void run() {
    try {
        Thread.sleep(3000);
    } catch (InterruptedException e) {
        e.printStackTrace();
    }
}

// Callable
public String call() throws InterruptedException {
    Thread.sleep(3000);
    return "callable";
}
```

### 优先级

线程优先级用于表示其重要性，线程调度器倾向于让优先级高的线程先执行。可以通过`getPriority()`和`setPriority(int newPriority)`来操作优先级，Java提供了10个优先级（从最低到最高优先级为1到10，5是默认优先级）。

由于不同操作系统对于线程的优先级级别数量和策略不同，因此只建议使用1,5和10这三个优先级（MIN_PRIORITY、NORM_PRIORITY和MAX_PRIORITY），在开发中应该尽量避免依赖优先级来控制线程的执行顺序。

### 让步yield()

静态方法`Thread.yield()`会建议线程调度器：当前正在执行的线程的工作已经告一段落，可以让出CPU给其他具有相同优先级的线程使用。但这仅仅只是一个建议，无法保证一定会被线程调度器采纳，因此开发中应该避免依赖该方法。

### 守护线程Daemon

守护线程，也叫后台线程，是程序运行时在后台提供服务的线程，不属于程序中不可或缺的部分。

1）当所有非守护线程结束时，程序也会结束，并杀死所有守护线程，因此不要把必须执行的任务放到后台线程中。<br>
2）main()属于非守护线程。<br>
3）在线程启动之前可以通过`setDaemon()`将一个线程设置为守护线程，如果线程已经启动再设置则会抛出异常。<br>
4）守护线程创建的线程会自动设置为守护线程，原理是初始化线程时会获取当前线程的daemon来设置自身的daemon。（daemon是Thread中的一个私有boolean变量，默认是false）

```java
// 使用守护线程的例子
public class DaemonThreadStudy {
    private static class DaemonThreadFactory implements ThreadFactory {
        @Override
        public Thread newThread(Runnable r) {
            Thread thread = new Thread(r);
            thread.setDaemon(true);
            return thread;
        }
    }

    private static class DaemonFromFactory implements Runnable {
        @Override
        public void run() {
            try {
                while (true) {
                    Thread.sleep(100);
                    System.out.println(Thread.currentThread() + " " + this);
                }
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }

    public static void main(String[] args) throws InterruptedException {
        Executor executor = Executors.newCachedThreadPool(new DaemonThreadFactory());
        for (int i = 0; i < 10; i++) {
            executor.execute(new DaemonFromFactory());
        }
        System.out.println("All daemons started");
        Thread.sleep(100);
    }
}
```

## 中断

一个线程执行完毕后会自动结束，如果在执行过程中发生异常也会提前结束。

### interrupt()

调用一个线程的`interrupt()`方法，会为该线程设置一个中断标志，至于该线程是否提前结束取决于具体的业务代码。但是如果该线程处于阻塞、限期等待或无限期等待状态，则会抛出InterruptedException，从而提前结束该线程，不再继续执行后续的语句。

需要注意的是，`interrupt()`方法无法中断处于I/O阻塞和synchronized锁阻塞的线程。（InterruptibleChannel和java.nio.channels.Selector除外，这两个会提前结束线程，详见jdk中interrupt()的注释。）

### isInterrupted()

用于判断线程是否处于中断状态，该方法实际是调用另一个同名带参数的本地方法：

```java
public boolean isInterrupted() {
    return isInterrupted(false);
}

// ClearInterrupted参数表明是否清除线程的中断标志
private native boolean isInterrupted(boolean ClearInterrupted);
```

### interrupted()

这是个静态方法，同样用于判断当前线程是否处于中断状态，但区别是该方法在调用后会清除当前线程的中断标志。换言之，对于一个处于中断状态的线程，在第二次调用该方法时会得到不同的结果。这也是该方法名字取过去式的原因，其源码如下：

```java
public static boolean interrupted() {
    return currentThread().isInterrupted(true);
}
```

### 样例代码

```java
// Demo#1
public class InterruptDemo implements Runnable {

    @Override
    public void run() {
        while (true) {
            System.out.println("Running.");
        }
    }

    public static void main(final String[] args) {
        final Thread thread = new Thread(new InterruptDemo());
        thread.start();
        // 仅仅设置了中断标志，线程依然会无限循环下去
        thread.interrupt();
    }
}

// Demo#2
public class InterruptDemo implements Runnable {

    @Override
    public void run() {
        while (true) {
            System.out.println("Running.");
            // 接收到中断标志，主动return，打破无限循环
            if (Thread.currentThread().isInterrupted()) {
                return;
            }
        }
    }

    public static void main(final String[] args) {
        final Thread thread = new Thread(new InterruptDemo());
        thread.start();
        thread.interrupt();
    }
}

// Demo#3
public class InterruptDemo implements Runnable {

    @Override
    public void run() {
        try {
            Thread.sleep(1000);
            // 由于线程处于休眠状态，此时调用其interrupt()导致抛出异常，try代码块中后续的语句无法继续执行
            System.out.println("Running.");
        } catch (final InterruptedException e) {
            e.printStackTrace();
        }
    }

    public static void main(final String[] args) {
        final Thread thread = new Thread(new InterruptDemo());
        thread.start();
        thread.interrupt();
    }
}
```

### Executor的中断操作

调用Executor的`shutdown()`方法会等待线程都执行完毕之后再关闭，但是如果调用的是`shutdownNow()`方法，则相当于调用每个线程的`interrupt()`方法。

```java
public static void main(String[] args) {
    ExecutorService executorService = Executors.newCachedThreadPool();
    executorService.execute(() -> {
        try {
            Thread.sleep(2000);
            System.out.println("Thread run");
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    });
    // 立刻中断所有线程
    executorService.shutdownNow();
    System.out.println("Main run");
}
```

如果只想中断Executor中的一个线程，可以通过使用`submit()`方法来提交一个线程，它会返回一个Future<?>对象，通过调用该对象的`cancel(true)`方法就可以中断线程。

```java
Future<?> future = executorService.submit(() -> {
    // ..
});
future.cancel(true);
```

## 互斥同步



## 线程间的协作

当多个线程在一起工作时，某些工作的部分存在先后顺序，则需要对线程进行协调。

### join()

一个线程如果需要等待另一个线程先执行完成，则可以调用另一个线程的`join()`方法。此时当前线程会被挂起，直到另一个线程结束。join()会抛出InterruptedException，因此同样可以用interrupt()来中断它。

join()方法有一个带时间参数的重载方法，当挂起指定时间后，不管目标线程是否结束都会返回执行原本挂起的线程。

### wait() notify() notifyAll()



## 参考链接

* [Java 并发](http://www.cyc2018.xyz/Java/Java%20%E5%B9%B6%E5%8F%91.html)
* [Java多线程开发（一）| 基本的线程机制](https://www.jianshu.com/p/d094f6adb78e)
* [Java中interrupt的使用](https://www.cnblogs.com/jenkov/p/juc_interrupt.html)