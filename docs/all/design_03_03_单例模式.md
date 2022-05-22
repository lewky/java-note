<!--
date: 2021-12-08T22:34:12+08:00
lastmod: 2022-05-15T22:34:12+08:00
-->
## 单例模式（Singleton Pattern）

单例模式就是确保一个类只有一个实例对象，这个单例类必须自己创建这个唯一的实例对象，并对外提供访问该单例对象的方式。单例类的构造方法是私有的，因此不能被继承。

单例模式可以减少内存的开销，开发中工具类通常都是单例类，但要注意线程安全问题。单例模式有多种实现方式。

## 饿汉式

```java
public class EagerSingleton {

    private static final EagerSingleton instance = new EagerSingleton();

    // 构造方法私有化，防止被外部类直接实例化
    private EagerSingleton() {}

    public static EagerSingleton getInstance() {
        return instance;
    }

}
```

饿汉式正如其名，这种方式每次在类加载时就创建单例对象，线程安全且性能高，属于用空间换时间的做法。所谓饿汉式，就是不使用懒加载的方式去实现单例模式，类似于Hibernate是否使用懒加载去查询db。

## 懒汉式

```java
public class LazySingleton {

    private static LazySingleton instance = null;

    private LazySingleton() {}

    // 加锁解决线程安全问题，但是降低了性能
    public static synchronized LazySingleton getInstance() {
        if (instance == null) {
            instance = new LazySingleton();
        }
        return instance;
    }

}
```

懒汉式每次在获取单例前需要先判断是否为创建了实例，如果还未创建单例对象再创建出来，也就是所谓的懒加载。为了解决线程安全问题，需要对方法进行同步加锁。

由于每次都需要进行一次判断，且使用了同步方法，性能较低，但只有在需要使用时才会去创建对应的单例对象，属于时间换空间的做法。

## 双重检查加锁

```java
public class LazySingleton2 {

    // volatile保证内存一致性
    private volatile static LazySingleton2 instance = null;

    private LazySingleton2() {}

    public static LazySingleton2 getInstance() {
        if (instance == null) {
            // 使用同步块进行第二次检测是否为null，解决线程安全问题
            synchronized (LazySingleton2.class) {
                if (instance == null) {
                    instance = new LazySingleton2();
                }
            }
        }
        return instance;
    }

}
```

双重检查加锁是对懒汉式做法的优化，不直接对整个方法进行同步，而是将同步范围缩小到方法内部。先第一次判断单例对象是否为null，不为null则直接返回该单例对象。

若为null，则进入同步块，再判断一次是否为null，若为null则创建单例对象。这里的同步块是为了解决线程安全问题，只有在第一次创建单例对象时才会进入到同步块。

同步块内再判断一次，是因为假若有多个线程在竞争锁，此时如果第一个获得锁的线程已经创建了单例对象，其他排队的线程则无需再次创建对象，因此需要在同步块内再判断一次。此外，为了保证其他线程能立刻获取instance变量最新的值，这里使用了volatile关键词。

被volatile修饰的变量的值，将不会被本地线程缓存，所有对该变量的读写都是直接操作共享内存。**注意：在java1.4及以前版本中，很多JVM对于volatile关键字的实现的问题，会导致“双重检查加锁”的失败，因此“双重检查加锁”机制只能用在java5及以上的版本。**

由于volatile关键字可能会屏蔽掉虚拟机中一些必要的代码优化，所以运行效率并不是很高。因此一般建议，没有特别的需要，不要使用。也就是说，虽然可以使用“双重检查加锁”机制来实现线程安全的单例，但并不建议大量采用，可以根据情况来选用。

## 静态内部类

利用类加载器机制和静态内部类，可以实现线程安全和延迟加载，且实现十分简单。

```java
public class LazySingleton3 {

    private LazySingleton3() {}

    // 静态内部类
    private static class SingletonHolder {
        private static final LazySingleton3 instance = new LazySingleton3();
    }

    // 在调用该方法之前，静态内部类不会被加载，因此也不会创建单例对象
    public static LazySingleton3 getInstance() {
        return SingletonHolder.instance;
    }

}
```

这种实现方式，无需使用同步，仅依赖于类加载器机制来保证线程安全，同时又实现了懒加载，是开发中很常用的单例模式实现方式。

## 枚举类

枚举也可以用来实现单例模式，且更简洁，能避免多线程同步问题，支持序列化机制，防止反序列化重新创建新的对象。

```java
public enum Singletion {
    // 只定义一个枚举的元素
    instance;
    
    // 枚举类可以进行某种操作
    public void doSomething() {
        
    }
}
```

这种方式不属于懒加载，如果需要避免反序列化来创建新对象可以用枚举类来实现单例。

## 参考链接

* [单例模式](https://www.runoob.com/design-pattern/singleton-pattern.html)
* [《JAVA与模式》之单例模式](https://www.cnblogs.com/java-my-life/archive/2012/03/31/2425631.html)