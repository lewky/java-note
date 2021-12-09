<!--
date: 2021-12-08T22:34:12+08:00
lastmod: 2021-12-09T22:34:12+08:00
-->
## 抽象工厂模式（Abstract Factory Pattern）

抽象工厂模式是在工厂方法模式的基础上增加了一层抽象，为所有的工厂类增加了一个抽象工厂，这个抽象工厂的目的是创建不同的工厂实例。换言之，工厂方法模式是通过工厂来创建实例，抽象工厂模式是先通过工厂的工厂来创建工厂，再用工厂来创建实例。

抽象工厂模式的目的是解决产品之间的依赖问题，这里涉及到两个概念：产品族和产品等级结构。

产品族由一系列不同类型的产品构成，相同的产品类型则构成一个产品等级结构。换言之，相当于存在两个不同的维度。不同的产品族之间的产品互不依赖，同一产品族中的产品互相依赖。

举个例子，电子设备工厂可以创建CPU和主板两种产品，而产品又分为两个种类：AMD和Intel。即现在有两个产品族：AMD和Intel，有两个产品等级结构：CPU和主板。AMD的CPU不能安装在Intel的主板上，这就是不同产品族之间互不依赖的关系。

抽象工厂模式保证了客户端始终只使用同一个产品族中的产品，但是缺点也很明显：产品族的扩展很困难，如果要在产品族中增加新的产品，需要改动很多地方的代码。

## 样例代码

产品接口：Device，Cpu，MainBoard。Cpu和MainBoard对应两种不同的产品。

```java
public interface Device {

}

public interface Cpu extends Device {

    public void calculate();
}

public interface MainBoard extends Device {

    public void installCPU(Cpu cpu);
}
```

具体的产品类：AmdCpu，IntelCpu，AmdMainBoard，IntelMainBoard。

AmdCpu和IntelCpu构成一个产品等级结构，AmdMainBoard和IntelMainBoard构成一个产品等级结构。

Amd和Intel的产品各自构成一个产品族，每个产品族下有两种不同的产品。

```java
public class AmdCpu implements Cpu {

    @Override
    public void calculate() {
        System.out.println("AMD cpu calculate.");
    }

}

public class IntelCpu implements Cpu {

    @Override
    public void calculate() {
        System.out.println("Intel cpu calculate.");
    }

}

public class AmdMainBoard implements MainBoard {

    @Override
    public void installCPU(final Cpu cpu) {
        System.out.println("AmdMainBoard install " + cpu.getClass().getSimpleName());
    }

}

public class IntelMainBoard implements MainBoard {

    @Override
    public void installCPU(final Cpu cpu) {
        System.out.println("IntelMainBoard install " + cpu.getClass().getSimpleName());
    }

}
```

抽象工厂：DeviceFactory，用以创建对应的产品工厂对象。

具体工厂：AmdFactory，IntelFactory。

```java
public abstract class DeviceFactory {

    public static DeviceFactory getFactory(final String type) {
        if ("AMD".equalsIgnoreCase(type)) {
            return new AmdFactory();
        } else if ("Intel".equalsIgnoreCase(type)) {
            return new AmdFactory();
        }

        return null;
    }

    public abstract Cpu createCpu();

    public abstract MainBoard createMainBoard();
}

public class AmdFactory extends DeviceFactory {

    @Override
    public Cpu createCpu() {
        return new AmdCpu();
    }

    @Override
    public MainBoard createMainBoard() {
        return new AmdMainBoard();
    }

}

public class IntelFactory extends DeviceFactory {

    @Override
    public Cpu createCpu() {
        return new IntelCpu();
    }

    @Override
    public MainBoard createMainBoard() {
        return new IntelMainBoard();
    }

}
```

测试结果：

```java
public class Test {

    public static void main(final String[] args) {
        final DeviceFactory factory = DeviceFactory.getFactory("AMD");
        final Cpu cpu = factory.createCpu();
        final MainBoard mainBoard = factory.createMainBoard();
        mainBoard.installCPU(cpu);
        cpu.calculate();
    }
}
```


输出如下：

```java
AmdMainBoard install AmdCpu
AMD cpu calculate.
```

## 参考链接

* [抽象工厂模式](https://www.runoob.com/design-pattern/abstract-factory-pattern.html)
* [《JAVA与模式》之抽象工厂模式](https://www.cnblogs.com/java-my-life/archive/2012/03/28/2418836.html)