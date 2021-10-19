# python 对象

## 模块

在`python`中，一个`py`文件称为一个模块

为了避免模块名冲突，`python`引入了按目录来组织模块的方法，称为包`package`

每个包目录下都会存在一个`__init__.py`，如果不存在，则`python`就把这个目录当成普通目录而不是一个包，`__init__.py`可以是空文件，也可以有`python`代码，因为它本身就是一个模块，而它的模块名就是包的名字

任何模块代码的第一个字符串都被视为模块的文档注释

```python
# 导入sys模块后，sys这个变量就指向了sys模块
import sys

# 表明模块的作者
__author__ = 'fjh'
print(__author__)

# sys.argv用list存储了命令行的所有参数，第一个参数永远都是该py文件的名字
print(sys.argv)
print(len(sys.argv))

# 运行一个模块时，python解析器会把一个特殊变量__name__赋值为__main__，从其他地方导入该模块时，则不会赋值
print(__name__)
```

## 作用域

类似`__xx__`，这样的变量是特殊变量，能直接引用，但是会有特殊用途

类似`_x`，`__xx`，这样的变量是非公开的，不应该被直接引用

> 之所以说不应该被引用而不是说不能被引用，是因为`python`并没有方法可以完全限制访问私有的函数和变量

## 模块搜索路径

默认情况下，`python`解析器会搜索当前目录，所有已安装的内置模块和第三方模块

模块搜索路径存放在`sys`模块的`path`变量中

设置环境变量`PYTHONPATH`，该环境变量的内容会被自动添加到模块搜索路径中

```python
import sys

print(sys.path)
```

## 对象

```python
# 通过class关键字定义类，class后面跟着类目(大写开头)，接着(object)，表示从哪个类继承下来
# object类，是所有类最终都会继承的类
class Student(object):
    # __slots__ = ('name', 'score')

    # 利用__init__方法，在创建实例时，就把name，score等自定义的属性进行绑定
    # __init__的第一个参数永远都是self，表示创建的实例本身，所以在__init__方法内就可以利用self将各种属性绑定到创建的实例中
    # 有了__init__方法，就必须传入与__init__方法匹配的参数，但self不需要，python解析器会自己把实例变量传入
    def __init__(self, name, score):
        self.name = name
        self.score = score

    # 在类中定义方法与普通定义函数不一样，就是第一个参数永远是self，并且调用时不用传递该参数，除了这一点，其他没有区别
    def print_score(self):
        print('%s: %s' % (self.name, self.score))


# 变量fang指向的是一个Student的实例，每个实例的内存地址都不一样，则类是创建实例的模板，而实例是一个个具体的对象，各个实例拥有的数据都是互相独立的互不影响的
# 封装数据的函数是和Student类本身关联起来的，这种函数成为类的方法
fang = Student('fang', 18)

print(fang.name)
fang.print_score()

# 注释掉__slots__，可以自由地给一个实例变量绑定属性，这点和静态语言不一样，即同一个类的不同实例，可以拥有不一样的实例变量
fang.age = 100
print(fang.age)
```

## 类的访问控制

类中的属性，以`__`开头，就会变成一个私有变量，只能从内部访问

> 不能访问是因为`python`解析器对外把`__name`变量改成了`_Student__name`，所以还是可以通过`_Student__name`来访问`__name`变量，但是不同版本的`python`解析器可能会把`__name`改成不同的变量名

在`python`中`__xx__`这种变量是特殊变量，这种可以直接访问

## 获取对象的信息

```python
import types

# 判断对象类型，可以传入对象实例也可以传入指向对象的变量，type()返回的是对应的Class类型
print(type(123))
print(type(123)==int)

def fun():
    pass

print(type(fun)==types.FunctionType)

# 获取一个对象的所有属性和方法
print(dir('abc'))

# len()实际上也是去调用该对象中的__len__
print(len('abc'))
print('abc'.__len__())

class Student(object):

    def __init__(self, name):
        self.name = name

# 获取或设置对象的属性
person = Student('fjh')
print(hasattr(person, 'age'))
setattr(person, 'age', 18)
print(hasattr(person, 'age'))
```

## 实例属性和类属性

直接在`class`中定义属性，这种属性是类属性，归类所有，所有类的实例都可以访问到

尽量不要对实例属性和类属性使用相同的名字，否则将产生难于发现的错误

```python
class Student(object):
    name = 'student class name'


a = Student()
# 因为实例并没有name属性，所以会继续查找class的name属性
print(a.name)
# 打印类的name属性
print(Student.name)
# 给实例绑定name属性
a.name = 'a name'
# 由于实例属性优先级比类属性高，因此，它会屏蔽掉类的name属性
print(a.name)
# 但是类属性并未消失，用Student.name仍然可以访问
print(Student.name)
# 删除实例的name属性
del a.name
# 再次调用实例的name属性，由于实例的name属性没有找到，类的name属性就显示出来了
print(a.name)
```

## 动态绑定属性和方法

给一个实例绑定属性和方法，对另外一个实例是不起作用的

为了给所有实例绑定方法，可以给`class`绑定方法

能使用`__slots__`来限制该`class`实例能添加的属性

```python
class Student(object):

    # 能使用__slots__来限制该class实例能添加的属性
    # 用tuple定义允许绑定的属性名称
    # __slots__定义的属性仅对当前实例起作用，对继承的子类是不起作用的，除非子类也定义了__slots__，这样子类实例允许定义的属性就是自身的__slots__加上父类的__slots__
    __slots__ = ('name')


a = Student()
a.name = 'f'
print(a.name)
# AttributeError: 'Student' object has no attribute 'age'
# a.age=1
```

## 利用`@property`定义只读属性

```python
class DataSet(object):
    def __init__(self):
        self._score = 1

    @property
    def score(self):
        return self._score


a = DataSet()
# 可以修改_score变量，但是不建议
a._score = 2
print(a._score)
# AttributeError: can't set attribute
# a.score=3
print(a.score)
```
