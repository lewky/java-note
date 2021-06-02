<!--
date: 2021-05-19T22:34:12+08:00
lastmod: 2021-06-02T22:34:12+08:00
-->
## 前言

本文的源码分析主要基于JDK 1.8。

## Enumeration

在JDK 1.0时使用的迭代器是`Enumeration`，被同样推出于JDK 1.0的`Hashtable`和`Vector`所使用：`java.util.Hashtable#elements()`、`java.util.Hashtable#keys()`、`java.util.Vector#elements()`。

但是该接口名字太长，方法也只有两个：`hasMoreElements()`、`nextElement()`，方法名同样很长。于是在之后被`Iterator`接口所取代，不仅将上述两个方法替换为名字更简洁的新方法，还新增了remove方法。

Enumeration的JavaDoc解释如下：

```java
 * NOTE: The functionality of this interface is duplicated by the Iterator
 * interface.  In addition, Iterator adds an optional remove operation, and
 * has shorter method names.  New implementations should consider using
 * Iterator in preference to Enumeration.
```

## Iterator

JDK 1.2新增了Iterator接口，取代了以前的Enumeration接口，提供了以下几个方法：

```java
// 是否存在下一个元素
boolean hasNext();

// 获取下一个元素
E next();

// 移除上次遍历的元素，默认实现会抛出异常
// 只有在每次执行完next()后才能执行一次remove()
default void remove() {
    throw new UnsupportedOperationException("remove");
}

// JDK 1.8添加的方法
// 遍历时集合中所有的元素，并执行Consumer接口
default void forEachRemaining(Consumer<? super E> action) {
    Objects.requireNonNull(action);
    while (hasNext())
        action.accept(next());
}
```

不同的容器由于内部的数据结构不同，遍历的方式也有所区别。**通过迭代器模式，不同的容器在各自的内部类中实现了各种迭代器，以实现各自的遍历需求。**

在使用迭代器遍历容器时，如果需要移除元素，则必须通过Iterator来移除元素，如果直接使用容器移除元素，会抛出`ConcurrentModificationException`。因为在迭代器在通过`next()`来获取下一个元素时会检测当前集合是否被修改过（直接比较`modCount`和`expectedModCount`的值），如果被修改过就会抛出并发修改异常，这种机制被称为快速失败机制（fail—fast）。

这是出于线程安全来考虑的，通常使用的容器并非线程安全的，在遍历时直接用容器的remove方法可能有线程安全问题。而迭代器则不同，因为容器获取迭代器时得到的总是一个新建的迭代器，这意味着使用迭代器遍历时通常是在局部作用域中，不存在线程安全问题。因此使用迭代器移除元素时，会直接将`modCount`和`expectedModCount`的值设为一样，这样就不会抛出并发修改异常。

### ArrayList的Iterator实现类

ArrayList通过一个私有内部类来实现Iterator接口：

```java
private class Itr implements Iterator<E> {
	// 当前遍历的游标
    int cursor;       // index of next element to return
	// 上一个遍历的元素下标
    int lastRet = -1; // index of last element returned; -1 if no such
	// 期望的修改次数，每次修改集合的数据结构时都会增大1，用来检测是否发生了并发修改集合操作
    int expectedModCount = modCount;

    public boolean hasNext() {
        return cursor != size;
    }

    @SuppressWarnings("unchecked")
    public E next() {
        checkForComodification();
        int i = cursor;
        if (i >= size)
            throw new NoSuchElementException();
        Object[] elementData = ArrayList.this.elementData;
        if (i >= elementData.length)
            throw new ConcurrentModificationException();
        cursor = i + 1;
        return (E) elementData[lastRet = i];
    }

    public void remove() {
        if (lastRet < 0)
            throw new IllegalStateException();
        checkForComodification();

        try {
            ArrayList.this.remove(lastRet);
            cursor = lastRet;
            lastRet = -1;
            expectedModCount = modCount;
        } catch (IndexOutOfBoundsException ex) {
            throw new ConcurrentModificationException();
        }
    }

    @Override
    @SuppressWarnings("unchecked")
    public void forEachRemaining(Consumer<? super E> consumer) {
        Objects.requireNonNull(consumer);
        final int size = ArrayList.this.size;
        int i = cursor;
        if (i >= size) {
            return;
        }
        final Object[] elementData = ArrayList.this.elementData;
        if (i >= elementData.length) {
            throw new ConcurrentModificationException();
        }
        while (i != size && modCount == expectedModCount) {
            consumer.accept((E) elementData[i++]);
        }
        // update once at end of iteration to reduce heap write traffic
        cursor = i;
        lastRet = i - 1;
        checkForComodification();
    }

    final void checkForComodification() {
        if (modCount != expectedModCount)
            throw new ConcurrentModificationException();
    }
}
```

