# servlet

## servlet的生命周期

1. 浏览器连上`web server`
2. 浏览器发送`http`请求
3. `web server`解析出请求要访问的主机名、`web`应用、`web`资源
4. 第一次访问`servlet`，`servlet`对象实例化(`servlet`对象仅创建一次)
5. 调用`servlet`对象的`init()`完成对象初始化
6. 创建`request`对象和`response`对象，调用`servlet`对象的`service()`处理浏览器的请求
7. `web server`从`response`对象中取出数据，构建一个`http`响应，返回给浏览器
8. 浏览器解析`http`响应，提取数据进行显示

## `web.xml`

`<load-on-startup>`标签决定`servlet`在web容器中的启动优先级，数字越小，启动优先级越高

`<servlet-mapping>`标签决定`servlet`的映射路径，`/`为默认`servlet`的映射路径，无法匹配的请求都由默认`servlet`处理

`<init-param>`标签用于配置`servlet`的初始化参数

`<context-param>`标签用于配置`web`应用的参数

## ServletConfig

`web`容器在创建`servlet`实例对象时，会自动把配置的初始化参数封装到`ServletConfig`对象中，并在调用`servlet`的`init()`时，将`ServletConfig`对象传递给`servlet`对象

## ServletContext

`web`容器在启动时，会为每个`web`应用程序都创建一个对应的`ServletContext`对象，它代表当前`web`应用程序，`web`容器关闭或删除该`web`应用时，会销毁相应的`ServletContext`对象

> `ServletConfig.getServletContext()` // 获取`ServletContext`对象

一个`web`应用中的所有`servlet`对象共享同一个`ServletContext`对象

## HttpServletRequest

```java
// 获取request中的同名参数
String[] values = request.getParameterValues("username");

// 获取request中的数据，再封装成对象
Map<String, String[]> map = request.getParameterMap();
User user = new User();
BeanUtils.populate(user, map);

// 解决request中文乱码
// post
request.setCharacterEncoding("utf-8");
// get
String username = request.getParameter("username");
username = new String(username.getBytes("iso8859-1"), "utf-8");
```

### 请求转发`forward`

```java
// 使用request把数据带给转发资源
request.setAttribute("data", data);
request.getRequestDispatcher("/other").forward(request, response);
```

把`response`的输出流`close`之后，就会对浏览器作出响应，响应之后不能将请求转发，否则会抛异常，每次响应后应该`return`

`forward`时，会清空`response`中的数据

请求转发的特点

* 客户端只发一次请求，服务端会发一次或多次请求
* 客户端请求地址不会发生变化

`web`工程中各类地址的写法

* 地址给服务端使用时，`/`表示当前`web`应用
* 地址给客户端使用时，`/`表示当前网站，网站下有多个`web`应用

### referer防盗链

```java
String referer = request.getHeader("referer");
if(referer == null || referer.startsWith("http://localhost")) {
    response.sendRedirect("/index");
    return;
}
```

### 会话

用户打开一个浏览器，点击多个超链接，访问服务器多个`web`资源，然后关闭浏览器，整个过程称之为一个会话

### Cookie

`Cookie`是客户端技术，把用户的数据以`Cookie`的形式返回给浏览器，并存储到本地硬盘中，当用户使用浏览器再次访问`web`资源时，就会把`Cookie`带上，服务端就会从`Cookie`中获取数据

### Session

`Session`是服务端技术，服务端为每个用户的浏览器创建一个独享的`Session`对象，当用户使用浏览器再次访问`web`资源时，服务端就会从对应的`Session`中获取数据

### Cookie与Session的联系

`Session`是基于`Cookie`的，`Session`的`id`以`Cookie`的形式返回给浏览器，当用户使用浏览器再次访问`web`资源时，就会带上`Cookie`形式的`SessionID`，服务端会根据`SessionID`来获取对应`Session`数据

### url重写

当浏览器禁用`Cookie`时，那就在超链接中加上`SessionID`来实现保存用户数据的功能

`response.encodeURL()`，用于禁用`Cookie`时，在地址加上`SessionID`

## HttpServletResponse

### 响应的编码

```java
// 通知浏览器用什么编码读取数据
response.setHeader("Content-type", "text/html;charset=UTF-8");
// 设置字符流的编码
response.setCharacterEncoding("UTF-8"); 
// 如果是字节流，要用相应的编码写入数据
response.getOutputStream().write(data.getBytes("UTF-8"));
```

### 请求重定向`redirect`

```java
response.setStaus(302);
response.setHeader("location", "/index.jsp");
// 上面两个语句的简化版
response.sendRedirect("/index.jsp");
```

请求重定向的特点

* 客户端发两次请求
* 客户端请求地址会发生变化

### 文件下载

```java
response.setHeader("content-disposition", "attachment;filename=" + URLEncoder.encode(filename, "UTF-8"));
response.getOutputStream().write(data);
```

## 读取配置文件

```java
InputStream in = this.getServletContext().getResourceAsStream("/WEB-INF/class/db.properties");
Properties properties = new Properties();
properties.load(in); // 底层通过map来存储数据
String url = properties.getProperty("url");

// 获取资源文件的绝对路径
String path = this.getServletContext().getRealPath("/WEB-INF/class/db.properties");

// 通过类加载器读取文件
// 类加载器在web容器启动时只加载一次，所以读取不到修改过的文件
Demo.class.getClassLoader().getResourceAsStream("db.properties");
// 可以通过获取资源文件的地址，来读取最新的文件
String path = Demo.class.getClassLoader().getResource("db.properties").getPath();
```

## 过滤器`filter`

`filter`用于对用户请求进行预处理和对响应进行后处理，是典型的处理链

`filter`不能有直接的响应

### 处理流程

1. `filter`在`web`应用加载时进行初始化，`filter`的声明顺序决定了初始化的顺序
2. `filter`对用户请求进行预处理，`filter`的声明顺序决定了`filter`的预处理顺序
3. 将预处理后的请求交给`servlet`进行处理并生成响应
4. `filter`对响应进行后处理，`filter`的后处理顺序跟预处理的顺序相反
5. `filter`在`web`应用卸载时进行销毁，`filter`的声明顺序决定了销毁的顺序
