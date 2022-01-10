<!--
date: 2022-01-06T22:34:12+08:00
lastmod: 2022-01-07T22:34:12+08:00
-->
## 外观模式（Facade Pattern）

外观模式也叫门面模式，提供一个外观类来帮助客户端调用系统的接口，对客户端屏蔽了系统的复杂性。在Java中外观模式应用很多，最典型的就是MVC设计，这里的Controller实际上就是一个外观类。

平时经常使用的日志功能同样使用到了外观模式：slf4j接口 + 某种具体的日志实现，具体的日志门面可以看这篇文章：[日志框架与门面模式](https://lewky.cn/posts/log-framework/)

## 样例代码

定义形状接口和子类：

```java
public interface Shape {
   void draw();
}

public class Rectangle implements Shape {
 
   @Override
   public void draw() {
      System.out.println("Rectangle::draw()");
   }
}

public class Circle implements Shape {
 
   @Override
   public void draw() {
      System.out.println("Circle::draw()");
   }
}
```

定义一个外观类：

```java
public class ShapeMaker {
   private Shape circle;
   private Shape rectangle;
   private Shape square;
 
   public ShapeMaker() {
      circle = new Circle();
      rectangle = new Rectangle();
      square = new Square();
   }
 
   public void drawCircle(){
      circle.draw();
   }
   public void drawRectangle(){
      rectangle.draw();
   }
   public void drawSquare(){
      square.draw();
   }
}
```

定义测试类：

```java
public class Test {
   public static void main(String[] args) {
      ShapeMaker shapeMaker = new ShapeMaker();
 
      shapeMaker.drawCircle();
      shapeMaker.drawRectangle();
      shapeMaker.drawSquare();      
   }
}
```

## 参考链接

* [外观模式](https://www.runoob.com/design-pattern/facade-pattern.html)