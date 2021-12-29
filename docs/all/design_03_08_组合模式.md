<!--
date: 2021-12-23T22:34:12+08:00
lastmod: 2021-12-29T22:34:12+08:00
-->
## 组合模式（Composite Pattern）

又叫部分整体模式，将一组类似的对象当做一个对象来处理，按照树形结构来组合对象，使客户端能以一致的方式来进行处理。

有很多场景都可以用这种树形结构来实现，比如多级菜单，城市县城之间的关系，权限组和权限角色等。在组合模式下，客户端无需关心这些类似对象之间的区别，能当做同一个对象来处理。

## 样例代码

定义一个抽象组件Component：

```java
public abstract class Component {

    // 当前组件的名字
    protected String name;

    // 当前组件的深度
    protected int depth;

    public Component(final String name) {
        this.name = name;
        depth = 0;
    }

    void add(final Component component) {
        throw new UnsupportedOperationException();
    }

    // 按照组件深度来打印
    void print() {
        throw new UnsupportedOperationException();
    }

}
```

定义叶子组件Leaf，Leaf不能添加组件：

```java
public class Leaf extends Component {

    public Leaf(final String name) {
        super(name);
    }

    @Override
    void print() {
        IntStream.range(0, depth).forEach(num -> {
            System.out.print("-");
        });
        System.out.print(name);
        System.out.println();
    }

}
```

定义组合组件Composite，Composite可以添加组件：

```java
public class Composite extends Component {

    private final ArrayList<Component> list = new ArrayList<>();

    public Composite(final String name) {
        super(name);
    }

    @Override
    void add(final Component component) {
        component.depth = this.depth + 1;
        list.add(component);
    }

    @Override
    void print() {
        IntStream.range(0, depth).forEach(num -> {
            System.out.print("-");
        });
        System.out.print(name);
        System.out.println();
        list.forEach(Component::print);
    }

}
```

定义客户端Test：

```java
public class Test {

    public static void main(final String[] args) {
        final Component root = new Composite("Root");

        final Component composite = new Composite("Composite");
        root.add(composite);
        final Component leaf1 = new Composite("Leaf1");
        composite.add(leaf1);

        final Component leaf2 = new Composite("Leaf2");
        root.add(leaf2);
        root.print();
    }
}
```

输出结果如下：

```java
Root
-Composite
--Leaf1
-Leaf2
```

## 参考链接

* [组合模式](https://www.runoob.com/design-pattern/composite-pattern.html)
* [简说设计模式——组合模式](https://www.cnblogs.com/adamjwh/p/9033547.html)