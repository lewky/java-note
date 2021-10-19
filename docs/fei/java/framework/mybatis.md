# MyBatis

## 处理流程

1. `SqlSessionFactoryBuilder`从`xml`配置文件或一个`Configuration`对象实例中获取数据构建`SqlSessionFactory`对象实例
2. `SqlSessionFactory`对象实例构建`SqlSession`对象实例(`SqlSession`对象实例不是线程安全的)
3. 从`SqlSession`对象实例中获取`Mapper`对象实例
4. `Mapper`对象实例执行`SQL`

## `SQL`语句的定义

* `xml`文件定义
* `java`注解定义

## 二级缓存

需要手动启用二级缓存，默认只对一个`Session`中的数据进行缓存

## 映射原理

`MyBatis`在`Mapper`接口上使用了动态代理中一种非常规的用法，当调用代理时，动态代理会利用该接口的全限定类名和当前调用的方法名组成的字符串去匹配映射`SQL`定义中的`namespace`和具体的方法名组成的字符串，通过这种方式把`Mapper`接口跟`SQL`关联起来

> 这里的动态代理没有对某个具体类进行代理，而是通过动态代理转化成了对其他代码的调用
