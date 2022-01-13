<!--
date: 2022-01-11T22:34:12+08:00
lastmod: 2022-01-12T22:34:12+08:00
-->
## 命令模式（Command Pattern）

命令模式的目的在于将调用者和被调用者解耦，调用者发出的命令（Command）由中间类（Broker）接收，再由中间类根据命令的不同去调用对应的命令处理类。这样就可以灵活地添加不同的命令种类和对应的命令处理类，客户端不需要了解有命令如何被处理，只需要发出命令到中间类就行。

中间类持有着命令对象，可以在处理命令时加入一些其他的处理，比如排队、延迟、打印日志等等，或者是对命令进行撤销或恢复。消息队列、注册中心等都可以看做是命令模式的应用。

## 样例代码

定义命令接口和接口子类：

```java
public interface Command {

    void execute();
}

public class BuyCommand implements Command {

    String item;
    Integer num;

    public BuyCommand(final String item, final Integer num) {
        super();
        this.item = item;
        this.num = num;
    }

    @Override
    public void execute() {
        System.out.println("购买" + item + " " + num + "个");
    }
}

public class SellCommand implements Command {

    String item;
    Integer num;

    public SellCommand(final String item, final Integer num) {
        super();
        this.item = item;
        this.num = num;
    }

    @Override
    public void execute() {
        System.out.println("出售" + item + " " + num + "个");
    }
}
```

定义接收命令对象的中间类：

```java
public class Broker {

    // 存放接收的命令
    private final List<Command> list = new ArrayList<>();
    // 存放撤销的命令
    private final List<Command> remove = new ArrayList<>();

    public void addCommand(final Command command) {
        list.add(command);
    }

    // 撤销命令
    public void removeCommand(final Command command) {
        list.remove(command);
        remove.add(command);
    }

    // 恢复命令
    public void resumeCommand(final Command command) {
        remove.remove(command);
        list.add(command);
    }

    public void executeCommand() {
        list.forEach(Command::execute);
    }

}
```

定义一个测试类：

```java
public class Test {

    public static void main(final String[] args) {
        final Command command1 = new BuyCommand("苹果", 10);
        final Command command2 = new BuyCommand("橙子", 5);
        final Command command3 = new SellCommand("梨子", 3);

        final Broker broker = new Broker();
        broker.addCommand(command1);
        broker.addCommand(command2);
        broker.addCommand(command3);
        broker.executeCommand();

        // 撤销command2
        broker.removeCommand(command2);
        broker.executeCommand();

        // 恢复command2
        broker.resumeCommand(command2);
        broker.executeCommand();
    }
}
```

## 参考链接

* [命令模式](https://www.runoob.com/design-pattern/command-pattern.html)