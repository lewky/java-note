# Spring 基础

## IOC，控制反转

* `IOC`容器

  容器具有容器中所有对象生命周期的控制权，包括创建、初始化、销毁等

* 松耦合

  面向接口的编程，能按照需求切换具体的实现类

* 少侵入

  切换实现类的时候，需要修改源码，可以将实现类的完全限定类名保存在配置文件中，再利用反射动态生成实现类

* `ID`，依赖注入，自动装配

  将动态生成的实现类注入到相应的对象属性中

## Spring 声明 bean 的方式

* 基于`xml`声明`bean`(需要提供`setter`)

  ```xml
  <bean id="student" class="Student">
    <property name="name" value="fjh"/>
    <property name="teacher" ref="teacher"/>
  </bean>
  ```

* 基于`@Component`等注解声明`bean`(不需要提供`setter`，但是这些`bean`需要被`scan`)

  ```java
  @Component("teacher")
  public class Teacher {
      @Value("jiehuifang")
      private String name;
  }
  ```

* 基于`java`类声明(需要提供`setter`)

  ```java
  @Configuration
  public class BeansConfiguration {
      @Bean
      public Student student() {
          Student student = new Student();
          student.setName("fjh");
          student.setTeacher(teacher());
          return student;
      }
  }
  ```

## Spring 自动装配

* 基于`xml`

  ```xml
  <bean id="student" class="Student" autowire="byName">
    <property name="name" value="fjh"/>
  </bean>

  <bean id="teacher" class="Teacher">
    <property name="name" value="jiehuifang"/>
  </bean>
  ```

  类型

  * `no`
  * `byName`
  * `byType`
  * `constructor`
  * `autodetect`

* 基于注解

  * `@Resource`，默认使用`byName`
  * `@Autowired`，默认使用`byType`

## Spring AOP

实现代理原理

* `JDK`动态代理，基于继承接口
* `CGLIB`动态代理，基于继承类

## SpringBoot自动装配原理(autoConfigure)

`@SpringBootApplication`->`@EnableAutoConfiguration`->`@Import(AutoConfigurationImportSelectors.class)`

1. `AutoConfigurationImportSelectors`类的`selectImports()`，会读取`META-INF/spring.factories`文件，并获取文件中所有配置类的全限定类型名
2. 所有配置类会根据`@ConditionalOnXXX`来决定是否加载到`IOC`容器中
3. `@EnableConfigurationProperties(XXXProperties.class)`来读取`application.properties`中的自定义配置

## Spring MVC的处理流程

关键组件

* 前端处理器`DispatcherServlet`

  接收请求，响应结果，相当于转发器

* 处理器映射器`HandlerMapping`

  根据请求的`URL`寻找处理器和处理器拦截器

* 处理器适配器`HandlerAdapter`

  按照`HandlerAdapter`的要求去执行`Handler`

* 后端处理器`Handler`

  接收请求进行业务的操作

* 视图解析器`ViewResolver`

  进行视图的解析

* 视图`View`

  前端页面

处理流程

1. 前端处理器`DispatcherServlet`接受用户的请求
2. `DispatcherServlet`调用`HandlerMapping`
3. `HandlerMapping`返回相应的`HandlerExecutionChain(Handler,HandlerInterceptor)`给`DispatcherServlet`
4. `DispatcherServlet`调用相应的`HandlerAdapter`
5. `HandlerAdapter`经过适配调用具体的`Handler`
6. `Handler`执行完业务操作返回`ModelAndView`给`handlerAdapter`
7. `handlerAdapter`将`ModelAndView`返回给`DispatcherServlet`
8. `DispatcherServlet`将`ModelAndView`传给`ViewResolver`
9. `ViewResolver`解析后返回具体的`View`给`DispatcherServlet`
10. `DispatcherServlet`渲染`View`
11. `DispatcherServlet`响应用户

### `controller`接收参数的方式

* `form`表单参数提交

  ```java
  // 不添加注解时，参数名要和表单参数名一致
  // 添加注解@RequestParam时，参数名和表单参数名不要求一致
  public void formPost(String userName, @RequestParam(value = "password", required = true, defaultValue = "123") String password) {}
  ```

* `form`表单`bean`实体提交

  ```java
  // bean的属性要和表单里的名称一致
  // 可以不添加注解
  // 可以添加注解@ModelAttribute，但是不能添加@RequestBody
  public void formPost(@ModelAttribute UserInfo userInfo, String testParam) {}
  ```

* `form`表单提交，相同参数名可以用数组接收

  ```java
  public void formPost(String[] userName, String testParam) {}
  ```

* 获取`url`后面的参数

  ```java
  @RequestMapping(value = "/link/get/{name}")
  public void LinkGetPath(@PathVariable("name") String userName) {}
  ```

* 获取`url`的查询参数

  ```java
  // url='link/get/params?name=a&pwd=b'
  // value必须和查询参数名一致
  public void LinkGetParams(@RequestParam("name") String name, @RequestParam("pwd") String password) {}
  // 或者参数名要和查询参数名一致
  public void LinkGet(String name, String pwd) {}
  ```

* 获取`url`中的查询参数

  ```java
  // url='link/get/params?name=a&pwd=b'
  // 不需要注解
  public void LinkGet(UserInfo userInfo) {}
  ```

* 获取`post`中的`json`字符串或`json`

  ```java
  public String ajaxPost(@RequestBody UserInfo userInfo) {}
  ```

* 使用`HttpServletRequest`接收

  ```java
  public String request(HttpServletRequest req, HttpServletResponse resp) {}
  ```
