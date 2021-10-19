# python 函数

函数名是指向一个函数对象的引用，能将函数对象的引用赋值给其他变量

## 定义函数

```python
# 用def语句定义，依次写出函数名，括号，参数和冒号，返回值用return语句
def my_fun(param):
    # 利用isinstance()做参数类型检查
    print(isinstance(param, (int,str)))
    return param

# 空函数
# pass语句相当于占位符，让空实现的函数不会报错
def null_fun():
    pass

# 支持多个返回值
def return_more():
    # 实际上会返回一个元组(1,2)
    return 1,2
# 而多个变量可以同时接收一个tuple，按位置赋值
a,b = return_more();
```

## 函数的参数形式

### 必选参数(位置参数)

```python
def fun1(a,b):
    return a + b

print(fun1(1,2))
```

### 默认参数

默认参数必须指向不变的对象

默认参数也是一个变量，它指向默认的对象，如果默认的对象可变并已被改变，下次调用时的默认参数指向的对象也会是被修改后的对象

```python
def fun2(x,y=2,z=3):
    return x + y + z

# 有多个默认参数时，调用的时候，即可以按照顺序提供参数，也可以把参数名带上
print(fun2(2,z=4))
```

### 可变参数(tuple)

```python
def fun3(*a):
    # 在函数内部，实际上是把传递进来的参数封装进一个tuple中，再赋值给参数a
    print(a)

fun3([1,2,3],(1,2,3)) # ([1, 2, 3], (1, 2, 3))
# 已有list或tuple，加上*能把list和tuple中的元素变成位置参数传递进去
fun3(*[1,2,3],*(1,2,3)) # (1, 2, 3, 1, 2, 3)
```

### 关键字参数(dict)

```python
def fun4(**a):
    print(a)

# 关键字参数允许传递0个或任意个含参数名的参数，这些参数会在函数内部自动封装进一个dict中
fun4()
fun4(a=1,b=2,c=3)
# 已有一个dict，加上**能把dict中的元素变成关键字参数传递进去
fun4(**{'f':1,'j':2},**{'h':3})
```

### 命名关键字参数

```python
# 目的是限制传入关键字参数的名字，命名关键字参数需要一个特殊的分隔符*，*后面的参数被视为命名关键字参数
def fun5(a,b,*,c,d):
    print(a,b,c,d)
# 如果函数定义中已经有一个可变参数，那么后面的命名关键字参数就不再需要*分隔符
def fun5_1(a,*b,c,d):
    print(a,b,c,d)
# 命名关键字参数必须传入参数名，并且也是必填
fun5(1,2,c=3,d=4)
fun5_1(1,2,c=3,d=4)
```

### 函数参数定义的顺序

1. 必选参数(位置参数)(必填)
2. 默认参数(可不填)
3. 可变参数(可不填)
4. 命名关键字参数(必填)
5. 关键字参数(可不填)

## 高阶函数

一个函数接收另一个函数作为参数，这种函数就称为高阶函数

```python
# 定义高阶函数
def add(x,y,f):
    return f(x) + f(y)

print(add(1.1,-1.2,abs))
```

常用高阶函数

* `map()`
* `reduce()`
* `filter()`
* `sorted()`

## 返回函数

高阶函数除了可以接收函数作为参数外，还可以把函数作为结果值返回

```python
def lazy_sum(*args):
    def sum():
        t = 0
        for i in args:
            t = t + i
        return t
    return sum

# 只是返回求和的过程，每次调用都是返回不同的对象
f = lazy_sum(1,2,3)
# 执行求和计算
print(f())
```

## 闭包

当函数A返回一个函数B后，函数A内部的局部变量还能被函数B引用

```python
def count():
    fs = []
    for i in range(1,4):
        def f():
            return i*i
        fs.append(f)
    return fs

f1,f2,f3=count()
print(f1(),f2(),f3())
```

```java
public class Test {

    interface F {
        Integer fun();
    }

    public static List<F> count() {
        List<F> fs = new ArrayList<>();
        final int final_i = 3;
        for (int i = 1; i < 4; i++) {
            // 闭包，这里的final_i指向的对象只是外面的变量指向的对象的拷贝，如果这个拷贝不为final，对这个拷贝进行修改就会导致拷贝和外面变量指向的对象不一致
            fs.add(() -> final_i * final_i);
        }
        return fs;
    }

    public static void main(String[] args) {
        List<F> list = count();
        for (F f : list) {
            // 调用函数
            System.out.println(f.fun());
        }
    }
}
```

## 匿名函数

在传入函数时，不需要显示定义函数，直接传入匿名函数更方便

`lambda`表示匿名函数，冒号前为函数参数

匿名函数只能有一个表达式，返回值就是表达式的结果

匿名函数也是一个函数对象，即也能赋值给变量

```python
def f1(x):
    return x*x

f2 = lambda x:x*x

print(f1(2))
print(f2(2))
```

## 偏函数

```python
import functools

# int2为二进制字符串转变成整数的函数
# functools.partial的作用就是把一个函数的某些参数固定住(设置默认参数)，再返回这个带默认参数的函数
int2 = functools.partial(int, base=2)

# 创建偏函数，实际上可以接收函数对象、*args和**kw这三个参数
# int2('1001')相当于kw={'base':2},int('1001',**kw)
print(int2('1001'))

max2 = functools.partial(max, 10)

# max2(5,6,7)相当于args=(10,5,6,7),max(*args)
print(max2(5, 6, 7))
```

## `Decorator`装饰器(python语法层次的装饰模式)

在代码运行期间动态增加功能的方式成为装饰器

`Decorator`实际上就是一个返回函数的高阶函数

```python
import functools

def log(func):
    def wrapper(*args, **kw):
        print('call %s():' % func.__name__)
        return func(*args, **kw)
    return wrapper

# 把@log放到now()函数的定义处，相当于执行了now=log(now)
@log
def now():
    print('2021-09-04')

# call now function
now()

# 如果Decorator本身需要传入参数，那写一个返回Decorator的高阶函数
def log2(text):
    def decorator(func):
        # 在定义的wrapper()的上面加上@functools.wraps(func)，这样就会把原函数的名字赋值到decorator返回的函数对象里
        @functools.wraps(func)
        def wrapper(*args, **kw):
            print('%s %s()' % (text, func.__name__))
            print(wrapper.__name__)
            return func(*args, **kw)
        return wrapper
    return decorator

# 相当于执行了now=log('execute')(now)
@log2('execute')
def now2():
    print('2021-09-04')

now2()
```
