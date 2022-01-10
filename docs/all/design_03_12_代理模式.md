<!--
date: 2022-01-10T22:34:12+08:00
lastmod: 2022-01-10T22:34:12+08:00
-->
## 代理模式（Proxy Pattern）

代理模式的目的是控制被代理对象的访问，比如说，客户端访问某个对象时可能存在一定的困难，如需要权限验证等操作，此时可以提供一个代理对象来完成对该对象的访问，在访问对象的前后增加一些其他的操作，比如Spring中的AOP就是很常见的代理模式。

代理模式和适配器模式、装饰模式的区别：

● 适配器模式提供的适配器类实现的是与被适配器类不同的另一个接口，目的是令被适配器类能够调用原本不兼容的其他接口。<br>
● 代理模式提供的代理类与被代理对象实现的是同一个接口，这样才能保证代理类和被代理类具备一样的访问接口，此外代理类对象持有着被代理类对象。<br>
● 装饰模式提供的装饰类与被装饰类同样实现的同一个接口，但是装饰类中持有的对象类型不是具体的被装饰类类型，而是二者共有的接口或者抽象类类型，因此装饰模式可以层层装饰，一直添加新的功能。

这三种模式都是用聚合关系来取代了继承，避免了对象之间的强耦合关系。

## 样例代码

定义一个形状接口Shape和圆形类Circle：

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

定义一个代理类CircleProxy：

```java
public class CircleProxy implements Shape {

    private final Circle circle;

    public CircleProxy(final Circle circle) {
        this.circle = circle;
    }

    @Override
    public void draw() {
        // 可以在调用draw()之前进行额外处理
        System.out.println("Before..");

        // 调用被代理对象的同名方法
        circle.draw();

        // 可以在调用draw()之后进行额外处理
        System.out.println("After..");
    }

}
```

定义一个测试类：

```java
public class Test {

    public static void main(final String[] args) {
        final Circle circle = new Circle();
        circle.draw();
        final CircleProxy proxy = new CircleProxy(circle);
        proxy.draw();
    }
}
```

代理模式的实现分为静态代理和动态代理，上面的写法就属于静态代理。

静态代理的代理类在程序运行之前就已经存在了，而动态代理的代理类是在程序运行时借助反射机制来动态生成的，动态代理又分为基于接口的JDK实现和基于继承的cglib实现。

## 参考链接

* [代理模式](https://www.runoob.com/design-pattern/proxy-pattern.html)