<!--
date: 2021-04-19T22:34:12+08:00
lastmod: 2021-06-23T22:34:12+08:00
-->
## 泛型（Generic）

泛型：把类型明确的工作推迟到创建对象或调用方法时再明确的特殊类型。

参数化类型：把类型当作参数来传递，这意味着参数化类型不能是基本数据类型，需要用对应的包装类来代替。

相关概念：

● `ArrayList<E>`中的`E`是类型参数变量（typeVariable，也叫泛型参数），除了E之外，也可以是任意标识符。其实就是一个变量名，常用的一般有`E（Element）、T（Type）、K（Key）、V（Value）`等。<br>
● `ArrayList<Integer>`中的`Integer`称为实际类型参数（ActualTypeArgument），上面的`E`相当于形参，这里的`Integer`相当于实参<br>
● `ArrayList<E>`整个被称为泛型类型（泛型类，GenericType）<br>
● `ArrayList<Integer>`整个被称为参数化的类型（ParameterizedType）<br>
● `ArrayList`被称为原生类型（RawType）<br>
● 泛型类中允许定义泛型内部类，那么外部泛型类的原生类型，被称为内部泛型类的拥有者类型（OwnerType)

## 泛型的好处

● 泛型的本质是为了参数化类型，在不创建新类型的情况下，通过泛型指定的不同类型来限制形参的具体类型，以此提高代码的复用性。<br>
● 安全性。泛型可以在编译时检查类型安全，避免在运行时发生类转换异常ClassCastException。<br>
● 可读性，代码更加简洁。泛型会进行隐式类型转换，比如在使用集合时，无需进行强制类型转换。比如遍历一个指定了泛型的集合时，就可以用增强for来进行遍历。

## 泛型类、泛型方法

只有声明了泛型参数的类才是泛型类（泛型接口同理），只有声明了泛型参数的方法才是泛型方法。也就是说，泛型类中使用了泛型的方法并不是泛型方法，**泛型类声明的泛型参数和泛型方法声明的泛型参数可以重名，但是二者并不存在任何关系。**

```java
// 泛型类
class TestGeneric<T> {

    // 不是泛型方法
    public T getT1(T t) {
        return t;
    }
    
    // 泛型方法，这里的T和泛型类的T完全没有关系
    public <T> T getT2(T t) {
        return t;
    }

}

// 不是泛型类
class TestGeneric2<Integer> {

    // 不是泛型方法
    public Integer getT1(Integer t) {
        return t;
    }
    
	// 泛型方法
    public <T> T getT2(T t) {
        return t;
    }

}
```

泛型是在类实例化或者方法调用时才明确类型的：

● 对于泛型类的泛型参数，需要在类实例化时才能明确类型；<br>
● 对于泛型方法的泛型参数，需要在方法调用时才能明确类型。

如果要继承泛型类，写法如下：

```java
// 定义一个泛型类
class TestGeneric<T> {

}

// 定义一个参数化类型
class TestChild extends TestGeneric<Integer> {

}

// 定义一个泛型类子类，子类必须声明泛型参数，否则编译报错
class TestChild2<T> extends TestGeneric<T> {

}
```

实现泛型接口的写法也是同理。

## 泛型原理：类型擦除

Java的泛型是伪泛型。泛型是提供给javac编译器使用的，在编译期间，泛型信息会被擦除掉，生成的class文件中将不再带有泛型信息。但是泛型擦除并不是完全擦除掉所有的泛型信息，参数化类型的泛型信息（相当于元数据）会被保留下来，可以通过反射获取到。

