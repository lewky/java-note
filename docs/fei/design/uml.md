# 统一建模语言`UML`

## 类图

### 类的种类

* `Entity Class`实体类

    实体类对应系统需求中的每个实体

* `Control Class`控制类

    控制类用于体现应用程序的执行逻辑

* `Boundary Class`边界类

    边界类用于对外部用户与系统之间的交互对象进行抽象

### 类的`UML`图示

在`UML`中，类使用类名，属性和操作且带有分隔线的长方形来表示

抽象类的类名以及抽象方法的名字都用斜体字表示

1. 类名

    例，`Demo`

    每个类都必须有一个名字，类名是一个字符串

    接口要在顶端加上`<<interface>>`

2. 属性

    例，`+ name:String fjh`

    一个类可以有任意多个属性，也可以没有属性

    1. 可见性，`+`表示`public`，`-`表示`private`，`#`表示`protected`
    2. 属性名:类型
    3. 默认值(可选项)

3. 操作

    例，`+ method1(name:String) :void`

    1. 可见性，`+`表示`public`，`-`表示`private`，`#`表示`protected`
    2. 方法名(参数列表)，参数定义与属性的定义相似，参数个数是任意的，多个参数之间用逗号隔开
    3. 返回类型(可选项)

### 类与类之间关系的`UML`图示

1. `association`关联关系，(实线、箭头)

    用于表示一类对象与另一类对象之间有联系

    通常将一个类的对象作为另一个类的成员变量

    1. 双向关联
    2. 单向关联
    3. 自关联
    4. 多重性关联

2. `dependent`依赖关系，(虚线、箭头)

    用于表示一类对象的改变会影响到使用该类对象的其他类对象

    1. 将一个类的对象作为另一个类中方法的参数，再调用其方法
    2. 将一个类的对象作为另一个类的局部变量，再调用其方法
    3. 在一个类中调用另一个类的静态方法

3. `aggregtion`聚合关系，(空心菱形、实线、箭头)

    用于表示整体与部分的关系，且部分对象可以脱离整体对象独立存在

    成员对象(部分对象)通常作为构造方法、`Setter`方法或者业务方法的参数注入到整体对象中

4. `composition`组合关系，(实心菱形、实线、箭头)

    用于表示整体与部分的关系，且整体对象可以控制部分对象的生命周期，部分对象不能独立存在

    成员对象(部分对象)通常在整体对象的构造方法中直接实例化

5. `generalization`泛化关系，(实线、空心三角形)

    用于表示父类与子类的继承关系

6. `realization`实现关系，(虚线、空心三角形)

    用于表示类实现了接口的关系