可以看到，`next()`方法最终就是直接用下标返回ArrayList底层的数组对象中的元素，但是比起直接用for
循环来遍历ArrayList，迭代器里需要记录遍历时的游标，还会检测是否发生了`ConcurrentModificationException`。因此用for循环遍历，效率会优于迭代器遍历。

下面是获取该迭代器的API：

```java
public Iterator<E> iterator() {
    return new Itr();
}
```

## ListIterator

ListIterator同样是JDK 1.2引入的迭代器接口，继承自Iterator接口并对其进行了扩展，允许向前遍历和set、add操作，还允许指定迭代器开始迭代时的下标索引。这个迭代器接口是专门针对List集合的。

```java
public interface ListIterator<E> extends Iterator<E> {
    // Query Operations
	// 下面的是查询操作

    boolean hasNext();

    E next();

	// 是否存在上一个元素
    boolean hasPrevious();

	// 获取上一个元素
    E previous();

	// 获取下一个要遍历的元素下标，如果迭代器已经在List末尾则返回List的size
    int nextIndex();

	// 获取上一个要遍历的元素下标，如果迭代器已经在List头部则返回-1
    int previousIndex();


    // Modification Operations
	// 下面的是修改操作

	// 移除上次遍历的元素，可以是被next()或者previous()遍历得到的元素
	// 只有在每次调用了next()或者previous()之后才能调用一次remove()，并且在调用remove()之前不能调用过remove()或add()
    void remove();

	// 替换上次遍历的元素，可以是被next()或者previous()遍历得到的元素
	// 并且在调用set()之前不能调用过add()
    void set(E e);

	// 添加新的元素，该元素会被立即插入到下次next()遍历的元素之前（如果存在该元素的话），或者立即插入到下次previous()遍历的元素之后（如果存在该元素的话）。
	// 该add操作会使nextIndex()或者previousIndex()的返回值增加1
    void add(E e);
}
```

### ArrayList的ListIterator实现类

ArrayList通过一个私有内部类来实现ListIterator接口：

```java
private class ListItr extends Itr implements ListIterator<E> {
	// 指定开始遍历时的游标
    ListItr(int index) {
        cursor = index;
    }

    public boolean hasPrevious() {
        return cursor != 0;
    }

    public E previous() {
        checkForComodification();
        try {
            int i = cursor - 1;
            E previous = get(i);
            lastRet = cursor = i;
            return previous;
        } catch (IndexOutOfBoundsException e) {
            checkForComodification();
            throw new NoSuchElementException();
        }
    }

    public int nextIndex() {
        return cursor;
    }

    public int previousIndex() {
        return cursor-1;
    }

    public void set(E e) {
        if (lastRet < 0)
            throw new IllegalStateException();
        checkForComodification();

        try {
            AbstractList.this.set(lastRet, e);
            expectedModCount = modCount;
        } catch (IndexOutOfBoundsException ex) {
            throw new ConcurrentModificationException();
        }
    }

    public void add(E e) {
        checkForComodification();

        try {
            int i = cursor;
            AbstractList.this.add(i, e);
            lastRet = -1;
            cursor = i + 1;
            expectedModCount = modCount;
        } catch (IndexOutOfBoundsException ex) {
            throw new ConcurrentModificationException();
        }
    }
}
```