* [Java的泛型擦除留下了什么？](https://blog.csdn.net/huluwaaaa/article/details/102699494)

Java是向前兼容的，泛型在Java5引入，需要兼容Java5之前的版本，这也是Java泛型需要类型擦除的原因之一。

在编译期，`ArrayList<Integer>`和`ArrayList`对于编译器是两个不同的类型；但是经过了泛型擦除后，在运行期，对于JVM来说就是一样的类型。此外，编译期会在使用泛型的地方自动生成类型转换的字节码，所以在使用泛型时无需进行强制类型转换。

## 泛型擦除导致的多态冲突

由于泛型擦除，会导致运行期的多态冲突。如下：

```java
class TestGeneric<T> {

    public T test(T t) {
        return t;
    }
}

class TestChild extends TestGeneric<Integer> {

    @Override
    public Integer test(Integer t) {
        return t;
    }
}
```

`@Override`表明子类重写了父类的方法，但实际上，经过泛型擦除之后，父类的`test方法`变成如下：

```java
public Object test(Object t) {
    return t;
}
```

如果是在非泛型类的继承关系中，其实这样并不是子类重写了父类的方法，而是子类重载了这个`test方法`。如下：

```java
class Animal {
    
    public Object test(Object t) {
        return t;
    }

}

class Cat extends Animal {
    
    // 编译出错，编译器认为这是方法重载而非重写，不能使用@Override
    @Override
    public Integer test(Integer t) {    // The method test(Integer) of type Cat must override or implement a supertype method
        return t;
    }
}
```

本意是想要重写父类方法，结果经过泛型擦除后实际上是方法重载。为了解决这个泛型擦除在继承关系中带来的多态冲突，编译器在生成泛型类子类的字节码时会生成桥方法（Bridge Method），用以桥接原来的方法。如果去看泛型类的子类字节码文件，会发现每一个使用了泛型的被重写的父类方法都多出来一个对应的被`bridge`修饰的桥方法。

对于上述的`TestChild`类，经过编译之后相当于变成如下：
```java

class TestChild extends TestGeneric<Integer> {

	// 原本的@Override注解被移除，实际上是方法重载
    public Integer test(Integer t) {
        return t;
    }

	// 编译器生成的桥方法，重写了父类的方法，并直接调用子类里重载的方法
	// 这样就可以实现泛型类在继承体系中的运行时多态
	public Object test(Object t) {
        return test((Integer)t);
    }
}
```

* [Java泛型中的桥方法(Bridge Method)](https://blog.csdn.net/hao_yan_bing/article/details/89447792)

## 泛型与反射

泛型的类型检查只是在编译期生效，所以可以在运行期通过反射往一个泛型集合中加入限制类型以外的元素。

由于泛型擦除的原因，虽然无法在运行期通过反射动态获取一个泛型类的实际类型，但依然可以用反射来获取参数化类型的泛型信息。**注意，泛型类和参数化类型不是一个东西。**

* [Java中的泛型会被类型擦除，那为什么在运行期仍然可以使用反射获取到具体的泛型类型？](https://www.zhihu.com/question/346911525/answer/830285753)

```java
class TestGeneric<T> {

    public TestInner child;

    public static void main(final String[] args) {
        final TestChild testChild = new TestChild();
        testChild.child = testChild.new TestInner();
        testChild.getArray(testChild, 3);
        System.out.println("===============================");
        testChild.getArray(testChild.child, 3);
    }

    class InnerGeneric<E> {

    }

    class TestInner extends InnerGeneric<Integer> {

    }

}

class TestChild extends TestGeneric<Integer> {

    public Integer[] getArray(final Object obj, final int length) {
      final ParameterizedType type = (ParameterizedType) obj.getClass().getGenericSuperclass();
      final Class <?> clazz = (Class <?>) type.getActualTypeArguments()[0];
      System.out.println("Class: " + obj.getClass().getName());
      System.out.println("ParameterizedType: " + type.getTypeName());
      System.out.println("RawType: " + type.getRawType());
      System.out.println("OwnerType: " + type.getOwnerType());
      System.out.println("ActualTypeArguments' number: " + type.getActualTypeArguments().length);
      System.out.println("ActualTypeArgument Class: " + clazz.getName());
      return (Integer[])Array.newInstance(clazz, length);
  }

}
```

运行结果如下：

```java
Class: test.TestChild
ParameterizedType: test.TestGeneric<java.lang.Integer>
RawType: class test.TestGeneric
OwnerType: null
ActualTypeArguments' number: 1
ActualTypeArgument Class: java.lang.Integer
===============================
Class: test.TestGeneric$TestInner
ParameterizedType: test.TestGeneric<T>.InnerGeneric<java.lang.Integer>
RawType: class test.TestGeneric$InnerGeneric
OwnerType: test.TestGeneric<T>
ActualTypeArguments' number: 1
ActualTypeArgument Class: java.lang.Integer
```

如果理解了前文提及的相关概念，那么自然也能理解这些泛型反射的方法。

## 泛型集合

泛型经常被用于集合，一个泛型集合不允许被加入指定类型以外的对象：

```java
public void test() {
    List<Dog> list = new ArrayList<>();
    list.add(new Dog());

    // 编译错误，不能添加指定类型以外的对象
    list.add(new Cat());  // The method add(Dog) in the type List<Dog> is not applicable for the arguments (Cat)

    Animal animal = new Dog();
    // 编译错误，虽然Animal是Dog的父类型，但依然不能添加进去集合里，因为Animal类型不一定能安全地转型为Dog类型。
    list.add(animal);  // The method add(Dog) in the type List<Dog> is not applicable for the arguments (Animal)

    BigDog bigDog = new BigDog();
    // 因为BigDog是Dog的子类型，一定可以转型为Dog类型，所以允许加入集合
    list.add(bigDog);  // ok
}
```

因为泛型集合在读取元素或者添加元素时，会有类型转换操作（在源码底层里进行的强制类型转换），如果不能把一个元素安全地进行类型转换，那就不能被添加到集合中。

此外，作用于编译期的类型检查是针对于引用类型的，跟引用类型指向的实际对象并无关系。如下：

```java
public void test() {
    // 指向一个泛型集合
    List list1 = new ArrayList<Dog>();
    list1.add(new Cat());  // ok

    // 指向一个非泛型集合
    List<Dog> list2 = new ArrayList();
    // 编译错误
    list2.add(new Cat());  // The method add(Dog) in the type List<Dog> is not applicable for the arguments (Cat)

    // 指向一个泛型集合
    List<Dog> list3 = new ArrayList<>();
    // 编译错误
    list3.add(new Cat());  // The method add(Dog) in the type List<Dog> is not applicable for the arguments (Cat)
}
```

## 泛型通配符

泛型参数除了可以是Java标识符之外，还可以指定为`?`无界通配符，用以表示不确定的Java类型。

**但是`?`不可用于声明泛型，只能用于使用泛型的场合**。因为`?`并不是合法的Java标识符，不可用于声明，只能作为实际类型参数来使用，效果相当于Object。通配符类型同样无法实例化。

```java
// 不能用通配符声明泛型类
class TestGeneric<?> {  // Syntax error on token "?", Identifier expected

    // 不能用通配符声明泛型方法
    public <?> void test1(final ? test) {  // Syntax error on token "?", byte expected

    }
    
    public void test2(final List<?> test) {  // ok
        // 不能实例化
        ? obj = new ?();  // Syntax error on token "?", invalid ClassType
		// 不能实例化
		List<?> list = new ArrayList<?>();  // Cannot instantiate the type ArrayList<?>
    }
}
```

### 无界通配符`<?>`

泛型使用最多的场景是集合，然而对于泛型集合来说，每个集合之间都是完全独立的：

```java
class Animal {}

class Cat extends Animal {}

class Dog extends Animal {}

class BigDog extends Dog {}

class TestGeneric {

    public static void main(final String[] args) {
        List<Dog> dogs = new ArrayList<>();
        List<Animal> animals = new ArrayList<>();
        // 编译错误
        dogs = animals; // Type mismatch: cannot convert from List<Animal> to List<Dog>
        // 编译错误
        animals = dogs; // Type mismatch: cannot convert from List<Dog> to List<Animal>
        
		// 在运行期这两个泛型集合的类型是同一个类对象
        System.out.println(dogs.getClass() == animals.getClass());  // true
        
        // 不能添加Animal对象
        dogs.add(new Animal());  // The method add(Dog) in the type List<Dog> is not applicable for the arguments (Animal)
        dogs.add(new Dog());  // ok
        
        animals.add(new Animal());  // ok
        animals.add(new Dog());  // ok
    }
}
```

可以看到，虽然`Dog`是`Animal`的子类，但是`List<Animal>`和`List<Dog>`这两个集合之间并不存在任何关系，不能把一个`List<Dog>`对象直接赋值给一个`List<Animal>`引用。于是这就会引发一个问题，如果一个方法的参数是泛型集合，就很容易出现类型不匹配的情况。为了避免这种情况，也更有利于代码的复用和简洁，就有了无界通配符`<?>`，如下：

```java
class TestGeneric {

    public static void test1(final List<Animal> list) {
        
    }

    public static void test2(final List<?> list) {
        
    }

    public static void main(final String[] args) {
        List<Dog> dogs = new ArrayList<>();
        List<Animal> animals = new ArrayList<>();
        List<?> list = new ArrayList<>();

        // 编译错误，无法调用
        test1(dogs);    // The method test1(List<Animal>) in the type TestGeneric is not applicable for the arguments (List<Dog>)
        test1(animals); // ok
        // 编译错误，无法调用
        test1(list);    // The method test1(List<Animal>) in the type TestGeneric is not applicable for the arguments (List<capture#2-of ?>)
        
        test2(dogs);    // ok
        test2(animals); // ok
        test2(list);    // ok

        dogs.add(new Dog());	// ok
        // 编译错误
        list.add(new Animal()); // The method add(capture#2-of ?) in the type List<capture#2-of ?> is not applicable for the arguments (Animal)
        // 编译错误
        list.add(new Dog());   // The method add(capture#3-of ?) in the type List<capture#3-of ?> is not applicable for the arguments (Dog)
        list.add(null); // ok

        list = dogs;
        // 将泛型通配符集合赋值为普通的泛型集合后，依然不能添加null以外的元素
        list.add(new Dog());   // The method add(capture#6-of ?) in the type List<capture#6-of ?> is not applicable for the arguments (Dog)

		// 泛型通配符集合中读取的元素为Object类型
        Object object = list.get(0);
		// 将读取的元素强转为原本的类型
        Dog dog = (Dog) list.get(0);
    }
}
```

可以发现，泛型通配符集合虽然被任意的泛型集合对象所赋值，但是却不能往这个集合里添加null以外的任何元素，只能读取这个集合的元素，并且被读取的元素都是Object类型。

这是因为经过泛型擦除后，通配符被擦除成了Object类型。也就是说，可以把一个无界通配符`<?>`看成是上界通配符`<? extends Object>`。

### 上界通配符`<? extends T>`

使用上界通配符可以将实际类型参数限制为指定的类型或者指定类型的子类，经过泛型擦除后，上界通配符被擦除成了指定的类型，即泛型擦除会保留上界。

需要注意的是，这里的上界虽然用了`extends`关键字，但实际上和类的继承不太一样。对于泛型来说，`extends`之后的具体类型可以是类，也可以是接口。（也就是说如果上界指定的是一个接口，也必须用`extends`关键字，而不是`implements`）

```java
public void test() {
    List<? extends Serializable> list1;  // ok
    // 编译错误
    List<? implements Serializable> list2;  // Incorrect number of arguments for type List<E>; it cannot be parameterized with arguments <?, Serializable>
}
```

泛型参数`T`也可以指定上界，但只能用于声明泛型的场景，这是和通配符的一个重要区别之一：

```java
// 用T声明一个有上界的泛型方法
public <T extends Serializable> void test(T t) {
    // 编译错误
    List<T extends Serializable> list1;  // Incorrect number of arguments for type List<E>; it cannot be parameterized with arguments <T, Serializable>
    
    List<? extends Serializable> list2;  // ok
}
```

### 下界通配符`<? super T>`

使用下界通配符可以将实际类型参数限制为指定的类型或者指定类型的父类，经过泛型擦除后，下界通配符会被擦除成Object类型，但是在编译期不允许传入非指定类型或其父类以外的类型。

泛型参数`T`不可以指定下界，这是和通配符的另一个区别：

```java
// 编译错误
public <T super Serializable> void test(T t) {  // Syntax error on token "super", , expected
    
}
```

### PECS原则

当泛型集合使用了有边界的通配符时，存在PECS原则（Producer Extends Consumer Super），如下：

```java
class TestGeneric {

    public static <T> void main(String[] args) {
        List<Dog> dogs = new ArrayList<>();
        dogs.add(new Dog());

        // 上界通配符泛型集合
        List<? extends Animal> list1 = dogs;

        /* 往上界通配符泛型集合添加元素 */
        // 编译错误，不能加入null以外的任意类型对象
        list1.add(new Dog());  // The method add(capture#1-of ? extends Animal) in the type List<capture#1-of ? extends Animal> is not applicable for the arguments (Dog)
        // 编译错误，不能加入null以外的任意类型对象
        list1.add(new Animal());  // The method add(capture#2-of ? extends Animal) in the type List<capture#2-of ? extends Animal> is not applicable for the arguments (Animal)
        list1.add(null);  // ok

        /* 从上界通配符泛型集合获取元素 */
        // 获取的元素默认是上界类型
        Animal animal = list1.get(0);  // ok
        // 强转为原来的类型
        Dog dog = (Dog) list1.get(0);  // ok

        // ----------------------------
        // 下界通配符泛型集合
        List<? super Dog> list2 = new ArrayList<>();

        /* 往下界通配符泛型集合添加元素 */
        // 编译错误，只能加入下界类型及其子类型
        list2.add(new Animal());  // The method add(capture#6-of ? super Dog) in the type List<capture#6-of ? super Dog> is not applicable for the arguments (Animal)
        list2.add(new Dog());  // ok
        list2.add(new BigDog());  // ok
        list2.add(null);  // ok

        /* 从下界通配符泛型集合获取元素 */
        // 获取的元素默认是Object类型
        Object object = list2.get(0);  // ok
        // 强转为原来的类型
        dog = (Dog) list2.get(0);  // ok
        BigDog bigDog = (BigDog) list2.get(1);  // ok
    }

}
```

对于上界通配符的泛型集合，不允许添加null以外的任何类型对象，因为泛型擦除导致JVM无法确定该集合究竟存放了什么类型的元素，只知道集合里的元素都是上界类型或者上界类型的子类型，这意味着无论被添加的元素是哪种类型，都无法确保安全的类型转换（比如集合里可能存的是Dog类型或者Animal类型，而被添加的类型如果是Cat类型就会发生类转型异常），所以会拒绝添加null以外的任意元素。而null是一个特殊的值，它可以转型为任意类型，因此能成功添加到上界通配符的泛型集合中。

但是这种集合可以往外读取元素，因为这些元素可以被JVM自动转型为上界类型。这种只允许读取，不允许写入的特性，被称为`Producer Extends`，相当于生产者的概念。

对于下界通配符的泛型集合，只能加入下界类型及其子类型，因为泛型擦除导致JVM只知道该集合中存放的都是下界类型或者下界类型的父类型，而下界类型及其子类型必然可以安全转型为下界类型，所以可以添加到该集合中。

但是这种集合只允许往外读取Object类型的元素，因为无法确定集合中的元素的具体类型，出于类型安全就只能作为Object类型被读取。如果将读取的元素进行强制类型转换，就要注意是否会发生类转换异常。这种只允许写入，不允许读取为原本类型的特性，被称为`Consumer Super`，相当于消费者的概念。

总结：

● 如果要从集合中读取类型`T`的数据，并且不能写入，可以使用`? extends T`通配符；(Producer Extends)<br>
● 如果要从集合中写入类型`T`的数据，并且不需要读取，可以使用`? super T`通配符；(Consumer Super)<br>
● 如果既要存又要取，那么就不要使用任何通配符。

在实际应用中，一般不直接对通配符泛型集合进行编辑操作，而是作为一个引用类型（可以是局部变量，或者是作为方法参数），并将另一个泛型集合对象赋值给该引用。

### 泛型的多重限定

泛型允许使用`&`进行多重限定，即`<T extends A & B>`，此时泛型的具体类型必须是这两个限定类型的最小范围或者共同子类型。并且，`&`的右值必须是接口，左值则没有这个要求。

此外，通配符`<?>`不可以使用多重限定。

```java
public <T extends List & Collection> void test1(T t) {  // ok
    
}

// 编译错误，Animal不是接口
public <T extends Dog & Animal> void test2(T t) {  // The type Animal is not an interface; it cannot be specified as a bounded parameter
    
}

// 编译错误
public <? extends List & Collection> void test3(T t) {  // Syntax error on token "?", Identifier expected
    
}
```

经过泛型擦除后，在字节码文件中多重限定会被擦除为`&`的左值类型，但在编译期时类型检查依然限定具体类型必须是这两个限定类型的最小范围或者共同子类型。

### `<T>`和`<?>`的区别

● `<T>`可以用于声明泛型，也可用于使用泛型的场合；`<?>`不可用于声明泛型，只能用于使用泛型的场合。<br>
● `<T>`可以指定上界，但此时只能用于声明泛型，`<T>`不可以指定下界；`<?>`可指定上下界，且只能用于使用泛型的场合。<br>
● `<T>`用于确保泛型参数的一致性，比如一个方法的参数是多个泛型`T`，那么调用方法传参时都必须是相同的类型；但如果一个方法的参数是多个泛型通配符`?`，则调用时传参不需要保持相同的类型，因为`?`表示随机类型。<br>
● `<T>`可以使用多重限定，而`<?>`不可以。<br>
● `<T>`无法创建参数化类型的数组，但`<?>`可以。

## 泛型数组

跟前面的泛型集合特性一样，只是把泛型集合塞到一个数组中而已。但不同的是，无法实例化一个参数化类型的数组对象，但可以实例化一个通配符类型的数组对象。

```java
public void test() {
    List<String>[] array1 = new ArrayList[10];  // ok
    array1[0].add("test");  // ok
    
    // 编译错误
    List<String>[] array2 = new ArrayList<String>[10];  // Cannot create a generic array of ArrayList<String>
    
    List<?>[] array3 = new ArrayList<?>[10];  // ok
    array3[0] = new ArrayList<String>();  // ok
    // 编译错误
    array3[0].add("test");  // The method add(capture#2-of ?) in the type List<capture#2-of ?> is not applicable for the arguments (String)
    array3[0].add(null);  // ok
}
```

## 不能使用泛型的场景

### 基本类型不能使用泛型

泛型的类型参数要求是Object的子类，所以不能使用基本数据类型，只能使用对应的包装类。

```java
List<int> list = new ArrayList<>();  // Syntax error, insert "Dimensions" to complete ReferenceType

List<Integer> list = new ArrayList<>();  // ok
```

### 泛型类型无法直接实例化

由于泛型擦除的原因，运行期泛型信息是不可见的，因此不能直接实例化。

```java
// 运行期不存在这个泛型E，所以无法实例化
public <E> void test(E e) {
    E e2 = new E();  // Cannot instantiate the type E
}
```

### 泛型类的泛型参数不能作为静态变量，也不能作为静态方法的返回值

泛型类在类实例化时才明确类型，而静态类型是在类加载时就初始化的，此时对于泛型类是无法明确泛型的具体类型的，所以泛型类的泛型参数不能作为静态变量。也就是说，泛型类的泛型参数默认就是非静态的。

但是，对于泛型方法，则可以被定义为静态的。原因是泛型方法在方法调用时明确类型，与类实例化无关，所以允许定义为静态的。

```java
class TestGeneric<T> {

    T t1;  // ok
    
    static T t2;  // Cannot make a static reference to the non-static type T
    
    static {
        T t3;  // Cannot make a static reference to the non-static type T
    }

    public static T getT(T t) {  // Cannot make a static reference to the non-static type T
        return t;
    }

    public static <E> E getE(E e) {  // ok
        return e;
    }
    
}
```

### 无法进行 instanceof 判断

Java的泛型是伪泛型，在编译期会被擦除，运行的字节码中不存在泛型，所以下面的判断条件无法进行：

```java
class TestGeneric<T> {

    public void test(ArrayList<T> list) {
        // Cannot perform instanceof check against parameterized type ArrayList<Integer>. 
        // Use the form ArrayList<?> instead since further generic type information will be erased at runtime
        if (list instanceof ArrayList<Integer>) {
            
        }

		// 编译报错
        if (list instanceof ArrayList<? super Animal>) {
            
        }

		// 编译报错
        if (list instanceof ArrayList<? extends Animal>) {
            
        }
    }

}
```

但是泛型的无界通配符`<?>`可以进行`instanceof`判断，**且只有无界通配符才可以用这个关键字判断，如果是有界通配符依然会编译报错**。

因为无界通配符类型的集合对象意味着可以是任意类型的集合，对于JVM来说都是同一个集合类型,无需具体区分是哪一种泛型集合，所以允许无界通配符使用`instanceof`进行判断。

### 无法创建参数化类型的数组

```java
public void test() {
    // 编译错误
    List<String>[] array = new ArrayList<String>[10];  // Cannot create a generic array of ArrayList<String>
}
```

因为在运行期泛型集合对于JVM来说都是同一种类型，这意味着对于一个泛型集合数组对象来说，无论放入的是`ArrayList<String>`还是`ArrayList<Integer>`都是一样的。这样就会和本身泛型集合数组的定义相矛盾，比如原本声明的是一个`List<String>[]`。为了避免发生这种逻辑错误，所以不允许创建参数化类型的数组。

但是可以创建通配符类型的数据，因为通配符类型是一个随机的类型，不会发生上述的自相矛盾：

```java
public void test() {
    List<?>[] array = new ArrayList<?>[10];  // ok
}
```

### 不能直接或者间接扩展Throwable

```java
// 不能间接地扩展 Throwable   
class IndirectException<T> extends Exception {}  // The generic class IndirectException<T> may not subclass java.lang.Throwable

// 不能直接地扩展 Throwable  
class DirectException<T> extends Throwable {}  // The generic class DirectException<T> may not subclass java.lang.Throwable
```

因为异常类在进行捕获时需要明确类型，在运行期JVM无法获取泛型异常的类型：
```java
try {
    
} catch (IndirectException<T> | DirectException<T> e) {
    // 类型不确定，无法处理具体的异常逻辑
}
```

但是可以抛出一个不确定的异常类型：

```java
class TestGeneric<T extends Exception> {

	// ok
    public void test1() throws T {
        
    }
    
	// ok
    public <E extends Exception> void test() throws E {
        
    }
}
```

### 泛型擦除后相同参数签名的方法不能重载

由于泛型擦除的原因，以下方法不能重载且会编译报错

```java
public void test(List<Integer> list) {}
public void test(List<Long> list) {}
```

## 一道笔试题

如何使用泛型实现LRU缓存？

LRU就是`Least Recently Used`的缩写，即最近最少使用。JAVA提供的`LinkedHashMap`可以拿来实现LRU缓存的功能，除了可以设定排序的模式（按照访问排序还是按照插入排序），还可以重写删除最旧键值对的方法。

● [一个简单的lru缓存实现](/all/container_05_源码分析-Map?id=一个简单的lru缓存实现)

## 参考链接

* [九、泛型](http://www.cyc2018.xyz/Java/Java%20%E5%9F%BA%E7%A1%80.html#九、泛型)
* [Java泛型中的PECS原则](https://blog.csdn.net/xx326664162/article/details/52175283)
* [万字长文详解Java泛型](https://imlql.cn/post/adb2faf0.html#post-comment)
* [什么情况下不能使用 Java 泛型](https://juejin.cn/post/6844904150514286599)
* [什么叫泛型？有什么作用？](https://www.zhihu.com/question/272185241)
* [使用泛型实现（LRU）缓存](https://blog.csdn.net/wang_muhuo/article/details/80390655)