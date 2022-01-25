<!--
date: 2022-01-18T22:34:12+08:00
lastmod: 2022-01-18T22:34:12+08:00
-->
## 观察者模式（Observer Pattern）

提供一个观察者类，当一个对象的状态发生改变时，所有依赖它的对象都由观察者类去通知并得到更新。在一对多的对象关系中，观察者模式可以降低耦合，保证协作。

观察者模式发布-订阅模式（Publish/Subscribe），被观察者是拥有数据源的上游，观察者在发现数据变化时要通知依赖于该数据源的下游。

被观察者可以自由注册或删除观察者，在状态改变时通知所有观察者。注意观察者和被观察者之间不能有循环依赖，否则可能导致系统崩溃。通常使用异步方式通知观察者。

## 样例代码

定义一个被观察者：

```java
public class Subject {
   
   private List<Observer> observers 
      = new ArrayList<Observer>();
   private int state;
 
   public int getState() {
      return state;
   }
 
   public void setState(int state) {
      this.state = state;
      // 状态改变时通知所有观察者
      notifyAllObservers();
   }
 
   public void attach(Observer observer){
      observers.add(observer);      
   }
 
   public void notifyAllObservers(){
      for (Observer observer : observers) {
         observer.update();
      }
   }  
}
```

定义观察者抽象类和子类：

```java
public abstract class Observer {
   protected Subject subject;
   public abstract void update();
}

public class BinaryObserver extends Observer{
 
   public BinaryObserver(Subject subject){
      this.subject = subject;
      this.subject.attach(this);
   }
 
   @Override
   public void update() {
      System.out.println( "Binary String: " 
      + Integer.toBinaryString( subject.getState() ) ); 
   }
}

public class OctalObserver extends Observer{
 
   public OctalObserver(Subject subject){
      this.subject = subject;
      this.subject.attach(this);
   }
 
   @Override
   public void update() {
     System.out.println( "Octal String: " 
     + Integer.toOctalString( subject.getState() ) ); 
   }
}

public class HexaObserver extends Observer{
 
   public HexaObserver(Subject subject){
      this.subject = subject;
      this.subject.attach(this);
   }
 
   @Override
   public void update() {
      System.out.println( "Hex String: " 
      + Integer.toHexString( subject.getState() ).toUpperCase() ); 
   }
}
```

定义一个测试类：

```java
public class ObserverPatternDemo {
   public static void main(String[] args) {
      Subject subject = new Subject();
 
      new HexaObserver(subject);
      new OctalObserver(subject);
      new BinaryObserver(subject);
 
      System.out.println("First state change: 15");   
      subject.setState(15);
      System.out.println("Second state change: 10");  
      subject.setState(10);
   }
}
```

## 参考链接

* [观察者模式](https://www.runoob.com/design-pattern/observer-pattern.html)