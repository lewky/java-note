<!--
date: 2022-01-18T22:34:12+08:00
lastmod: 2022-01-25T22:34:12+08:00
-->
## 模板方法模式（Template Method Pattern）

在抽象类定义一个模板方法（final的，避免被重写），在该方法中定义一系列抽象方法的执行模板，子类负责重写这些抽象方法，这样在调用模板方法时就可以按照抽象类中定义的顺序来执行。

## 样例代码

定义一个抽象类模板和子类：

```java
public abstract class BaseAction {

    // 模板方法，定义为final，避免被重写
    public final void execute() {
        preprocess();
        process();
        postprocess();
    }

    abstract void preprocess();
    abstract void process();
    abstract void postprocess();
}

public class OpenDocAction extends BaseAction {

    @Override
    void preprocess() {
        System.out.println("Preprocess OpenDocAction.");
    }

    @Override
    void process() {
        System.out.println("Process OpenDocAction.");
    }

    @Override
    void postprocess() {
        System.out.println("Postprocess OpenDocAction.");
    }
}
```

定义测试类：

```java
public class Test {

    public static void main(final String[] args) {
        final BaseAction action = new OpenDocAction();
        action.execute();
    }
}
```

## 参考链接

* [模板方法模式](https://www.runoob.com/design-pattern/template-pattern.html)