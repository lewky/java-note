<!--
date: 2021-12-21T22:34:12+08:00
lastmod: 2021-12-23T22:34:12+08:00
-->
## 桥接模式（Bridge Pattern）

桥接模式的目的是解决继承滥用的问题，旨在用聚合来取代继承。

因为继承是一种强耦合关系，当存在多层次的继承时，如果改变父类，会导致需要修改所有的子类；如果将抽象类和实现类分离，使用聚合关系来将二者关联起来，使二者可以独立的变化，就可以避免这种问题。这也符合组合/聚合复用原则。

## 非桥接模式的样例代码

举个例子，现在有个图形，分为形状和颜色两个维度的属性。形状可以有圆形，颜色可以有红色。现在需要一个红色圆形类。

如果不用桥接模式来做，代码如下：

```java
public abstract class Shape {

    abstract void showShape();
}

public class Circle extends Shape {

    @Override
    public void showShape() {
        System.out.println("This is a circle.");
    }

}

public abstract class Color {

    abstract String showColor();
}

public class Red extends Color {

    @Override
    public String showColor() {
        return "red";
    }

}
```

现在形状和颜色对应两个不同抽象类，客户端需要一个红色圆形类，可以新增一个RedCircle类，该类继承Circle：

```java
public class RedCircle extends Circle {

    private final Color red = new Red();

    @Override
    public void showShape() {
        System.out.println("This is a " + red.showColor()  + " circle.");
    }

}
```

可以看到，这种实现有两层继承：RedCircle继承了Circle，Circle继承了Shape，RedCircle里拥有Red对象。

在这种多层继承实现中，耦合太强。如果客户端现在想要新增一种新的颜色：绿色，此时就需要新增Green类和GreenCircle类。

如果客户端还想要新增一个矩形类，那么除了新增Retangle类外，还需要新增RedRetangle类和GreenRetangle类来支持有颜色的矩形。可以看到，随着功能的扩展，会大量增加类的数量。

在这种情况下，可以通过桥接模式来对颜色和形状进行解耦，使用聚合来替代继承的实现。

## 桥接模式的样例代码

修改Shape抽象类，令其持有Color抽象类对象，将这两个维度的变化用聚合关系关联起来。也可以反过来，令Color抽象类持有Shape抽象类对象。

```java
public abstract class Shape {

    private Color color;

    public Color getColor() {
        return color;
    }

    public void setColor(final Color color) {
        this.color = color;
    }

    abstract void showShape();
}
```

修改Circle类：

```java
public class Circle extends Shape {

    @Override
    public void showShape() {
        if (getColor() != null) {
            System.out.println("This is a " + getColor().showColor()  + " circle.");
        } else {
            System.out.println("This is a circle.");
        }
    }

}
```

这样一来，就实现了两个抽象类之间的解耦。如果客户端想新增一个绿色矩形，只需要新增Retangle类和Green类即可。

## 参考链接

* [桥接模式](https://www.runoob.com/design-pattern/bridge-pattern.html)
* [简说设计模式——桥接模式](https://www.cnblogs.com/adamjwh/p/9033548.html)