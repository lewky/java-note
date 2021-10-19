# python继承

## 继承与多态

```python
class Animal(object):
    def run(self):
        print('Animal is running...')

class Dog(Animal):
    def run(self):
        print('Dog is running...')

class Cat(Animal):
    def run(self):
        print('Cat is running...')

animal = Animal()
animal.run()
# 当子类和父类都存在相同的run()时，子类run()会覆盖父类run()
animal = Dog()
animal.run()

# isinstance 判断一个对象是否是该类型本身，或者位于该类型的父继承链上
# 能用type()判断的基础类型也可以用isinstance判断
# 还可以判断一个变量是否为某些类型中的一种
print(isinstance(animal, Animal))

def call_run(animal):
    animal.run()

# 多态
# 静态语言，传入的对象必须是Animal类型或者子类，否则无法调用run()
# 动态语言，传入的对象只要拥有run()方法就可以调用
call_run(Dog())

# 多重继承
class Mammal(Animal):
    pass

class Runnable(object):
    pass

# 通过多重继承，一个子类就能同时获取多个父类的所有功能
# 这种让Dog继承自Mammal，再同时继承Runnable的设计通常称为Mixln
# Mixln的目的就是给一个类添加多个功能，在设计类时，我们优先考虑通过多重继承来组合多个Mixln的功能，而不是设计多层次的继承关系
class Dog2(Mammal, Runnable):
    pass
```

## `__str__`与`__repr__`

```python
class Student(object):
    def __init__(self, name):
        self.name = name
    def __str__(self):
        return 'Student object (name:%s) ' % self.name
    # 令__repr__也返回友好的字符串
    __repr__=__str__

# print()调用的是__str__，用于返回用户看到的字符串
# 直接显示变量调用的是__repr__，用于返回程序开发者看到的字符串
print(Student('fjh'))
```

## `__iter__`

一个类想被用于`for...in`，那么它必须实现`__iter__`，该方法返回一个迭代对象，那么`for`循环就会不断调用迭代对象的`__next__`方法拿到下一个值，直到遇到`StopIteration`错误时才退出循环

## `__getitem__`

按照下标取出元素

## `__setitem__`

把对象视作为`list`或`dict`来对集合赋值

## `__delitem__`

用于删除某个函数

## `__getattr__`

动态返回一个属性

```python
class Student(object):
    def __init__(self):
        self.name = 'fjh'
    def __getattr__(self, attr):
        if attr == 'score':
            return 100

stu = Student()
# 但只有在没有找到属性的情况下，才会调用__getattr__
# 利用__getattr__，实际上可以把一个类的所有属性和方法调用全部动态处理
print(stu.score)
```

## `__call__`

直接对实例进行调用

```python
class Student(object):
    def __init__(self, name):
        self.name = name
    def __call__(self):
        print('My name is %s.' % self.name)

s = Student('fjh')
s()
# 通过callable()函数，我们就可以判断一个对象是否"可调用"对象
print(callable(s))
```

## 枚举

枚举类型定义一个`class`类型，然后每一个常量都是`class`中的唯一实例

```python
from enum import Enum,unique

Month = Enum('month', ('Jan', 'Feb', 'Mar', 'Apr', 'May',
             'Jun', 'Jul', 'Aug', 'Sep', 'Oct', 'Nov', 'Dec'))

for name, member in Month.__members__.items():
    # value属性是自动赋值的，默认从1开始赋值
    print(name, '=>', member, ',', member.value)

# unique修饰器能帮我们检查没有重复值
@unique
class Weekday(Enum):
    # Sun的value被设定为0
    sun = 0
    Mon = 1
    Tue = 2
    Wed = 3
    Thu = 4
    Fri = 5
    Sat = 6

for name, member in Weekday.__members__.items():
    # value属性是自动赋值的，默认从1开始赋值
    print(name, '=>', member, ',', member.value)
```

## 使用元类

动态语言和静态语言最大的不同，就是函数和类的定义，不是编译时定义的，而是运行时动态创建的

当`Python`解释器载入`hello`模块时，就会依次执行该模块的所有语句

```python
class Student(object):
    pass

s = Student()

# Student是一个class，它的类型就是type
print(type(Student))
# s是一个实例，它的类型就是class Student
print(type(s))
```

`class`的定义是运行时动态创建的，而创建`class`的方法就是使用`type()`函数

`type()`函数即可以返回一个对象的类型，又可以创建出新的类型，比如，我们可以通过`type()`函数创建出`Hello`类，而无需通过`class Hello(object)`的定义

```python
def fn(self, name='world'):
    print('hello,%s.' % name)

# 创建 Hello class
# type()函数依次传入3个参数
# 1. class的名称
# 2. 继承的父类集合，注意Python支持多重继承，如果只有一个父类，别忘了tuple的单元素写法
# 3. class的方法名称与函数绑定，这里我们把函数fn绑定到方法名hello上
Hello = type('Hello', (object,), dict(hello=fn))

h = Hello()
h.hello()

print(type(Hello))
print(type(h))
```

