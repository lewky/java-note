<!--
date: 2022-01-18T22:34:12+08:00
lastmod: 2022-01-18T22:34:12+08:00
-->
## 中介者模式（Mediator Pattern）

通过提供一个中介类来封装一系列对象之间的交互，降低对象之间的耦合性，将一对多转为了一对一的关系。

比如一群用户之间的聊天交互（群聊天），可以通过提供一个聊天室中介者来接收并显示所有用户的消息。

## 样例代码

定义一个聊天室：

```java
public class ChatRoom {
   public static void showMessage(User user, String message){
      System.out.println(new Date().toString()
         + " [" + user.getName() +"] : " + message);
   }
}
```

定义用户类：

```java
public class User {
   private String name;
 
   public String getName() {
      return name;
   }
 
   public void setName(String name) {
      this.name = name;
   }
 
   public User(String name){
      this.name  = name;
   }
 
   public void sendMessage(String message){
      ChatRoom.showMessage(this,message);
   }
}
```

定义一个测试类：

```java
public class MediatorPatternDemo {
   public static void main(String[] args) {
      User robert = new User("Robert");
      User john = new User("John");
 
      robert.sendMessage("Hi! John!");
      john.sendMessage("Hello! Robert!");
   }
}
```

## 参考链接

* [中介者模式](https://www.runoob.com/design-pattern/mediator-pattern.html)