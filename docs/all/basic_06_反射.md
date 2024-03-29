<!--
date: 2021-04-19T22:34:12+08:00
lastmod: 2021-06-08T22:34:12+08:00
-->
## 反射（Reflection）

每个类都有一个对应的`Class`对象，保存着与之相关的类信息。每编译一个新类，就会生成一个`.class`文件，该文件中保存着Class对象。（`.class`文件有点像是Class对象的序列化文件，但二者并不是一回事）

类加载相当于Class对象的加载。类在第一次使用时才会动态加载到JVM中，可以用`Class.forName()`来加载类，一般都用这个方法来加载数据库的驱动类。

反射可以提供运行时的类信息，并且这个类可以在运行时才加载进来，甚至在编译时期该类的`.class`文件不存在时也可以加载进来。

Class类和`java.lang.reflect`类库一起对反射提供了支持，主要涉及到下面三个类：

● Field类，用以操作对象的字段。<br>
● Method类，用以操作对象的方法。<br>
● Constructor类，用以操作对象的创建。

## 获取Class对象的方式

● Class.forName()<br>
● 类名.class<br>
● 对象实例.getClass()<br>
● 基本类型也有对应的Class对象，可以用基本类型.class获取，也可以用其包装类.TYPE来获取。基本类型的Class对象不等同于包装类的Class对象。

```java
System.out.println(int.class == Integer.TYPE);  //true
System.out.println(int.class == Integer.class); //false
```

## 反射优缺点

优点：

● 能够运行时动态获取类的实例，提高灵活性；<br>
● 与动态编译结合

尽管反射非常强大，但也不能滥用。如果一个功能可以不用反射完成，那么最好就不用。

缺点：

● 性能较低，由于涉及到动态类型的解析，所以JVM无法对这部分代码进行优化。<br>
● 相对不安全，破坏了封装性，因为可以通过反射获取私有的变量和方法，还可以通过反射越过泛型检查。

如何优化反射性能：

● 通过`setAccessible(true)`关闭JDK的安全检查来提升反射速度<br>
● 多次创建一个类的实例时，有缓存会快很多<br>
● ReflectASM工具类，通过字节码生成的方式加快反射速度

* [ReflectASM-invoke，高效率java反射机制原理](https://www.cnblogs.com/tohxyblog/p/8661090.html)

## 常用的反射API

```java
Class clazz = Class.forName("test.Test");

// 获取自身类中所有的字段、方法和构造方法
Field[] field=clazz.getDeclaredFields();
Method[] method = clazz.getDeclaredMethods();
Constructor[] constructor = clazz.getDeclaredConstructors();

// 获取自身类中以及父类中所有的public的字段、方法和构造方法
Field[] field=clazz.getFields();
Method[] method = clazz.getMethods();
Constructor[] constructor = clazz.getConstructors();
```

## 参考链接

* [七、反射](http://www.cyc2018.xyz/Java/Java%20%E5%9F%BA%E7%A1%80.html#七、反射)
