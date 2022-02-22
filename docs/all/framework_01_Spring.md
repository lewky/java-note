<!--
date: 2022-02-22T22:34:12+08:00
lastmod: 2022-02-22T22:34:12+08:00
-->
## Bean定义5种作用域

singleton（单例），prototype（原型），request，session，global session

## Bean的生命周期

1）实例化Bean：Ioc容器通过获取BeanDefinition对象中的信息进行实例化，实例化对象被包装在BeanWrapper对象中<br>
2）设置对象属性（DI）：通过BeanWrapper提供的设置属性的接口完成属性依赖注入；<br>
3）注入Aware接口（BeanFactoryAware， 可以用这个方式来获取其它 Bean，ApplicationContextAware）：Spring会检测该对象是否实现了xxxAware接口，并将相关的xxxAware实例注入给bean<br>
4）BeanPostProcessor：自定义的处理（分前置处理和后置处理）<br>
5）InitializingBean和init-method：执行我们自己定义的初始化方法<br>
6）使用<br>
7）destroy：bean的销毁

## IOC和DI

IOC：控制反转，将对象的创建权交由Spring管理。

DI：依赖注入，在Spring创建对象的过程中，把对象依赖的属性注入到对象中。

## Spring的IOC注入方式

构造器注入，setter方法注入，注解注入，接口注入

## 怎么检测是否存在循环依赖

Bean在创建的时候可以给该Bean打标，如果递归调用回来发现正在创建中的话，即说明存在循环依赖。

## Spring如解决Bean循环依赖问题

Spring中循环依赖场景有：

1）构造器的循环依赖<br>
2）属性的循环依赖

Spring通过三级缓存来解决属性的循环的依赖问题：

1）singletonObjects：第一级缓存，里面存放的是实例化好的单例对象<br>
2）earlySingletonObjects：第二级缓存，里面存放的是提前曝光的单例对象，即尚未彻底完成属性依赖注入的单例对象<br>
3）singletonFactories：第三级缓存，里面存放的是要被实例化的对象的对象工厂

在创建Bean时，Spring首先去一级缓存获取，如果获取不到，并且对象正在创建中，就去二级缓存获取。如果还是获取不到，就去三级缓存获取：Bean通过构造函数实例化后，此时对象属性还未填充就通过三级缓存提前对外曝光，获取到Bean后将其移动到二级缓存，等完全初始化之后则移到到一级缓存。

**由于加入singletonFactories三级缓存的前提是执行了构造器，因此构造器的循环依赖无法解决。**

构造器循环依赖解决办法：在构造函数中使用@Lazy注解延迟加载。在注入依赖时，先注入代理对象，当首次使用时再创建对象完成注入。

三级缓存中的Bean是互斥的关系，而非层次递进的关系，实际上就是三个Map里存放了不同生命周期的Bean，最终只会将Bean存放到一级缓存中。

## Spring中使用了哪些设计模式

工厂模式：spring中的BeanFactory就是简单工厂模式的体现，根据传入唯一的标识来获得bean对象；

单例模式：提供了全局的访问点BeanFactory；

代理模式：AOP功能的原理就使用代理模式（1、JDK动态代理。2、CGLib字节码生成技术代理。）

装饰器模式：依赖注入就需要使用BeanWrapper；

观察者模式：spring中Observer模式常用的地方是listener的实现。如ApplicationListener。

策略模式：Bean的实例化的时候决定采用何种方式初始化bean实例（反射或者CGLIB动态字节码生成）

## AOP核心概念

1、切面（aspect）：类是对物体特征的抽象，切面就是对横切关注点的抽象

2、横切关注点：对哪些方法进行拦截，拦截后怎么处理，这些关注点称之为横切关注点。

3、连接点（joinpoint）：被拦截到的点，因为 Spring 只支持方法类型的连接点，所以在Spring 中连接点指的就是被拦截到的方法，实际上连接点还可以是字段或者构造器。

4、切入点（pointcut）：对连接点进行拦截的定义

5、通知（advice）：所谓通知指的就是指拦截到连接点之后要执行的代码，通知分为前置、后置、异常、最终、环绕通知五类。

6、目标对象：代理的目标对象

7、织入（weave）：将切面应用到目标对象并导致代理对象创建的过程

8、引入（introduction）：在不修改代码的前提下，引入可以在运行期为类动态地添加方法或字段。

## AOP思想

传统OOP面向对象开发思想是自上而下的，这个过程会产生一些横切性问题，这些问题与主业务逻辑关系不大，且会散落在代码的各个地方，造成难以维护。AOP面向切面编码的思想，就是把业务逻辑和横切的问题进行解耦，提高代码的复用率和开发效率。

## AOP的主要应用场景

记录日志，监控性能，权限控制，事务管理

## AOP使用哪种动态代理

当Bean对象存在接口，或者是Proxy的子类时，则使用JDK动态代理。如果不存在接口，则采用CGLIB字节码生成技术来生成代理对象。

JDK动态代理主要涉及到`java.lang.reflect`包中的两个类：Proxy和InvocationHandler。通过`Proxy.newProxyInstance(target)`生成代理对象，代理对象通过反射invoke方法实现调用真实对象的方法。

## 动态代理与静态代理区别

静态代理，程序运行前代理类的.class文件就存在了。

动态代理：在程序运行时利用反射动态创建代理对象。（复用性，易用性，更加集中都调用invoke）

## CGLIB与JDK动态代理区别

JDK动态代理必须提供接口才能使用。

CGLIB不需要，只要一个非抽象类就能实现动态代理。

## 参考链接

* [敖丙在位置上肝了一个月的后端知识点长啥样？](https://mp.weixin.qq.com/s/gBr3UfC1HRcw4U-ZMmtRaQ)