<!--
date: 2021-04-14T23:31:12+08:00
lastmod: 2021-06-08T23:31:12+08:00
-->
## final

● 修饰类：表示该类不能被继承，比如String类。<br>
● 修饰方法：表示方法不能被重写。private 方法被隐式地指定为 final，如果在子类中定义的方法和基类中的一个 private 方法签名相同，此时子类的方法不是重写基类方法，而是在子类中定义了一个新的方法。<br>
● 修饰变量：表示变量为常量，对于基本类型，初次赋值后不能被修改；对于对象类型，初次赋值后引用不变，但对象本身可以被修改。

## static

### 静态变量

静态变量又名类变量，类变量属于类，被类的所有实例共享，可以通过类名来访问类变量。静态变量在内存中只存在一份。

实例变量属于实例，每创建一个实例都会产生对应的实例变量。

对于局部变量，只能使用final修饰，不能使用static、public、protected或者private修饰。因为局部变量是在调用方法时创建的，方法结束时会释放，这与上述所说的修饰符是矛盾的。

### 静态方法

静态方法在类加载时就已存在，不依赖于具体实例，所以静态方法必须要有实现，即静态方法不能是抽象方法。

静态方法只能访问所属类的静态变量和静态方法，方法中不能有this和super关键字，因为这两个关键字和实例对象有关。

### 静态代码块

```java
static{
    //do something
}
```

静态代码块如上所示，和静态变量、静态方法一样，在类被类加载器首次加载时被执行，之后就不会被再次执行了(除非类加载器卸载该类后重新加载这个类)。

当有多个静态代码块时按顺序执行。

### 静态内部类

非静态内部类依赖于外部类的实例，也就是说需要先创建外部类实例，才能用这个实例去创建非静态内部类。而静态内部类不需要。

静态内部类不能访问外部类的非静态的变量和方法。

```java
public class OuterClass {

    class InnerClass {
    }

    static class StaticInnerClass {
    }

    public static void main(String[] args) {
        // OuterClass.InnerClass innerClass = new InnerClass(); // 'OuterClass.this' cannot be referenced from a static context
        OuterClass outerClass = new OuterClass();
        OuterClass.InnerClass innerClass = outerClass.new InnerClass();
        OuterClass.StaticInnerClass staticInnerClass = new OuterClass.StaticInnerClass();
    }
}
```

每个类会产生一个.class文件，文件名即为类名。同样，内部类也会产生这么一个.class文件，但是它的名称却不是内部类的类名，而是有着严格的限制：外围类的名字，加上$，再加上内部类名字。

### 静态导包

在使用静态变量和方法时不用再指明 ClassName，从而简化代码，但可读性大大降低。

```java
import static com.xxx.ClassName.*;
```

### 初始化顺序

**构造代码块**

```java
public class Test{
    {
        //do something
    }
}
```

和静态代码块类似，但是没有`static`，只能出现在类中，若出现在某个方法中则是普通代码块。

构造代码块会在new实例对象时，优先于构造方法调用，也就是说，先执行完构造代码块，才会接着执行构造方法。如果在一个构造方法里调用了另一个构造方法，此时构造代码块只会被执行一次，而不是执行两次。

当有多个构造代码块时按顺序执行。

**代码块**

```java
public class Test{
    public static void main(String[] args){
        {
            //do something
        }
    }
}
```

和构造代码块类似，但是只在方法或语句中出现，执行顺序和普通语句一样，先出现就先执行。

局部代码块是为了缩短变量的生命周期，定义在局部代码块中的变量在出了代码块之后就结束其生命周期，释放内存。

**总结**

执行顺序：（优先级从高到低）

>静态代码块>main方法>构造代码块>构造方法

其中静态代码块只执行一次，构造代码块在每次创建对象是都会执行。

下面是一道与之相关的题目，执行下边的Test类，其输出的结果是什么？

```java
public class Test{
    static Test test = new Test(1);

    static{
        System.out.println("static code block");
    }

    {
        System.out.println("constructor code block");
    }

    Test(){
        System.out.println("constrctor method");
        System.out.println("a=" + a + ",b=" + b);
        a++;
        b++;
    }

    Test(int a){
        this();
        this.a = a;
        System.out.println("constrctor method2");
        System.out.println("a=" + a + ",b=" + b);
    }

    public static void main(String[] args){
        {
            System.out.println("code block");
        }

        method();

        Test test2 = new Test();
        method();
    }

    public static void method(){
        System.out.println("static method");
    }

    int a = 10;
    static int b = 100;
}
```

