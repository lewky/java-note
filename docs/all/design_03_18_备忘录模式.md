<!--
date: 2022-01-18T22:34:12+08:00
lastmod: 2022-01-18T22:34:12+08:00
-->
## 备忘录模式（Memento Pattern）

保存一个对象的某个状态，以便在适当的时候恢复对象。应用实例比如游戏的存档、数据库的事务操作等。

备忘录模式通常使用三个类：Memento包含需要被恢复对象的状态，Originator创建对象并在Memento存储对应的状态，Caretaker负责从Memento中恢复对象的状态。

## 样例代码

定义Memento类：

```java
public class Memento {
   private String state;
 
   public Memento(String state){
      this.state = state;
   }
 
   public String getState(){
      return state;
   }  
}
```

定义Originator类：

```java
public class Originator {
   private String state;
 
   public void setState(String state){
      this.state = state;
   }
 
   public String getState(){
      return state;
   }
 
   public Memento saveStateToMemento(){
      return new Memento(state);
   }
 
   public void getStateFromMemento(Memento Memento){
      state = Memento.getState();
   }
}
```

定义Caretaker类：

```java
public class CareTaker {
   private List<Memento> mementoList = new ArrayList<Memento>();
 
   public void add(Memento state){
      mementoList.add(state);
   }
 
   public Memento get(int index){
      return mementoList.get(index);
   }
}
```

定义测试类：

```java
public class MementoPatternDemo {
   public static void main(String[] args) {
      Originator originator = new Originator();
      CareTaker careTaker = new CareTaker();
      originator.setState("State #1");
      originator.setState("State #2");
      careTaker.add(originator.saveStateToMemento());
      originator.setState("State #3");
      careTaker.add(originator.saveStateToMemento());
      originator.setState("State #4");
 
      System.out.println("Current State: " + originator.getState());    
      originator.getStateFromMemento(careTaker.get(0));
      System.out.println("First saved State: " + originator.getState());
      originator.getStateFromMemento(careTaker.get(1));
      System.out.println("Second saved State: " + originator.getState());
   }
}
```

## 参考链接

* [备忘录模式](https://www.runoob.com/design-pattern/memento-pattern.html)