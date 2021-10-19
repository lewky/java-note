# python基础

`python`是解析语言

`python`是动态类型语言

* 动态类型语言，变量本身类型不固定的语言
* 静态类型语言，在定义变量时，必须指定变量的类型，赋值的类型必须与变量的类型一致

## 编码

`python`默认使用`unicode`编码

`Bytes`类型的数据用`b`前缀的单引号和双引号表示，`Bytes`类型的每个字符只占用一个字节

以`unicode`表示的字符串能用`encode()`变成指定编码的字节`Bytes`

```python
'ABC'.encode('ascii') # b'ABC'
'中'.encode('utf-8') # b'\xe4\xb8\xad'
```

将网络或磁盘上的字节流转换成字符串

```python
b'\xe4\xb8\xad'.decode('utf-8',errors='ignore') # '中'
```

尽量用`utf-8`保持源代码，为了让`python`解析器能用`utf-8`来读取源码，在源码文件顶部加上

```python
#!/usr/bin/python3
# -*- coding: utf-8 -*-
```

## 运行代码

需要`python`解析器去执行`py`文件

`Cpython`是官方的解析器，用C语言编写，把`python`代码解析成机器语言，让机器能识别

## 注释

单行 #

## 语法

当以冒号结尾时，缩进的代码会被视为代码块

程序大小写敏感

变量名字必须是大小写英文，数字和`_`的组合，不能以数字开头

## 数据类型

* 整数

  十六进制，`0x`开头

  `python`允许在数字中间用`_`隔开

  大小没有限制

* 浮点数

  浮点数运算会有误差

  大小没有限制，但是超出范围就用`inf`表示无限大

* 字符串

  以`'`和`"`括起来的任意文本

  `r'some text'`，表示''内的文本不进行转义

  `'''some text'''`，表示多行内容

  以`\x`为前缀，表示接下来的两个字符被解释为字符代码的十六进制数字

* 布尔值

  `True`，`False`，大小写敏感

  布尔运算符：`and`，`or`，`not`

* 空值

  `None`

* 列表

  `list`，一种有序列表

  `-1`能作为最后一个元素的索引

  `classmates = ['fjh','xiaomi']`

* 元组

  `tuple`，一种有序列表，但是初始化后不能修改，但如果元组中的元素是`list`这样的结构，`list`里的元素也能改变

  `classmates = ('fjh','xiaomi')`

  定义只有一个元素的元组时，`T=(1)`，直接赋值为整数1，`T=(1,)`，赋值为只有一个元素为整数1的元组

  `python`在显示只有一个元素的元组时，也会带上逗号

* 字典

  `dict`，键值对，无序，充当键的对象必须是不可变的，例如字符串类型、整数类型

  `namemap = {'f':0,'j':1,'h':2}`

  获取`f`键的值，`namemap['f']`

  把数据放进字典，除了在初始化时放入，还可以通过`key`放入，`namemap['f'] = 3`

  获取不存在的`key`，会报错，避免报错的方法

  ```python
  if 'a' in namemap:
      print(namemap['a'])
  else:
      print(namemap.get('a','default value'))
  ```

* 无序的列表

  set，创建`set`，需要提供`list`，重复的元素会被过滤

  `dict`跟`set`的区别在于`set`没有`value`，只有`key`，则作为`key`的对象也必须是不可变的

  ```python
  set = {1,2,3,3}
  print(set) # {1, 2, 3}
  ```

## 常量

在`python`中通常用大写的变量表示常量，这个是习惯用法

`python`中没有机制保证变量中的值不会被修改

## 算术运算符

`/`，结果是浮点数

`//`，结果只取整数

## 格式化字符串

* 用`%`实现

  `print('hello,%%%s%s'%('world','!'))`，用`%%`转义`%`

* `format()`，占位符，`{0}{1}...`，下标必须从0开始

  `print('f{0}h,{1}'.format('j','yes'))`

* `f''`，占位符，`{xxx}`

  ```python
  name = 'jiehuifang'
  thing = 'yes'
  print(f'{name},{thing}')
  ```

## 流程控制

### 条件控制

