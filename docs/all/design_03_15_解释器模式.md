<!--
date: 2022-01-12T22:34:12+08:00
lastmod: 2022-01-17T22:34:12+08:00
-->
## 解释器模式（Interpreter Pattern）

解释器模式主要用来解析文本上下文，通过递归调用的方式最终将这个字符串表达式解析得到最终的结果。这种模式被用在 SQL 解析、符号处理引擎等，很多框架也都有内置的表达式，如Spring的SPEL，Hibernate的HQL，第三方库的MVEL。

表达式分为终结符表达式（Terminal Expression）和非终结符表达式（Nonterminal Expression），终结符表达式是不可再分割的最小语法单元，非终结符表达式代表了一个文法规则，一般是运算符或者关键字。

解释器模式扩展性好，易于添加新的解释表达式，但是容易引起类膨胀。由于解析过程中用的是递归调用，存在着效率问题。

## 样例代码

定义一个表达式接口：

```java
public interface Expression {

    boolean interpret(Context context);
}
```

定义一个上下文类Context，用以存放解析表达式时需要使用到的变量：

```java
public class Context {

    private final Map<VarExpression, Boolean> map = new HashMap<VarExpression, Boolean>();

    public void addVar(final VarExpression var , final boolean value){
        map.put(var, value);
    }

    public boolean lookup(final VarExpression var) throws IllegalArgumentException {
        final Boolean value = map.get(var);
        if(value == null){
            throw new IllegalArgumentException();
        }
        return value.booleanValue();
    }
}
```

定义一个变量表达式类：

```java
public class VarExpression implements Expression {

    private final String name;

    public VarExpression(final String name) {
        this.name = name;
    }

    @Override
    public boolean interpret(final Context context) {
        return context.lookup(this);
    }

}
```

定义一个符号抽象类以及子类：

```java
public abstract class SymbolExpression implements Expression {

    protected final Expression left;
    protected final Expression right;

    public SymbolExpression(final Expression left, final Expression right) {
        this.left = left;
        this.right = right;
    }

}

public class AndExpression extends SymbolExpression {

    public AndExpression(final Expression left, final Expression right) {
        super(left, right);
    }

    @Override
    public boolean interpret(final Context context) {
        return left.interpret(context) && right.interpret(context);
    }

}

public class OrExpression extends SymbolExpression {

    public OrExpression(final Expression left, final Expression right) {
        super(left, right);
    }

    @Override
    public boolean interpret(final Context context) {
        return left.interpret(context) || right.interpret(context);
    }

}

public class NotExpression extends SymbolExpression {

    public NotExpression(final Expression right) {
        super(null, right);
    }

    @Override
    public boolean interpret(final Context context) {
        return !right.interpret(context);
    }

}
```

定义一个测试类：

```java
public class Test {

    public static void main(final String[] args) {
        final String str = "a & b | !c";
        // 设置a，b，c的值
        final VarExpression a = new VarExpression("a");
        final VarExpression b = new VarExpression("b");
        final VarExpression c = new VarExpression("c");
        final Context context = new Context();
        context.addVar(a, true);
        context.addVar(b, false);
        context.addVar(c, true);
        final Expression and = new AndExpression(a, b);
        final Expression not = new NotExpression(c);
        final Expression or = new OrExpression(and, not);
        // false
        System.out.println(or.interpret(context));
    }
}
```

## 参考链接

* [解释器模式](https://www.runoob.com/design-pattern/interpreter-pattern.html)
* [《JAVA与模式》之解释器模式](https://www.cnblogs.com/java-my-life/archive/2012/06/19/2552617.html)