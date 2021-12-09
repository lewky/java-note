<!--
date: 2021-12-08T22:34:12+08:00
lastmod: 2021-12-08T22:34:12+08:00
-->
## 工厂方法模式（Factory Pattern）

对于实现同一接口的对象，不直接用new来创建，而是引入一个工厂类（或者是实现同一工厂接口的若干个工厂类）来创建对应的对象。为了便于扩展，工厂类中创建对象的方法返回的是一个抽象类或者接口。

这种模式应用了多态，将对象的创建延迟到子类进行，创建的实例类型由子类自己决定。客户端只需要调用工厂接口的方法来创建指定类型的实例，不需要关注创建对象的具体过程。

工厂方法模式易于扩展，每多一种产品，都可以通过引入一个新的子类对象和对应的对象创建工厂。但缺点也显而易见，引入的具体类和工厂类会在一定程度上增加系统复杂度。

## 使用场景

常见的，比如数据库连接jdbc，有多种不同的数据库驱动，如MySQL、PostgreSQL等，在使用jdbc连接数据库时，通过上层的jdbc接口来创建对应的数据库连接对象。

除了数据库连接，还有日志的记录：比如是记录到磁盘、数据库、系统事件或者远程服务器等。在各种框架中，工厂方法模式通常与SPI机制结合在一起使用。

## 样例代码

日志记录器：记录到数据库或者磁盘。

```java
public interface MessageLogger {

    public void log();
}

public class DbMessageLogger implements MessageLogger {

    @Override
    public void log() {
        System.out.println("Log message to DB.");
    }

}

public class FileMessageLogger implements MessageLogger {

    @Override
    public void log() {
        System.out.println("Log message to File.");
    }

}
```

工厂类：用以创建指定的日志记录器。

```java
public class MessageLoggerFactory {

    public MessageLogger createLogger(final String type) {
        MessageLogger logger = null;
        if ("DB".equalsIgnoreCase(type)) {
            logger = new DbMessageLogger();
        } else if ("File".equalsIgnoreCase(type)) {
            logger = new FileMessageLogger();
        }

        return logger;
    }

}
```

## 参考链接

* [工厂模式](https://www.runoob.com/design-pattern/factory-pattern.html)