```python
count = 2
if count == 1:
    print("one")
else:
    print("not one")
```

### 循环

* `for`语句

  ```python
  list = [1,2,3,4]
  for i in list:
      print(i)
  ```

* `while`语句

  ```python
  list = [1,2,3,4]
  n = len(list) - 1
  while(n >= 0):
      print(list[n])
      n = n - 1
  ```

* `break`语句，退出循环
* `continue`语句，退出本次循环

## 切片

```python
list = [1,2,3,4,5]
# 获取前三个元素，从索引0开始取，到索引3为止，不包括索引3
print(list[0:3]) # [1, 2, 3]
print(list[:3]) # [1, 2, 3]
# -1为倒数第一个元素的下标
print(list[-2:-1]) # [4]
# 前5个，每2个取一个
print(list[:5:2])
# 原样复制一个list
print(list[:])
```

> 字符串，元组也同样适用切片

## 迭代

可直接用于`for`循环的对象统称为可迭代对象

1. 集合数据类型，`list`，`tuple`，`dict`，`str`，`set`
2. 生成器，包括列表生成式和带`yield`的函数

判断对象是否为可迭代对象

```python
from collections.abc import Iterable
print(isinstance(1, Iterable))
print(isinstance('1', Iterable))
```

```python
dict = {'f': 1, 'j': 2, 'h': 3}
for k in dict:
    print(k)
for v in dict.values():
    print(v)
for k, v in dict.items():
    print('{0}:{1}'.format(k,v))
```

通过内置的`enumerate`函数把一个`list`变成索引-元素对

```python
list = [1,2,3]
for i,v in enumerate(list):
    print('{0}:{1}'.format(i,v))
```

同时引用两个变量

```python
list = [(1,1),(2,2)]
for x,y in list:
    print('{0}:{1}'.format(x,y))
```

### 迭代器

可以被`next()`函数调用并不断返回下一个值的对象称为迭代器`Iterator`，它表示一个惰性计算的序列

生成器是`Iterator`，但是`list`，`dict`，`str`等不是`Iterator`，可以用`iter()`让它们变成`Iterator`

```python
from typing import Iterator

print(isinstance([], Iterator))
print(isinstance(iter([]), Iterator))
```

## 列表生成式

```python
# 生成1*1 2*2 ... 5*5的列表
print([x*x for x in range(1, 6)])
# 上面的列表，仅保留偶数的平方
print([x*x for x in range(1, 6) if x % 2 == 0])
# 多个for组合
print([m+n for m in 'ABC' for n in 'XYZ'])
# if语句位置的关系
print([x for x in range(1,6) if x % 2 == 0]) # 这个if表示筛选
print([x if x % 2 == 0 else -x for x in range(1,6)]) # 这个if else是表达式，它必须根据x计算出一个结果
```

## 生成器

不必在循环前创建完整的`list`，在循环时边循环边计算，以节省内存，这种机制称为生成器

```python
l = [x*x for x in range(10)] # 列表
g = (x*x for x in range(10)) # 生成器
```

> 列表与生成器的区别是用`[]`还是用`()`

通过`next(g)`，来获取`g`生成器的下一个值

生成器也是可迭代对象

用类似的列表生成式的`for`循环无法实现时，可以通过函数实现

```python
def fib(max):
    n, a, b = 0, 0, 1
    while n < max:
        yield b
        # t = (b, a + b)，t为元组tuple
        # a = t[0]
        # b = t[1]
        a, b = b, a+b
        n = n+1
    return 'done'

g = fib(5)

for x in g:
    print(x)
```

如果一个函数中存在`yield`，则这个函数就不是一个普通的函数，而是一个生成器，生成器执行流程不同于函数的顺序流程，而是每次调用`next()`函数，遇到`yield`就返回，再次执行时就从上次返回的`yield`语句的地方继续执行

想要获取生成器中`return`的值，必须获取`StopIteration`，返回值在它的`value`中

对于函数改成的生成器，遇到`return`和执行到最后一行语句，就是结束生成器的指令，`for`循环也会随之结束

调用函数改成的生成器，实际上是返回一个生成器对象
