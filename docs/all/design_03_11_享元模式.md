<!--
date: 2022-01-06T22:34:12+08:00
lastmod: 2022-01-10T22:34:12+08:00
-->
## 享元模式（Flyweight Pattern）

享元模式目的在于尽量重用已有的对象，减少对象的创建，以此减少内存的开销。先从缓存池中寻找匹配的对象，找不到再创建新的。

最典型的例子就是各种池化技术，如线程池、数据库连接池，还有各大包装类内置的[缓存池](https://javanote.doc.lewky.cn/#/all/basic_01_数据类型?id=%e7%bc%93%e5%ad%98%e6%b1%a0)

## 样例代码

定义一个形状接口Shape和圆形类Circle：

```java
public interface Shape {

    void draw();
}

public class Circle implements Shape {

    private final String color;

    public Circle(final String color) {
        this.color = color;
    }

    @Override
    public void draw() {
        System.out.println("Shape: Circle, Color: " + color);
    }

}
```

定义一个圆形工厂CircleFactory：

```java
public class CircleFactory {

    // 缓存池，使用color作为唯一标志
    private static final HashMap<String, Circle> map = new HashMap<>();

    public static Circle getCircle(final String color) {
        Circle circle = map.get(color);
        if (circle == null) {
            circle = new Circle(color);
            // 新创建的对象要放到缓存池中
            map.put(color, circle);
        }

        return circle;
    }

}
```

## 参考链接

* [享元模式](https://www.runoob.com/design-pattern/flyweight-pattern.html)