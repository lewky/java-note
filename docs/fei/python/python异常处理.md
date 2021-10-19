# python异常处理

## `try except finally`的错误处理机制

```python
try:
    print('try...')
    r=10/int('2')
    print('result:',r)
# python的错误其实也是class，所有的错误类型都继承自BaseException，所以在使用except时需要注意的是，它不但捕获该类型的错误，还把其子类也捕获了
except ValueError as e:
    print('ValueError:',e)
except ZeroDivisionError as e:
    print('ZeroDivisionError:',e)
# 如果没有错误发生，可以在except语句块后面加一个else，当没有错误发生时，会自动执行else语句
else:
    print('no error!')
finally:
    print('finally...')
print('end')
```

## 调用栈

如果错误没有被捕获，它就会一直往上抛，最后被python解释器捕获，打印一个错误信息，然后程序退出

## 记录错误

```python
# 内置的logging模块
# 通过配置，logging还可以把错误记录到日志文件里，方便事后排查
import logging

def foo(s):
    return 10/int(s)

def bar(s):
    return foo(s) * 2

def main():
    try:
        bar('0')
    except Exception as e:
        logging.exception('recording:',e)

main()
print('end')
```

## 抛出异常

因为错误是`class`，捕获一个错误就是捕获到该`class`的一个实例

如果要抛出错误，首先根据需要，可以定义一个错误的`class`，选择好继承关系，然后，用`raise`语句抛出一个错误的实例

```python
class FooError(ValueError):
    pass
def foo(s):
    n=int(s)
    if n==0:
        raise FooError('invalid value:%s' % s)
    return 10/n

foo('0')
```

只有在必要的时候才定义我们自己的错误类型，如果可以选择`python`已有的内置的错误类型(`ValueError`，`TypeError`)，尽量使用`Python`内置的错误类型

```python
def foo(s):
    n=int(s)
    if n==0:
        raise ValueError('invalid value:%s' % s)
    return 10/n

def bar():
    try:
        foo('0')
    except ValueError as e:
        print('ValueError!')
        raise

bar()
```

由于当前函数不知道应该怎么处理该错误，所以，最恰当的方式是继续往上抛，让顶层调用者去处理

`raise`语句如果不带参数，就会把当前错误原样抛出，此外，在`except`中`raise`一个`error`，还可以把一种类型的错误转化成另一种类型

```python
try:
    10/0
except ZeroDivisionError:
    raise ValueError('input error')
```

## 调试

### 调试方式

1. 利用`print()`
2. 利用断言`assert()`

    ```python
    def foo(s):
    n=int(s)
    # 期望n不为0，否则抛出AssertionError: n is zero!
    assert n!=0, 'n is zero'
    return 10/n

    foo('0')
    ```

    程序中如果到处充斥着`assert`和`print()`相比也好不到哪去，不过，启动`Python`解释器时可以用`-O`参数来关闭`assert`

3. 利用`logging`

## 单元测试

```python
import unittest

# 编写单元测试时，我们需要编写一个测试类，从unittest.TestCase继承
class MyTest(unittest.TestCase):
    # 以test开头的方法就是测试方法，不以test开头的方法不被认为是测试方法，测试的时候不会被执行
    # 对每一类测试都需要编写一个test_xxx()方法，由于unittest.TestCase提供了很多内置的条件判断，我们只需要调用这些方法就可以断言输出是否我们所期望的
    def test_1(self):
        # 最常用的断言就是assertEqual()
        # 断言函数返回的结果与1相等
        print('function test_1() run...')
        self.assertEqual(abs(-1), 1)

    # 在每调用一个测试方法前被执行
    def setUp(self):
        print('unit test start')

    # 在每调用一个测试方法后被执行
    def tearDown(self):
        print('unit test end')


# 运行单元测试
# 在命令行通过参数`-m unittest直接运行单元测试
if __name__ == '__main__':
    unittest.main()
```
