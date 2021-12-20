<!--
date: 2021-12-08T22:34:12+08:00
lastmod: 2021-12-20T22:34:12+08:00
-->
## 原型模式（Prototype Pattern）

原型模式的目的是高效的创建重复的对象，换言之，就是高效的克隆一个对象。

关于Java的克隆可以看这个：[Java#clone](https://javanote.doc.lewky.cn/#/all/basic_04_继承?id=clone)

原型模式通常需要维护一个关于若干原型对象的集合作为缓存，便于从中取出客户端指定的对象并进行克隆。

## 样例代码

```java
public class Book implements Cloneable {

    int id;
    String name;

    // 必须显示重写clone方法才能调用
    @Override
    public Object clone() throws CloneNotSupportedException {
        return super.clone();
    }
}

public class PrototypeManager {

    public static ConcurrentHashMap<Integer, Book> cache = new ConcurrentHashMap<>();

    public Book getBook(final int id) {
        final Book book = cache.get(id);
        Book result = book;
        if (book != null) {
            try {
                result = (Book) book.clone();
            } catch (final CloneNotSupportedException e) {
                e.printStackTrace();
            }
        }

        return result;
    }

    public void putBook(final Book book) {
        cache.put(book.id, book);
    }
}
```

## 参考链接

* [原型模式](https://www.runoob.com/design-pattern/prototype-pattern.html)
* [《JAVA与模式》之原型模式](https://www.cnblogs.com/java-my-life/archive/2012/04/11/2439387.html)