<!--
date: 2022-01-17T22:34:12+08:00
lastmod: 2022-01-18T22:34:12+08:00
-->
## 迭代器模式（Iterator Pattern）

迭代器模式提供一种顺序访问容器中元素的方法，而无需关注容器的底层存储结构。比如Java容器中就通过迭代器模式来遍历不同的容器：[容器中的设计模式](https://javanote.doc.lewky.cn/#/all/container_01_数据结构概览?id=%e5%ae%b9%e5%99%a8%e4%b8%ad%e7%9a%84%e8%ae%be%e8%ae%a1%e6%a8%a1%e5%bc%8f)。

## 样例代码

定义一个迭代器接口和实现类：

```java
// 迭代器对外提供的统一接口
interface Iterator {
    boolean hasNext();

    Object next();
}

// 底层存储结构为数组的迭代器
class ArrayIterator implements Iterator {

    String[] stringArray;
    int index = 0;

    public ArrayIterator(String[] stringArray) {
        this.stringArray = stringArray;
    }

    @Override
    public boolean hasNext() {
        return index < stringArray.length && stringArray[index] != null;
    }

    @Override
    public Object next() {
        return stringArray[index++];
    }

    public static Iterator iterator(String[] stringArray) {
        return new ArrayIterator(stringArray);
    }
}
```

定义一个测试类：

```java
public class Test {
    public static void main(String[] args) {
        String[] array = {"a", "b", "c"};
        Iterator iterator = ArrayIterator.iterator(array);
        while (iterator.hasNext()) {
            System.out.println(iterator.next());
        }
    }
}
```

## 参考链接

* [迭代器模式](https://www.runoob.com/design-pattern/iterator-pattern.html)