下面是获取该迭代器的API：

```java
public ListIterator<E> listIterator(int index) {
    if (index < 0 || index > size)
        throw new IndexOutOfBoundsException("Index: "+index);
    return new ListItr(index);
}

public ListIterator<E> listIterator() {
    return new ListItr(0);
}
```

## Spliterator

JDK 1.8新增的迭代器接口，是一个可分割迭代器（splitable iterator），允许对一个容器进行分割、遍历，分割容器时采用的是二分法，分割成功时会返回一个新的Spliterator对象以遍历集合的一半元素，原本的Spliterator对象则遍历剩下的一半元素。该迭代器主要用于并行遍历元素，比如`parallelStream().forEach()`。

```java
public interface Spliterator<T> {
	// 尝试遍历下一个元素并执行对应的action，然后返回true
	// 如果没有下一个元素可以遍历则返回false
    boolean tryAdvance(Consumer<? super T> action);

	// 遍历所有的元素并执行对应的action
    default void forEachRemaining(Consumer<? super T> action) {
        do { } while (tryAdvance(action));
    }

	// 将当前迭代器进行分割，返回一个新的迭代器，原本的迭代器则变为一个新的用以迭代剩下元素的迭代器
	// 通过反复调用该方法来一直进行分割，直到最终无法继续分割时会返回null
    Spliterator<T> trySplit();

	// 返回迭代器剩下的尚未遍历的元素数量
    long estimateSize();

	// 迭代器特征值为SIZED类型时则会调用estimateSize()来获取需要遍历的元素个数
	// 对于其他类型的迭代器则返回-1
    default long getExactSizeIfKnown() {
        return (characteristics() & SIZED) == 0 ? -1L : estimateSize();
    }

	// 返回迭代器特征值：ORDERED, DISTINCT, SORTED, SIZED, NONNULL, IMMUTABLE, CONCURRENT, SUBSIZED
	// 无论是否进行了分割，迭代器的特征值都是固定不变的
    int characteristics();

	// 判断迭代器是否具有指定的特征值
    default boolean hasCharacteristics(int characteristics) {
        return (characteristics() & characteristics) == characteristics;
    }

	//如果Spliterator的list是通过Comparator排序的，则返回Comparator
	//如果Spliterator的list是自然排序的（即实现了Comparable接口） ，则返回null
	//其他情况下抛错
    default Comparator<? super T> getComparator() {
        throw new IllegalStateException();
    }

	// 专门针对原始类型设计的Spliterator
    public interface OfPrimitive<T, T_CONS, T_SPLITR extends Spliterator.OfPrimitive<T, T_CONS, T_SPLITR>>
            extends Spliterator<T> {

        T_SPLITR trySplit();

        boolean tryAdvance(T_CONS action);

        default void forEachRemaining(T_CONS action) {
            do { } while (tryAdvance(action));
        }
    }

	// A Spliterator specialized for int values.
	// 专门用于int值的Spliterator
    public interface OfInt extends OfPrimitive<Integer, IntConsumer, OfInt> {

        @Override
        OfInt trySplit();

        @Override
        boolean tryAdvance(IntConsumer action);

        @Override
        default void forEachRemaining(IntConsumer action) {
            do { } while (tryAdvance(action));
        }

        @Override
        default boolean tryAdvance(Consumer<? super Integer> action) {
            if (action instanceof IntConsumer) {
                return tryAdvance((IntConsumer) action);
            }
            else {
                if (Tripwire.ENABLED)
                    Tripwire.trip(getClass(),
                                  "{0} calling Spliterator.OfInt.tryAdvance((IntConsumer) action::accept)");
                return tryAdvance((IntConsumer) action::accept);
            }
        }

        @Override
        default void forEachRemaining(Consumer<? super Integer> action) {
            if (action instanceof IntConsumer) {
                forEachRemaining((IntConsumer) action);
            }
            else {
                if (Tripwire.ENABLED)
                    Tripwire.trip(getClass(),
                                  "{0} calling Spliterator.OfInt.forEachRemaining((IntConsumer) action::accept)");
                forEachRemaining((IntConsumer) action::accept);
            }
        }
    }

	// A Spliterator specialized for int values.
	// 专门用于long值的Spliterator
    public interface OfLong extends OfPrimitive<Long, LongConsumer, OfLong> {

        @Override
        OfLong trySplit();

        @Override
        boolean tryAdvance(LongConsumer action);

        @Override
        default void forEachRemaining(LongConsumer action) {
            do { } while (tryAdvance(action));
        }

        @Override
        default boolean tryAdvance(Consumer<? super Long> action) {
            if (action instanceof LongConsumer) {
                return tryAdvance((LongConsumer) action);
            }
            else {
                if (Tripwire.ENABLED)
                    Tripwire.trip(getClass(),
                                  "{0} calling Spliterator.OfLong.tryAdvance((LongConsumer) action::accept)");
                return tryAdvance((LongConsumer) action::accept);
            }
        }

        @Override
        default void forEachRemaining(Consumer<? super Long> action) {
            if (action instanceof LongConsumer) {
                forEachRemaining((LongConsumer) action);
            }
            else {
                if (Tripwire.ENABLED)
                    Tripwire.trip(getClass(),
                                  "{0} calling Spliterator.OfLong.forEachRemaining((LongConsumer) action::accept)");
                forEachRemaining((LongConsumer) action::accept);
            }
        }
    }

    // A Spliterator specialized for double values.
	// 专门用于double值的Spliterator
    public interface OfDouble extends OfPrimitive<Double, DoubleConsumer, OfDouble> {

        @Override
        OfDouble trySplit();

        @Override
        boolean tryAdvance(DoubleConsumer action);

        @Override
        default void forEachRemaining(DoubleConsumer action) {
            do { } while (tryAdvance(action));
        }

        @Override
        default boolean tryAdvance(Consumer<? super Double> action) {
            if (action instanceof DoubleConsumer) {
                return tryAdvance((DoubleConsumer) action);
            }
            else {
                if (Tripwire.ENABLED)
                    Tripwire.trip(getClass(),
                                  "{0} calling Spliterator.OfDouble.tryAdvance((DoubleConsumer) action::accept)");
                return tryAdvance((DoubleConsumer) action::accept);
            }
        }

        @Override
        default void forEachRemaining(Consumer<? super Double> action) {
            if (action instanceof DoubleConsumer) {
                forEachRemaining((DoubleConsumer) action);
            }
            else {
                if (Tripwire.ENABLED)
                    Tripwire.trip(getClass(),
                                  "{0} calling Spliterator.OfDouble.forEachRemaining((DoubleConsumer) action::accept)");
                forEachRemaining((DoubleConsumer) action::accept);
            }
        }
    }
}
```