通过`type()`函数创建的类和直接写`class`是完全一样的，因为`Python`解释器遇到`class`定义时，仅仅是扫描一下`class`定义的语法，然后调用`type()`函数创建出`class`

正常情况下，我们都用`class`来定义类，但是，`type()`函数也允许我们动态创建出类来，也就是说，动态语言本身支持运行期动态创建类，这和静态语言有非常大的不同，要在静态语言运行期创建类，必须构造源代码字符串再调用编译器，或者借助一些工具生成字节码实现，本质上都是动态编译，会非常复杂

### `metaclass`

先定义`metaclass`，再创建类，最后创建实例

```python
# 按照默认习惯，metaclass的类名总是以Metaclass结尾，以便清楚地表示这是一个metaclass
# metaclass是类的模板，所以必须从type类型派生
class ListMetaclass(type):
    # 利用__new__()可以修改类的定义，比如，加上新的方法，然后，返回修改后的定义
    # __new__()方法接收到的参数依次是
    # 1. 当前准备创建的类的对象
    # 2. 类的名字
    # 3. 类继承的父类集合
    # 4. 类的方法集合
    def __new__(cls, name, bases, attrs):
        attrs['add'] = lambda self, value: self.append(value)
        return type.__new__(cls, name, bases, attrs)

# 有了ListMetaclass，我们在定义类的时候还要指示使用ListMetaclass来定制类，传入关键字参数metaclass
# 当我们传入关键字参数metaclass时，魔术就生效了，它指示python解释器在创建MyList时，要通过ListMetaclass.__new__()来创建
class MyList(list, metaclass=ListMetaclass):
    pass

list=[]
# list.add('f')
print(list)
list=MyList()
list.add('f')
print(list)
```

### 自定义`orm`框架

```python
# 定义Field类，它负责保存数据库表的字段名和字段类型
class Field(object):
    def __init__(self, name, column_type):
        self.name = name
        self.column_type = column_type
    def __str__(self):
        return '<%s:%s>' % (self.__class__.__name__, self.name)

# 在Field的基础上，进一步定义各种类型的Field，比如StringField, IntegerField等等
class StringField(Field):
    def __init__(self, name):
        super(StringField, self).__init__(name, 'varchar(100)')

class IntegerField(Field):
    def __init__(self, name):
        super(IntegerField, self).__init__(name, 'bigint')

# 编写ModelMetaclass
# 在ModelMetaclass中，一共做了几件事情
# 排除掉对Model类的修改
# 在当前类中查找定义的类的所有属性，如果找到一个Field属性，就把它保存到一个__mappings__的dict中，同时从类属性中删除该Field属性，否则，容易造成运行时错误(实例的属性会遮住类的同名属性)
# 把表明保存到__table__中，这里简化为表名默认为类名
# 在Model类中，就可以定义各种操作数据库的方法，比如save()，delete()，find()，update()等等
# 我们实现了save()方法，把一个实例保存到数据库中，因为有表名，属性到字段的映射和属性值的集合，就可以构造出insert语句
class ModelMetaclass(type):
    def __new__(cls, name, bases, attrs):
        if name == 'Model':
            return type.__new__(cls, name, bases, attrs)
        print('Found model:%s' % name)
        mappings = dict()
        for k, v in attrs.items():
            if isinstance(v, Field):
                print('Found mapping: %s ==> %s' % (k, v))
                mappings[k] = v
        for k in mappings.keys():
            attrs.pop(k)
        # 保存属性和列的映射关系
        attrs['__mappings__'] = mappings
        # 假设表名和类名一致
        attrs['__table__'] = name
        return type.__new__(cls, name, bases, attrs)

# 基类Model
class Model(dict, metaclass=ModelMetaclass):
    def __init__(self, **kw):
        super(Model, self).__init__(**kw)
    def __getattr__(self, key):
        try:
            return self[key]
        except KeyError:
            raise AttributeError(r"'Model' object has no attribute '%s'" % key)
    def __setattr__(self, key, value):
        self[key] = value
    def save(self):
        fields = []
        params = []
        args = []
        for k, v in self.__mappings__.items():
            fields.append(v.name)
            params.append("?")
            args.append(getattr(self, k, None))
        sql = 'insert into %s (%s) values (%s)' % (self.__table__, ','.join(fields), ','.join(params))
        print('SQL: %s' % sql)
        print('ARGS: %s' % str(args))

# 定义user类
# 当用户定义一个class User(Model)时，Python解释器首先在当前类User的定义中查找metaclass，如果没有找到，就继续在父类Model中查找metaclass，找到了，就使用Model中定义的metaclass的ModelMetaclass来创建User类，也就是说，metaclass可以隐式地继承到子类，但子类自己却感觉不到
class User(Model):
    # 定义类的属性到列的映射
    id = IntegerField('id')
    name = StringField('username')
    email = StringField('email')
    password = StringField('password')

# 创建一个实例
u = User(id=12345, name='fjh', email='123@qq.com', password='123')
# 保存到数据库
u.save()
```
