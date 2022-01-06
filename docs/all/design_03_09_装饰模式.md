<!--
date: 2021-12-29T22:34:12+08:00
lastmod: 2022-01-06T22:34:12+08:00
-->
## 装饰模式（Decorator Pattern）

在不改变原对象结构的情况下，对其进行功能上的扩展。实际上是用聚合来取代继承，装饰模式是在不想使用继承来扩展功能时的一种替代方案。（继承是一种强耦合关系）

在Java中，一个典型的装饰模式的应用就是IO流，装饰模式可以非常灵活地一层一层装饰原本对象的功能。

装饰模式和桥接模式有些类似，都是用聚合取代继承，且都打破了冗长的继承链（不存在多层继承）。但区别在于装饰模式是装饰器和原对象都实现了同一个接口，因此装饰器持有着该接口就可以随意进行一层层装饰。

而桥接模式中，是将多种不同维度的变化分离开来，各自抽象，并将这些抽象类用聚合关系关联起来，因此桥接模式无法像装饰模式那样灵活的一层层装饰。

## 样例代码

定义一个形状接口Shape和一个圆形类Circle。

```java
public interface Shape {

    void draw();
}

public class Circle implements Shape {

    @Override
    public void draw() {
        System.out.println("Shape: Circle");
    }

}
```

然后定义一个装饰器抽象类ColorShapeDecorator，以及装饰器类RedColorShapeDecorator。该装饰器可以用来给原本的形状接口装饰上额外的颜色功能。

```java
public abstract class ColorShapeDecorator implements Shape {

    private final Shape decoratedShape;

    public ColorShapeDecorator(final Shape decoratedShape) {
        this.decoratedShape = decoratedShape;
    }

    @Override
    public void draw() {
        decoratedShape.draw();
        // 装饰上新的功能
        expandColor();
    }

    abstract void expandColor();
}

public class RedColorShapeDecorator extends ColorShapeDecorator {

    public RedColorShapeDecorator(final Shape decoratedShape) {
        super(decoratedShape);
    }

    @Override
    void expandColor() {
        System.out.println("Color: Red");
    }

}
```

再定义一个类似的装饰器：SizeShapeDecorator和BigSizeShapeDecorator，该装饰器可以用来给原本的形状接口装饰上额外的尺寸功能。

```java
public abstract class SizeShapeDecorator implements Shape {

    private final Shape decoratedShape;

    public SizeShapeDecorator(final Shape decoratedShape) {
        this.decoratedShape = decoratedShape;
    }

    @Override
    public void draw() {
        decoratedShape.draw();
        // 装饰上新的功能
        expandSize();
    }

    abstract void expandSize();
}

public class BigSizeShapeDecorator extends SizeShapeDecorator {

    public BigSizeShapeDecorator(final Shape decoratedShape) {
        super(decoratedShape);
    }

    @Override
    void expandSize() {
        System.out.println("Size: Big");
    }

}
```

最后定义一个测试类：

```java
public class Test {

    public static void main(final String[] args) {
        final Shape circle = new Circle();
        circle.draw();
        System.out.println("=========");

        final Shape redCircle = new RedColorShapeDecorator(circle);
        redCircle.draw();
        System.out.println("=========");

        final Shape bigCircle = new BigSizeShapeDecorator(circle);
        bigCircle.draw();
        System.out.println("=========");

        final Shape bigRedCircle = new BigSizeShapeDecorator(redCircle);
        bigRedCircle.draw();
    }
}
```

测试结果如下：

```java
Shape: Circle
=========
Shape: Circle
Color: Red
=========
Shape: Circle
Size: Big
=========
Shape: Circle
Color: Red
Size: Big
```

## 参考链接

* [装饰器模式](https://www.runoob.com/design-pattern/decorator-pattern.html)