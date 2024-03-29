<!--
date: 2021-03-27T23:48:12+08:00
lastmod: 2021-05-18T23:48:12+08:00
-->
## 注解（Annotation）

注解是java 5引入的新特性， 用于提供程序相关的元数据（metadata）。

注解用`@interface`来定义，本质上还是一个接口，只不过比较特殊，编译器会对其进行一些处理。注解里可以定义若干个方法（可以不定义任何方法，相当于该注解只是一个不存储元数据的标志）。如下是一个自定义的注解：

```java
@Documented
@Inherited
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.TYPE)
public @interface TestAnnotation1 {

    String value();

    String name() default "";

    String[] vars() default {};

}
```

注解里的方法相当于一个个属性，在使用注解时通过给这些属性赋值来为程序提供元数据。

`default`关键字表示默认值，**没有使用该关键字表示该属性是必填项**。在上述代码中，给`name`设置了空字符串`""`的默认值，给`vars`设置了空数组`{}`的默认值。

**注解的属性类型目前只支持8种基本数据类型、String、Class类型、Annotation注解类型、Enum枚举类型、以及由前面类型构成的一维数组类型。**并且只能用`public`或`默认（default）`这两个访问权修饰。

给注解的属性赋值时，如果只给一个属性赋值，且注解存在一个名为`value`的方法，则可以不指定属性名字进行赋值。如下：

```java
// 只给一个属性赋值时可以不指定属性名，会默认给value属性赋值
@TestAnnotation1("test1")
public class Test1 {
}

@TestAnnotation1(value="test2", name="test2", vars={"test2"})
class Test2 {
}
```

## 元注解（meta-annotation）

元注解用于给注解提供相关的元数据，换言之，元注解是专门用于注解的注解。如果要自定义注解，就需要使用到这些元注解。

### `@Retention`

用以定义注解的保留级别（即生命周期），支持以下三个值：

● RetentionPolicy.SOURCE<br>
● RetentionPolicy.CLASS<br>
● RetentionPolicy.RUNTIME

`SOURCE`表明注解只保留于源码中，编译之后的字节码文件不存在该注解。这种保留策略一般用于编译期的检查，比如常见的`@Override`、`@SuppressWarnings`。

`CLASS`表明注解可以保留到字节码文件中，在类加载时丢弃，也就是说在运行期是不可见的。默认使用这个保留策略。这种保留策略用于编译期处理字节码文件，比如MapStruct框架就提供了大量这种保留策略的注解来提供代码生成。

`RUNTIME`表明注解可以保留到运行期，所以可以用反射来获取注解信息，自定义注解一般都使用这种策略。

### `@Target`

用以定义注解的适用范围，即注解可以被使用在哪个地方，取值如下：

● ElementType.TYPE，作用于类、接口（包括注解类型）、枚举类型<br>
● ElementType.FIELD，作用于字段（包括枚举常量，即枚举值）<br>
● ElementType.METHOD，作用于方法<br>
● ElementType.PARAMETER，作用于方法参数<br>
● ElementType.CONSTRUCTOR，作用于构造器<br>
● ElementType.LOCAL_VARIABLE，作用于局部变量<br>
● ElementType.ANNOTATION_TYPE，作用于注解类型<br>
● ElementType.PACKAGE，作用于包<br>
● ElementType.TYPE_PARAMETER（jdk1.8新增的注解），作用于类型参数，即可以作用于泛型类、泛型接口和泛型方法<br>
● ElementType.TYPE_USE（jdk1.8新增的类型注解），作用于任何使用类型的场合，范围比`TYPE_PARAMETER`更大，如下场景都能使用：

* 创建对象
* 类型转换
* 使用extends继承类或者implements实现接口
* 使用throws 声明抛出异常 

#### 类型注解

**注意，声明了`ElementType.TYPE_USE`的注解是类型注解，无法作用于返回值是`void`的方法。**

```java
@Target(ElementType.TYPE_USE)
public @interface NotNull {}

@NotNull
class TestNotNull<@NotNull T> extends @NotNull HashMap<String, String> implements @NotNull Serializable {

    @NotNull
    private T t;

    @NotNull
    private String test;

    public static void main(@NotNull String[] args) throws @NotNull Exception {
        @NotNull
        Object object;

        // 创建对象使用类型注解
        object = new @NotNull String("小钻风");

        // 强制类型转换使用类型注解
        String string = (@NotNull String) object;
    }

    @NotNull
    public String test1() {
        return "";
    }

    // 编译报错，因为类型注解必须作用于使用类型的场合，该方法无返回类型所以不能使用类型注解
    @NotNull    // Type annotation is illegal for a method that returns void
    public void test2() {}
}
```

### `@Documented`

这是一个标记注解，用以指明注解信息是否添加到Java Doc中。

### `@Inherited`

同样是一个标记注解，用以指明注解可以被继承。如果一个使用了`@Inherited`的注解类型被用于一个类，则这个注解将同样作用于该类的子类。

### `@Repeatable`

jdk1.8新增的元注解，表示可以对同一个元素多次使用相同的注解。其实就是个类似语法糖的东东，在编译期会自动把这个元注解修饰的注解转成另一个被指定的注解，这个新的注解只有一个value属性，存放的是一个注解数组，数组里放的就是被元注解修饰的注解。

比如JPA的`@AttributeOverride`就用了这个元注解：

```java
@Repeatable(AttributeOverrides.class)
@Target({TYPE, METHOD, FIELD}) 
@Retention(RUNTIME)
public @interface AttributeOverride {

    String name();

    Column column();
}
```