### ArrayList的Spliterator实现类

ArrayList通过一个**静态私有内部类**来实现Spliterator，这是一个基于下标进行二分法分割的迭代器，且是一个懒加载的迭代器（静态内部类的特性，单例模式常用这种方式来实现）：

```java
/** Index-based split-by-two, lazily initialized Spliterator */
static final class ArrayListSpliterator<E> implements Spliterator<E> {
	// 被遍历的数组对象
    private final ArrayList<E> list;
	// 遍历的游标，只有在用遍历下一个元素、分割迭代器时才会被修改
    private int index; // current index, modified on advance/split
	// 迭代器需要遍历的元素数量（不一定等于list的size，因为迭代器是可分割的），第一次使用之前值为-1
    private int fence; // -1 until used; then one past last index
	// 检测list是否发生了并发修改
    private int expectedModCount; // initialized when fence set

    /** Create new spliterator covering the given  range */
    ArrayListSpliterator(ArrayList<E> list, int origin, int fence,
                         int expectedModCount) {
        this.list = list; // OK if null unless traversed
        this.index = origin;
        this.fence = fence;
        this.expectedModCount = expectedModCount;
    }

    private int getFence() { // initialize fence to size on first use
        int hi; // (a specialized variant appears in method forEach)
        ArrayList<E> lst;
        if ((hi = fence) < 0) {
            if ((lst = list) == null)
                hi = fence = 0;
            else {
                expectedModCount = lst.modCount;
                hi = fence = lst.size;
            }
        }
        return hi;
    }

	// 尝试使用二分法来分割迭代器，会将原本的迭代器按照list的下标平均分为一左一右两个迭代器
	// parallelStream的遍历会call这个方法来实现并行遍历，根据设定的并行线程数量来分割成对应数量的迭代器
	// hi其实就是high，lo是low，和mid一样各自代表着数组的下标
	// 迭代器的分割不需要开发者操心，并行流会自行处理
    public ArrayListSpliterator<E> trySplit() {
        int hi = getFence(), lo = index, mid = (lo + hi) >>> 1;
        return (lo >= mid) ? null : // divide range in half unless too small
            new ArrayListSpliterator<E>(list, lo, index = mid,
                                        expectedModCount);
    }

	// 尝试遍历下一个元素并执行对应的action，也就是parallelStream的forEach传递进来的Consumer参数
    public boolean tryAdvance(Consumer<? super E> action) {
        if (action == null)
            throw new NullPointerException();
        int hi = getFence(), i = index;
        if (i < hi) {
            index = i + 1;
            @SuppressWarnings("unchecked") E e = (E)list.elementData[i];
            action.accept(e);
            if (list.modCount != expectedModCount)
                throw new ConcurrentModificationException();
            return true;
        }
        return false;
    }

	// 遍历所有的元素并执行对应的action，在不需要分割迭代器时直接使用这个方法进行遍历
    public void forEachRemaining(Consumer<? super E> action) {
        int i, hi, mc; // hoist accesses and checks from loop
        ArrayList<E> lst; Object[] a;
        if (action == null)
            throw new NullPointerException();
        if ((lst = list) != null && (a = lst.elementData) != null) {
            if ((hi = fence) < 0) {
                mc = lst.modCount;
                hi = lst.size;
            }
            else
                mc = expectedModCount;
            if ((i = index) >= 0 && (index = hi) <= a.length) {
                for (; i < hi; ++i) {
                    @SuppressWarnings("unchecked") E e = (E) a[i];
                    action.accept(e);
                }
                if (lst.modCount == mc)
                    return;
            }
        }
        throw new ConcurrentModificationException();
    }

	// 评估当前迭代器还有多少个元素还未遍历
    public long estimateSize() {
        return (long) (getFence() - index);
    }

	// 获取当前迭代器的特征值：ORDERED表示按顺序进行分割迭代，比如ArrayList
	// SIZED和SUBSIZED表示迭代器为无序迭代器，比如HashSet；SUBSIZED是被分割过的无序迭代器
    public int characteristics() {
        return Spliterator.ORDERED | Spliterator.SIZED | Spliterator.SUBSIZED;
    }
}
```

下面是获取该迭代器的API：

```java
public Spliterator<E> spliterator() {
    return new ArrayListSpliterator<>(this, 0, -1, 0);
}
```

## 参考链接

* [Java 集合源码解析（1）：Iterator](https://shixin.blog.csdn.net/article/details/52743564)
* [jdk8中Spliterator的作用](https://www.cnblogs.com/nevermorewang/p/9368431.html)