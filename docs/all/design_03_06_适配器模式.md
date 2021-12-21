<!--
date: 2021-12-21T22:34:12+08:00
lastmod: 2021-12-21T22:34:12+08:00
-->
## 适配器模式（Adapter Pattern）

适配器模式的目的是解决不同接口间的兼容问题，为目标类提供一个适配器类，将一个接口适配成期望的另一个接口。

适配器不是在详细设计时添加的，而是为了解决项目中存在的接口不兼容问题，换言之，项目中不应该存在太多适配器类，如果有可能，应该在最开始时就避免这种情况，或者对系统进行重构。

## 样例代码

Duck是客户端期望的接口，MallardDuck实现了Duck接口。

```java
public interface Duck {
    void quack();

    void fly();
}

// 绿头鸭
public class MallardDuck implements Duck {

    @Override
    public void quack() {
        System.out.println("MallardDuck quack()");
    }

    @Override
    public void fly() {
        System.out.println("MallardDuck fly()");
    }
}
```

WildTurkey实现了Turkey接口，客户端希望将Turkey接口的实现类适配成Duck接口。

```java
public interface Turkey {
    void gobble();

    void fly();
}

// 野生火鸡
public class WildTurkey implements Turkey {

    @Override
    public void gobble() {
        System.out.println("WildTurkey gobble()");
    }

    @Override
    public void fly() {
        System.out.println("WildTurkey fly()");
    }
}
```

TurkeyAdapter是适配器类Adapter，将Turkey接口的实现类适配成Duck接口，Turkey接口的实现类是被适配器类Adaptee。

```java
public class TurkeyAdapter implements Duck {

    // 被适配器Adaptee
    Turkey turkey;

    public TurkeyAdapter(final Turkey turkey) {
        this.turkey = turkey;
    }

    // Turkey接口的gobble()方法被适配成了Duck接口的quack()
    @Override
    public void quack() {
        turkey.gobble();
    }

    @Override
    public void fly() {
        turkey.fly();
    }
}
```

可以看到，原本的WildTurkey实现的是Turkey接口，无法转型为Duck接口。现在将WildTurkey包装为TurkeyAdapter后，就可以和MallardDuck一样都向上转型成Duck接口。

因此适配器模式是在不修改原本接口或者接口实现类的情况下，通过引入一个适配器类来解决接口不兼容的问题。

## 参考链接

* [适配器模式](https://www.runoob.com/design-pattern/adapter-pattern.html)
* [《JAVA与模式》之适配器模式](https://www.cnblogs.com/java-my-life/archive/2012/04/13/2442795.html)