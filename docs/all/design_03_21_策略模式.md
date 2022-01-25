<!--
date: 2022-01-18T22:34:12+08:00
lastmod: 2022-01-25T22:34:12+08:00
-->
## 策略模式（Strategy Pattern）

若对象的行为或算法可以在运行时改变，则可以用策略模式。可以通过一个策略对象来改变一个对象的行为或算法，比如超市结算时在不同时间段或者面对不同客户使用不同的折扣算法。

策略模式相当于一个函数式：`y = f(x)`，x是输入值，y是输出值，不同的策略提供了不同的x到y的映射关系。

策略模式和状态模式的区别也在于此，状态模式中的状态是有限的，且状态跟对象本身息息相关。策略模式则不同，策略跟对象本身的状态无关，而是代表对象的一种行为或算法。

## 样例代码

定义折扣策略接口和实现类：

```java
public interface DiscountStrategy {

    BigDecimal discount(BigDecimal money);
}

// 原价
public class None implements DiscountStrategy {

    @Override
    public BigDecimal discount(final BigDecimal money) {
        return money;
    }
}

// 打七折
public class ThirtyPercentOff implements DiscountStrategy {

    @Override
    public BigDecimal discount(final BigDecimal money) {
        return money.multiply(BigDecimal.valueOf(0.7));
    }
}

// 打一折
public class NinetyPercentOff implements DiscountStrategy {

    @Override
    public BigDecimal discount(final BigDecimal money) {
        return money.multiply(BigDecimal.valueOf(0.1));
    }
}
```

定义超市类：

```java
public class Supermarket {

    private DiscountStrategy strategy = new None();

    public DiscountStrategy getStrategy() {
        return strategy;
    }

    public void setStrategy(final DiscountStrategy strategy) {
        this.strategy = strategy;
    }

    public BigDecimal check(final BigDecimal money) {
        return strategy.discount(money);
    }
}
```

定义测试类：

```java
public class Test {

    public static void main(final String[] args) {
        final Supermarket supermarket = new Supermarket();
        final BigDecimal money = new BigDecimal("100");
        // 100
        System.out.println(supermarket.check(money));

        supermarket.setStrategy(new ThirtyPercentOff());
        // 70.0
        System.out.println(supermarket.check(money));

        supermarket.setStrategy(new NinetyPercentOff());
        // 10.0
        System.out.println(supermarket.check(money));
    }
}

```

## 参考链接

* [策略模式](https://www.runoob.com/design-pattern/strategy-pattern.html)