<!--
date: 2022-01-10T22:34:12+08:00
lastmod: 2022-01-11T22:34:12+08:00
-->
## 责任链模式（Chain of Responsibility Pattern）

责任链模式目的在于将请求对象和处理对象解耦，双方间只存在最简单的依赖关系。同时将若干个处理对象组成链表结构，每个处理对象都持有下一个处理对象，当一个处理对象无法处理该请求时就将请求传递给下一个处理对象。

处理对象之间的顺序可以自由组合，客户端不需要知道具体有哪些处理对象，只需要将请求对象传递给第一个处理对象即可。

实际的应用实例有：Java Servlet中的过滤器Filter，日志框架的记录器Logger等。

## 样例代码

定义一个圆形类Circle：

```java
public class Circle {

    String color;
    String radius;
    String border;

    public Circle(final String color, final String radius, final String border) {
        super();
        this.color = color;
        this.radius = radius;
        this.border = border;
    }

}
```

定义一系列处理器类：

```java
public abstract class AbstractHandler {

    AbstractHandler nextHandler;

    public AbstractHandler getNextHandler() {
        return nextHandler;
    }

    public void setNextHandler(final AbstractHandler nextHandler) {
        this.nextHandler = nextHandler;
    }

    abstract void handle(Circle circle);

}

public class ColorHandler extends AbstractHandler {

    @Override
    void handle(final Circle circle) {
        if (circle.color == null && nextHandler != null) {
            nextHandler.handle(circle);
        } else {
            System.out.println("Circle color: " + circle.color);
        }
    }

}

public class RadiusHandler extends AbstractHandler {

    @Override
    void handle(final Circle circle) {
        if (circle.radius == null && nextHandler != null) {
            nextHandler.handle(circle);
        } else {
            System.out.println("Circle radius: " + circle.radius);
        }
    }

}

public class BorderHandler extends AbstractHandler {

    @Override
    void handle(final Circle circle) {
        if (circle.border == null && nextHandler != null) {
            nextHandler.handle(circle);
        } else {
            System.out.println("Circle border: " + circle.border);
        }
    }

}
```

定义一个测试类：

```java
public class Test {

    public static void main(final String[] args) {
        // 构建责任链，按顺序处理color, radius, border
        final ColorHandler handlerChain = new ColorHandler();
        final RadiusHandler radiusHandler = new RadiusHandler();
        final BorderHandler borderHandler = new BorderHandler();
        handlerChain.setNextHandler(radiusHandler);
        radiusHandler.setNextHandler(borderHandler);

        final Circle circle1 = new Circle("red", "30px", "1px");
        handlerChain.handle(circle1);
        final Circle circle2 = new Circle(null, "30px", "1px");
        handlerChain.handle(circle2);
        final Circle circle3 = new Circle(null, null, "1px");
        handlerChain.handle(circle3);
    }
}
```

## 参考链接

* [责任链模式](https://www.runoob.com/design-pattern/chain-of-responsibility-pattern.html)