答案如下：

```java
constructor code block
constrctor method
a=10,b=0
constrctor method2
a=1,b=1
static code block
code block
static method
constructor code block
constrctor method
a=10,b=100
static method
```

### 静态内部类和非静态内部类的区别

内部类分为：成员内部类、局部内部类、匿名内部类和静态内部类。前三个是非静态内部类，依赖于外部类的实例。可以把内部类当做一个普通的变量，从变量的作用域和生命周期来进行区分。

成员内部类：

● 可以无条件访问外部类的所有字段和方法（包括private、static）。<br>
● 可以拥有private、default、protecte、public访问权限修饰符。<br>
● 成员内部类的生命周期无法在编译期确定下来，所以不能存在static的变量和方法，但是允许定义静态常量（必须是编译时常量），因为其可以在编译期被优化成字面量。

局部内部类：

● 定义在方法里的内部类，跟局部变量一样不能拥有private、default、protecte、public以及static修饰符。<br>
● 只能访问局部final变量

匿名内部类：

● 一般用来new一个接口的匿名实现类对象，从java8开始可以用lambda来代替<br>
● 同样不能拥有访问权限修饰符以及static修饰符<br>
● 匿名内部类是唯一没有构造方法的类<br>
● 只能访问局部final变量

静态内部类：

● 不依赖于外部类的实例，不能访问外部类的非静态的变量和方法。<br>
● 可以存在静态的变量和方法，也可以存在非静态的变量和方法。

### 为什么局部内部类和匿名内部类只能访问局部final变量

跟变量的生命周期有关。

局部内部类和匿名内部类虽然定义在方法内部，但跟外部类是同一级别的。如果这两种内部类访问了局部变量，当方法执行结束时，局部变量被释放了，内部类却不会跟着被销毁，它依然保留着对这个局部变量的引用。这样就会出现该局部变量生命周期不一致的问题，为了解决这个问题，内部类实际上是保存着局部变量的一份拷贝，为了避免两边变量值不一致的情况，于是就要求该局部变量必须是final的。

从java8开始增加了`Effectively final`特性，方法内部类访问到的方法的局部变量允许不修饰为final。只要在初次赋值后不再改变局部变量的值，编译器会将其视为最终final的。如果重新赋值，依然是会报错`Local variable limit defined in an enclosing scope must be final or effectively final`。

### 成员内部类的优点和特性

● 内部类可以拥有比非内部类更多的访问权限，如protected、private权限，可以更好地实现封装，除了其外部类，其他类都不能访问。<br>
● 成员内部类天然拥有着外部类对象的引用，能访问外部类的所有成员和方法。<br>
● 能够非常好地解决多重继承的问题，通过定义多个内部类并继承不同的父类来间接实现多重继承。<br>
● 可以让多个内部类以不同方式实现同一个接口，或者继承同一个类。比如迭代器模式。

## instanceof

instanceof严格来说是Java中的一个双目运算符，用来测试一个对象是否为一个类的实例。用法如：
```java
boolean result = obj instanceof Class
```

其中 obj 为一个对象（不能是基本数据类型），Class 表示一个类或者一个接口，当 obj 为 Class 的对象，或者是其直接或
间接子类，或者是其接口的实现类，结果result 都返回 true，否则返回false。

注意：编译器会检查 obj 是否能转换成右边的class类型，如果不能转换则直接报错，如果不能确定
类型，则通过编译，具体看运行时定。

该关键字和另一个API`isAssignableFrom()`类似，如下：
```java
List.class.isAssignableFrom(ArrayList.class);	//true
```

用来判断当前Class对象所表示的类或接口与指定的Class参数所表示的类或接口是否相同，或是否是其接口或者父类。如果是返回true，否则返回false。这个可以用来判断接口或者抽象类。

## 参考链接

* [四、关键字](http://www.cyc2018.xyz/Java/Java%20%E5%9F%BA%E7%A1%80.html#四、关键字)
* [Java中静态内部类和非静态内部类到底有什么区别？](https://www.zhihu.com/question/285579995)
* [Java内部类和类访问权限控制](https://blog.csdn.net/qq_32362489/article/details/91881579)