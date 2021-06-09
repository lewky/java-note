<!--
date: 2021-03-27T22:34:12+08:00
lastmod: 2021-06-08T22:34:12+08:00
-->
## 8种基本数据类型（Primitive Data Types）

|基本数据类型|占用字节数|占用位数|
|-:|-:|-:|
|byte|1|8|
|char|2|16|
|short|2|16|
|int|4|32|
|long|8|64|
|float|4|32|
|double|8|64|
|boolean|无明确规定具体大小|无明确规定具体大小|

一个字节能表示的最大无符号十进制整数是255（`0~255`），两个字节能表示的最大无符号十进制整数是65535（`0~65535`）。

byte是一个八位带符号的二进制补码整数，取值范围为`-128~127`。有符号整数在计算机中都是以补码形式存储，因为计算机只有加法没有减法，为了以更低成本计算出结果，所以用补码存储有符号整数。

为了避免浮点型数据运算时出现误差，可以使用BigDecimal类进行浮点型数据的运算。

boolean只有两个值：true和false，可以用1bit来存储，但是Java无法直接操作位，只能通过`BitSet`来间接操作。

* [Java BitSet类](https://www.runoob.com/java/java-bitset-class.html)

BitSet通过`set()`来设置对应下标的值为`1`，而`and()`、`or()`、`xor()`等方法可以提供位运算。需要注意的是，这些位运算方法并不是返回一个新new出来的BitSet对象，而是直接改变原本BitSet对象的值。

关于位运算的规律：

● `a & b | b = b`<br>
● `(a | b) & b = b`<br>
● `a ^ a = 0`<br>

### char可以赋值为负数吗

char是无符号类型，如果赋值为负数，最终会转化为对应的正数。如下：

```java
final Character a = new Character((char) -1);
final Character b = new Character((char) 65535);
System.out.println(a.equals(b));    //true
System.out.println((char) -1 == (char) 65535);    //true
```

-1的补码是`1111 1111 1111 1111`，赋值给char后被当作无符号数处理，即65535。

### char能不能存贮一个中文汉字？

char类型可以存储一个中文汉字，因为Java中使用的编码是Unicode（不选择任何特定的编码，直接使用字符在字符集中的编号，这是统一的唯一方法），一个char类型占2个字节（16比特），所以放一个中文是没问题的。

>补充：使用Unicode意味着字符在JVM内部和外部有不同的表现形式，在JVM内部都是Unicode，当这个字符被从JVM内部转移到外部时（例如存入文件系统中），需要进行编码转换。所以Java中有字节流和字符流，以及在字符流和字节流之间进行转换的转换流，如InputStreamReader和OutputStreamReader，这两个类是字节流和字符流之间的适配器类，承担了编码转换的任务。

## 包装类(Wrapper Classes)

每种基本数据类型都有对应的包装类型，二者之间的转换可以通过自动装箱和自动拆箱完成(Autoboxing and Unboxing)。

所有的包装类都是final的，都实现了Serializable接口。

|基本数据类型|包装类|
|-:|-:|
|byte|Byte|
|char|Character|
|short|Short|
|int|Integer|
|long|Long|
|float|Float|
|double|Double|
|boolean|Boolean|

```java
Integer a = 1;     // 装箱，调用了 Integer.valueOf(1)
int b = x;         // 拆箱，调用了 a.intValue()
```

### 为什么有基本数据类型了，还需要包装类？

● 基本数据类型之间的转换可以通过包装类来实现。<br>
● Java是面向对象的语言，而基本数据类型不是对象类型。在一些场景中需要使用到对象类型，比如在集合中的元素就必须是对象类型。

## 缓存池

除了浮点数的包装类之外，其它包装类都有对应的缓存池或者缓存的设置。当基本数据类型在装箱时出于缓存池范围内，则会直接返回缓存中对应的包装类，而不是直接返回一个new出来的包装类。

这意味着缓存池范围内的基本数据类型，通过`valueOf()`获取的包装类对象一直都是同一个对象。

|包装类|缓存池|缓存池大小|
|-:|-:|-:|
|Byte|ByteCache|`-128~127`|
|Character|CharacterCache|`(char)0~(char)127`，即`\u0000 ~ \u007F`|
|Short|ShortCache|`-128~127`|
|Integer|IntegerCache|默认是`-128~127`，在jdk1.8种该最大值可以jvm启动参数`-XX:AutoBoxCacheMax=<size>`来指定|
|Long|LongCache|`-128~127`|
|Float|无|无|
|Double|无|无|
|Boolean|直接定义了两个静态常量Boolean：TRUE和FALSE|Boolean.TRUE和Boolean.FALSE|

## 类型转换

类型转换分隐式转换和显式转换，其实就是自动类型转换和强制类型转换。

## long、float、double变量的L、F、D尾缀

在声明long、float、double变量的时候，往往会在字面量末尾加上对应的L、F、D，也可以是小写的l、f、d，一般long变量尽量用大写的L，避免和数字的1和大写的i混淆。如下：

```java
long a = 123L;
float b = 1.23F;
double c = 1.23D;
```

等号右边的数值叫做字面量`literal`，跟在字面量末尾的字母是为了告诉编译器数值的类型。有时候不添加尾缀，编译期也不会报错，因为整数型的字面量默认是int，浮点型的字面量默认是double，如下：

```java
long a = 123;
double c = 1.23;
```

这里不会在编译期报错，是因为范围小的类型可以自动转换成范围大的类型，因为不会丢失精度，也叫向上转型。而`float b = 1.23;`会报错，是因为`1.23`默认是double类型，精度比float大，无法自动转换类型，需要进行强制类型转换，即`float b = (float) 1.23;`。

long的字节数是8，和float字节数一样，但是`long b = 1.23f;`还是会报错。这是因为long和float的存储方式不一样，尽管二者占据一样多的字节，但是float能表示的数值范围远超long，所以需要强制类型转换：`long b = (long) 1.23f;`。

**既然不能直接将字面量赋值给精度更小的类型，那为什么`byte b = 12; short s = 12;`却不需要强制类型转换也不会报错？**

这是因为编译器对整数型数值做了处理，对于整数字面量，如果赋值给比int范围更小的类型时（即byte/char/short类型），如果该字面量没有超出对应的赋值类型的范围，就会自动进行隐式类型转换。如果超出了范围，就会报错，需要经过强制类型转换才可以。

**为什么`byte b = 100`不会报错，而`int a = 100; byte b = a;`却会报错？**

前者是因为100属于byte的范围内，会隐式类型转换。后者的变量a已经被声明为int类型，将其直接赋值给byte变量需要经过强制类型转换：`int a = 100; byte b = (byte) a;`

## short s1 = 1; s1 = s1 + 1;有错吗?short s1 = 1; s1 += 1;有错吗？

对于`short s1 = 1; s1 = s1 + 1;`由于1是int类型，因此`s1+1`运算结果也是int 型，需要强制转换类型才能赋值给short型。而`short s1 = 1; s1 += 1;`可以正确编译，因为`s1+= 1;`相当于`s1 = (short)(s1 + 1);`其中有隐含的强制类型转换。

总结就是`+=`有隐式的强制类型转换。

## 参考链接

* [一、数据类型](http://www.cyc2018.xyz/Java/Java%20%E5%9F%BA%E7%A1%80.html#一、数据类型)
* [JAVA中为什么要有包装类，作用是什么](https://zhidao.baidu.com/question/2052192149152534987.html)
* [Java中给byte变量直接赋值可以自动转换,但为什么把int变量赋给byte变量需要强制转换，同样是int。](https://zhidao.baidu.com/question/878702194900509172.html)
* [Java中float、double、long类型变量赋值添加f、d、L尾缀问题](https://blog.csdn.net/FX677588/article/details/52663805)
