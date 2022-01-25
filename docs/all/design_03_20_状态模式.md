<!--
date: 2022-01-18T22:34:12+08:00
lastmod: 2022-01-25T22:34:12+08:00
-->
## 状态模式（State Pattern）

当对象的行为随着自身状态的改变而改变时，可以使用状态模式，一般不超过5个状态，避免类膨胀。

## 样例代码

定义状态接口和实现类：

```java
public interface State {

    void doAction();
}

public class OpenState implements State {

    @Override
    public void doAction() {
        System.out.println("The light is opening.");
    }
}

public class CloseState implements State {

    @Override
    public void doAction() {
        System.out.println("The light is closed.");
    }
}

public class FlashState implements State {

    @Override
    public void doAction() {
        System.out.println("The light is flashing.");
    }
}
```

定义电灯类，持有着所有状态：

```java
public class Light {

    public static final State OPEN = new OpenState();
    public static final State CLOSE = new CloseState();
    public static final State FLASH = new FlashState();

    private State state;

    public Light() {
        this.state = CLOSE;
    }

    public State getState() {
        return state;
    }

    public void setState(final State state) {
        this.state = state;
    }

    public void interact() {
        state.doAction();
    }

}
```

定义测试类：

```java
public class Test {

    public static void main(final String[] args) {
        final Light light = new Light();
        light.interact();
        light.setState(Light.OPEN);
        light.interact();
        light.setState(Light.FLASH);
        light.interact();
    }

}
```

## 参考链接

* [状态模式](https://www.runoob.com/design-pattern/state-pattern.html)