## 注解的原理

注解本质上是一个特殊的接口，在编译之后会自动继承`Annotation`接口，如下：

```java
@Retention(RetentionPolicy.RUNTIME)
public @interface TestAnnotation {}
```

用`javap -p -v TestAnnotation.class`命令查看编译后的字节码文件：

```java
D:\workspace1\my-test\target\classes\cn\lewky>javap -p -v TestAnnotation.class
Classfile /D:/workspace1/my-test/target/classes/cn/lewky/TestAnnotation.class
  Last modified 2021-5-3; size 298 bytes
  MD5 checksum 13bb45528202675aea6b1cdfc82900bd
  Compiled from "TestAnnotation.java"
public interface cn.lewky.TestAnnotation extends java.lang.annotation.Annotation
  minor version: 0
  major version: 52
  flags: ACC_PUBLIC, ACC_INTERFACE, ACC_ABSTRACT, ACC_ANNOTATION
Constant pool:
   #1 = Class              #2             // cn/lewky/TestAnnotation
   #2 = Utf8               cn/lewky/TestAnnotation
   #3 = Class              #4             // java/lang/Object
   #4 = Utf8               java/lang/Object
   #5 = Class              #6             // java/lang/annotation/Annotation
   #6 = Utf8               java/lang/annotation/Annotation
   #7 = Utf8               SourceFile
   #8 = Utf8               TestAnnotation.java
   #9 = Utf8               RuntimeVisibleAnnotations
  #10 = Utf8               Ljava/lang/annotation/Retention;
  #11 = Utf8               value
  #12 = Utf8               Ljava/lang/annotation/RetentionPolicy;
  #13 = Utf8               RUNTIME
{
}
SourceFile: "TestAnnotation.java"
RuntimeVisibleAnnotations:
  0: #10(#11=e#12.#13)
```

可以发现，`@interface`变成了`interface`，并且自动添加了`extends java.lang.annotation.Annotation`。

而注解的使用是和反射绑定在一块的，只有通过反射才能在运行期获取注解相关的信息。但注解本质是一个接口，通过反射获取的注解对象，实际上是这个注解接口的动态代理类。

Java通过基于接口的jdk动态代理来生成注解的实现类，这也是为什么注解是一个接口，而不是一个类。注解接口会默认继承`Annotation`接口，也是因为代理对象需要额外调用一些公共的方法。通过代理对象调用注解的方法时，会最终调用到`AnnotationInvocationHandler`的`invoke`方法。注解接口的方法名字，会在`invoke`方法中作为一个key从`memberValues`中获取对应的属性值，而`memberValues`的来源是Java常量池。

## 注解与反射

### 获取注解的属性值

常用的注解反射相关的方法：

● `isAnnotationPresent(Class<? extends Annotation> annotationClass)`用于判断是否存在某个注解。<br>
● `getAnnotation(Class<A> annotationClass)`用于获取指定的注解，包括继承的注解。<br>
● `getDeclaredAnnotation(Class<A> annotationClass)`用于获取指定的注解，**不包括继承的注解**。<br>
● `getAnnotations()`获取所有的注解，包括继承的注解。<br>
● `getDeclaredAnnotations()`获取所有的注解，**不包括继承的注解**。<br>
● `getAnnotationsByType(Class<A> annotationClass)`，jdk1.8新增的方法，用于获取指定的注解，包括继承的注解。主要是针对可重复注解，所以返回值是一个注解数组。<br>
● `getDeclaredAnnotationsByType(Class<A> annotationClass)`，jdk1.8新增的方法，用于获取指定的注解，**不包括继承的注解**。主要是针对可重复注解，所以返回值是一个注解数组。<br>

## 动态修改注解的属性值

可以在运行期通过反射来修改注解的属性值，如下：

```java
@TestAnnotation(name = "TestA")
public class Test {

    public static void main(String[] args) throws Exception {
        // Get annotation from EntityMongo class
        final TestAnnotation annotation = Test.class.getAnnotation(TestAnnotation.class);
        if (annotation == null) {
            return;
        }
        // 原本的值为：TestA
        System.out.println(annotation.name());

        // Get Invocation Handlder：sun.reflect.annotation.AnnotationInvocationHandler
        final InvocationHandler handler = Proxy.getInvocationHandler(annotation);
        // Get private field: memberValues
        final Field field = handler.getClass().getDeclaredField("memberValues");
        field.setAccessible(true);
        // Get memberValues field value and amend property value
        final Map<String, Object> memberValues = (Map<String, Object>) field.get(handler);
        String newValue = "TestB";
        // 注解默认必有的一个属性：value
        memberValues.put("value", newValue);
        // 自定义注解TestAnnotation中的属性：name
        // 属性名和注解里的方法名字是一样的
        memberValues.put("name", newValue);
        // 现在的值为：TestB
        System.out.println(annotation.name());
    }

}

@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@interface TestAnnotation {
    String name() default "";
}
```

如果看懂了前文提及的注解原理，自然可以理解这种动态修改注解属性值的方式。

## 参考链接

* [注解Annotation实现原理与自定义注解例子](https://www.cnblogs.com/acm-bingzi/p/javaAnnotation.html)
* [Java 注解完全解析](https://www.jianshu.com/p/9471d6bcf4cf)
* [Java注解总结](https://www.jianshu.com/p/73a778c1b5b7)
* [类型注解](https://blog.csdn.net/fishgeneral/article/details/100141131)