<!--
date: 2021-06-03T22:34:12+08:00
lastmod: 2021-06-08T22:34:12+08:00
-->
## 前言

本文的源码分析主要基于JDK 1.8。

## ArrayList

### 概览

Java的数组是不可变的，因为数组的内存是连续分配的，在初始化之后就不能改变数组的长度，如果允许修改长度则无法保证内存是连续分配的。但在很多时候我们需要一个长度可变的数组，于是就有了ArrayList。

ArrayList基于动态数组实现，支持随机访问。（即实现了`RandomAccess`标记接口，属于有序的集合。随机访问，意味着元素在内存中为顺序存储，这样才可以通过下标来直接访问到任意元素。如果使用for循环遍历，效率会优于迭代器遍历。与之相对的概念是顺序访问与随机存储，比如链表的元素在内存中并不是连续存储的，因此只能从头开始遍历寻找元素。）

```java
public class ArrayList<E> extends AbstractList<E>
        implements List<E>, RandomAccess, Cloneable, java.io.Serializable
```

ArrayList底层维护着一个不可序列化的数组对象来存放元素：

```java
transient Object[] elementData;
```

默认的容器大小为10，也就是说在不指定ArrayList大小时，默认创建一个容量为10的数组：

```java
private static final int DEFAULT_CAPACITY = 10;
```

底层提供了两个空数组对象：

```java
// 指定ArrayList的容量为0时会使用这个空数组对象
private static final Object[] EMPTY_ELEMENTDATA = {};

// 不指定容量时会使用这个空数组对象，后续用以扩容，使用两个空数组对象是为了方便区分指定容量为0的情况
private static final Object[] DEFAULTCAPACITY_EMPTY_ELEMENTDATA = {};
```

ArrayList当前存放的元素数量：

```java
private int size;
```

ArrayList底层的数组对象的最大容量：

```java
// 由于一些虚拟机会在数组对象中预留一些对象头信息，比如数组对象长度等，所以这里减去8，以屏蔽掉不同虚拟机的实现差异
// 如果尝试指定超出最大容量的大小，会抛出OutOfMemoryError: Requested array size exceeds VM limit
private static final int MAX_ARRAY_SIZE = Integer.MAX_VALUE - 8;
```

* [从零开始学Java-你不知道的数组长度的秘密](https://blog.csdn.net/Evan_L/article/details/116545055)

### 获取元素

ArrayList在指定泛型之后，可以直接获取到对应类型的元素，因为底层帮我们进行了类型转换：

```java
public E get(int index) {
    rangeCheck(index);

    return elementData(index);
}

E elementData(int index) {
    return (E) elementData[index];
}
```

### 扩容

每次添加元素时通过`ensureCapacityInternal();`来确保容器足够，如果容量不足则通过`grow()`进行扩容：`oldCapacity + (oldCapacity >> 1)`，将容量变为原本的1.5倍。(`oldCapacity >> 1`需要取整，oldCapacity为偶数就是1.5倍，为奇数就是1.5倍-0.5）

扩容时需要通过`Arrays.copyOf()`来拷贝原本的数组到新数组中，该操作代价较高，因此需要尽量在创建ArrayList时就指定好适当的容量，以减少扩容的次数。之所以默认使用10作为容量，这是统计出来的结果，一般在使用时不会超过这个容量。如果超过时，最好一开始就自行指定容量。

`Arrays.copyOf()`最终是调用的本地方法`System.arraycopy()`来实现数组拷贝（是浅拷贝），比起普通的遍历拷贝方式效率更高。

```java
public boolean add(E e) {
    ensureCapacityInternal(size + 1);  // Increments modCount!!
    elementData[size++] = e;
    return true;
}

public boolean addAll(Collection<? extends E> c) {
    Object[] a = c.toArray();
    int numNew = a.length;
    ensureCapacityInternal(size + numNew);  // Increments modCount
    System.arraycopy(a, 0, elementData, size, numNew);
    size += numNew;
    return numNew != 0;
}

private void ensureCapacityInternal(int minCapacity) {
    if (elementData == DEFAULTCAPACITY_EMPTY_ELEMENTDATA) {
        minCapacity = Math.max(DEFAULT_CAPACITY, minCapacity);
    }

    ensureExplicitCapacity(minCapacity);
}

private void ensureExplicitCapacity(int minCapacity) {
    modCount++;

    // overflow-conscious code
    if (minCapacity - elementData.length > 0)
        grow(minCapacity);
}

private void grow(int minCapacity) {
    // overflow-conscious code
    int oldCapacity = elementData.length;
    int newCapacity = oldCapacity + (oldCapacity >> 1);
    if (newCapacity - minCapacity < 0)
        newCapacity = minCapacity;
    if (newCapacity - MAX_ARRAY_SIZE > 0)
        newCapacity = hugeCapacity(minCapacity);
    // minCapacity is usually close to size, so this is a win:
    elementData = Arrays.copyOf(elementData, newCapacity);
}
```

### 添加元素

添加元素分为两种，一种是不指定下标，此时添加到数组末尾，如果不发生扩容，时间复杂度只有O(1)；一种是添加到指定的下标处，此时会拷贝数组以进行移位，时间复杂度为O(n)。

```java
public boolean add(E e) {
    ensureCapacityInternal(size + 1);  // Increments modCount!!
    elementData[size++] = e;
    return true;
}

public boolean addAll(Collection<? extends E> c) {
    Object[] a = c.toArray();
    int numNew = a.length;
    ensureCapacityInternal(size + numNew);  // Increments modCount
    System.arraycopy(a, 0, elementData, size, numNew);
    size += numNew;
    return numNew != 0;
}

public void add(int index, E element) {
    rangeCheckForAdd(index);

    ensureCapacityInternal(size + 1);  // Increments modCount!!
    System.arraycopy(elementData, index, elementData, index + 1,
                     size - index);
    elementData[index] = element;
    size++;
}

public boolean addAll(int index, Collection<? extends E> c) {
    rangeCheckForAdd(index);

    Object[] a = c.toArray();
    int numNew = a.length;
    ensureCapacityInternal(size + numNew);  // Increments modCount

    int numMoved = size - index;
    if (numMoved > 0)
        System.arraycopy(elementData, index, elementData, index + numNew,
                         numMoved);

    System.arraycopy(a, 0, elementData, index, numNew);
    size += numNew;
    return numNew != 0;
}
```

### 删除元素

删除元素时会调用`System.arraycopy()`来将后面的元素向前移位，时间复杂度为O(n)，同样代价昂贵：

```java
    public E remove(int index) {
        rangeCheck(index);

        modCount++;
        E oldValue = elementData(index);

        int numMoved = size - index - 1;
        if (numMoved > 0)
            System.arraycopy(elementData, index+1, elementData, index,
                             numMoved);
        elementData[--size] = null; // clear to let GC do its work

        return oldValue;
    }
```

### 序列化

ArrayList底层保存元素的数组对象是`transient`的，该数组对象不会被序列化。但为了能序列化被保存的元素，ArrayList实现了对象流序列化相关的两个方法，当通过对象流来写入、读取ArrayList对象时，就会自动通过反射来调用这两个方法，注意这两个方法都必须是私有的：

```java
private void readObject(java.io.ObjectInputStream s)
    throws java.io.IOException, ClassNotFoundException {
    elementData = EMPTY_ELEMENTDATA;

    // Read in size, and any hidden stuff
    s.defaultReadObject();

    // Read in capacity
    s.readInt(); // ignored

    if (size > 0) {
        // be like clone(), allocate array based upon size not capacity
        ensureCapacityInternal(size);

        Object[] a = elementData;
        // Read in all elements in the proper order.
        for (int i=0; i<size; i++) {
            a[i] = s.readObject();
        }
    }
}

private void writeObject(java.io.ObjectOutputStream s)
    throws java.io.IOException{
    // Write out element count, and any hidden stuff
    int expectedModCount = modCount;
    s.defaultWriteObject();

    // Write out size as capacity for behavioural compatibility with clone()
    s.writeInt(size);

    // Write out all elements in the proper order.
    for (int i=0; i<size; i++) {
        s.writeObject(elementData[i]);
    }

    if (modCount != expectedModCount) {
        throw new ConcurrentModificationException();
    }
}
```

### Fail-Fast

Fail-Fast快速失败机制主要使用在非线程安全的容器中，通过`modCount`来记录当前容器对象的数据结构修改次数，当增删元素、或发生了扩容时，都会使该计数+1。

在对集合进行迭代、序列化等操作时，会通过比较该计数是否发生了变化来决定是否抛出`ConcurrentModificationException`。

同步的容器在用迭代器遍历时也会检测是否发生了`ConcurrentModificationException`，因为在遍历时是不期望有其他线程改变了容器底层的数据结构。

### JDK-6260652 Bug

ArrayList的有个构造器的实现比较奇怪，在注释里提及`6260652`的bug：

```java
public ArrayList(Collection<? extends E> c) {
    elementData = c.toArray();
    if ((size = elementData.length) != 0) {
        // c.toArray might (incorrectly) not return Object[] (see 6260652)
        if (elementData.getClass() != Object[].class)
            elementData = Arrays.copyOf(elementData, size, Object[].class);
    } else {
        // replace with empty array.
        this.elementData = EMPTY_ELEMENTDATA;
    }
}
```

ArrayList重写了`toArray()`方法：

```java
public Object[] toArray() {
    return Arrays.copyOf(elementData, size);
}
```

该方法的返回值类型是`Object[]`类型，但由于传入的参数是Collection接口，也就是说`c.toArray()`是存在多态性的。

如果此时传入的是`java.util.Arrays$ArrayList`，即`java.util.Arrays#asList()`生成的List对象，就会触发`JDK-6260652`这个bug，此时需要重新用`Arrays.copyOf(elementData, size, Object[].class)`拷贝一次，将原数组拷贝到一个新的`Object[]`里。

`java.util.Arrays#asList()`返回的是Arrays的内部类ArrayList（前文提及的容器中的适配器模式），该内部类同样重写了`toArray()`，虽然返回值是`Object[]`，但实际类型并不一定是`Object[]`类型（典型的父类引用指向子类对象）：

```java
private static class ArrayList<E> extends AbstractList<E>
    implements RandomAccess, java.io.Serializable
{
    private static final long serialVersionUID = -2764017481108945198L;
    private final E[] a;

    ArrayList(E[] array) {
        a = Objects.requireNonNull(array);
    }

    @Override
    public Object[] toArray() {
        return a.clone();
    }
}
```

对于这种数组对象，是无法往其中添加规定类型以外的对象的：

```java
public class Test {

    public static void main(String[] args) throws Exception {
        Object[] arr1 = new Object[10];
        arr1[0] = new Integer(1);    // ok
        arr1[1] = new Test();        // ok

        Object[] arr2 = Arrays.asList(1, 2, 3).toArray();
        arr2[0] = new Integer(1);    // ok
        arr2[1] = new Test();        // java.lang.ArrayStoreException
    }

}
```

为了避免发生`java.lang.ArrayStoreException`，所以需要在数组对象实际类型不是`Object[]`时重新拷贝一次：`Arrays.copyOf(elementData, size, Object[].class)`。

## Vector

### 同步

Vector跟ArrayList的实现类似，但它是一个同步的容器，在一些方法上使用了`synchronized`进行同步，因此性能较低。

```java
public synchronized boolean add(E e) {
    modCount++;
    ensureCapacityHelper(elementCount + 1);
    elementData[elementCount++] = e;
    return true;
}

public synchronized E get(int index) {
    if (index >= elementCount)
        throw new ArrayIndexOutOfBoundsException(index);

    return elementData(index);
}
```

### 扩容

Vector在扩容时会在原容量的基础上增大`capacityIncrement`，该变量可以由构造器传入，在不指定时默认为0；此时容器会扩容为原本的两倍。

```java
private void grow(int minCapacity) {
    // overflow-conscious code
    int oldCapacity = elementData.length;
    int newCapacity = oldCapacity + ((capacityIncrement > 0) ?
                                     capacityIncrement : oldCapacity);
    if (newCapacity - minCapacity < 0)
        newCapacity = minCapacity;
    if (newCapacity - MAX_ARRAY_SIZE > 0)
        newCapacity = hugeCapacity(minCapacity);
    elementData = Arrays.copyOf(elementData, newCapacity);
}
```

### Vector和ArrayList的比较

ArrayList是JDK 1.2提供的容器，而Vector则是JDK 1.0提供的容器。Vector相当于同步版本的ArrayList，由于在一些方法上使用了`synchronized`进行同步，开销更大，性能较低。如果需要同步时，不推荐使用Vector，可以使用`Collections.synchronizedList()`来将其转换成线程安全的容器后再使用（这是装饰者模式的应用，将已有对象传入另一个类的构造器中创建新的对象来增强实现）。

Vector每次扩容时变为原来的两倍，或者增大指定的扩容参数；ArrayList则是扩容为原来的1.5倍，对于奇数容量会取整（1.5倍-0.5）。

## CopyOnWriteArrayList

### 读写分离

CopyOnWriteArrayList是基于写入时复制的思想来设计的ArrayList，可以实现读写分离，即读取元素时不加锁，写入元素时加锁。底层通过一个独占锁（互斥锁）`ReentrantLock`来给写操作上锁进行同步，而保存元素的数组对象则是`volatile`的，保证在写入元素后可以让所有线程读取到最新的数组对象。

```java
/** The lock protecting all mutators */
final transient ReentrantLock lock = new ReentrantLock();

/** The array, accessed only via getArray/setArray. */
private transient volatile Object[] array;
```

直接读取元素，不加锁：

```java
@SuppressWarnings("unchecked")
private E get(Object[] a, int index) {
    return (E) a[index];
}

/**
 * {@inheritDoc}
 *
 * @throws IndexOutOfBoundsException {@inheritDoc}
 */
public E get(int index) {
    return get(getArray(), index);
}

/**
 * Gets the array.  Non-private so as to also be accessible
 * from CopyOnWriteArraySet class.
 */
final Object[] getArray() {
    return array;
}
```

每次写入元素时加锁，且会拷贝到新的数组，因此没有扩容的概念，写操作时非常耗费性能，并且在写入成功之前其他线程读取到的是旧的数组对象，因此无法保证实时性：

```java
public boolean add(E e) {
    final ReentrantLock lock = this.lock;
    lock.lock();
    try {
        Object[] elements = getArray();
        int len = elements.length;
        Object[] newElements = Arrays.copyOf(elements, len + 1);
        newElements[len] = e;
        setArray(newElements);
        return true;
    } finally {
        lock.unlock();
    }
}
```

### 适用场景

和Vector相比，CopyOnWriteArrayList在写操作的同时允许读操作，大大提高了读操作的性能，因此很适合**读多写少**的应用场景，比如缓存。

但是CopyOnWriteArrayList的缺点也很明显，首先是写操作的拷贝数组行为会导致内存占用变为原来两倍，如果原数组的元素很多，可能会导致内存不足而发生gc行为。此外，写操作需要时间，其他线程读取到的可能还是之前的值，无法满足实时性要求，所以CopyOnWriteArrayList**不适合内存敏感以及对实时性要求很高**的场景。

## LinkedList

### 概览

基于双向链表实现，LinkedList持有着头、尾两个结点，允许双向插入数据，并且在根据索引查询结点时，可以选择从更靠近索引位置的一端来遍历以提高查询效率：

```java
transient Node<E> first;

transient Node<E> last;
```

底层通过一个私有的静态内部类`Node`来存储结点信息，并持有着上一个和下一个结点数据：

```java
private static class Node<E> {
    E item;
    Node<E> next;
    Node<E> prev;

    Node(Node<E> prev, E element, Node<E> next) {
        this.item = element;
        this.next = next;
        this.prev = prev;
    }
}
```

### LinkedList和ArrayList的比较

ArrayList基于动态数组实现，LinkedList基于双向链表实现。ArrayList和LinkedList的区别可以归结为数组和链表的区别。

1）数组支持随机访问，但插入删除的代价很高，需要移动大量元素；<br>
2）链表不支持随机访问，但插入删除只需要改变指针；内存的利用率更高。

## 参考链接

* [Java 容器](http://www.cyc2018.xyz/Java/Java%20%E5%AE%B9%E5%99%A8.html)
* [Jdk 6260652 Bug](https://www.cnblogs.com/lsf90/p/5366325.html)
* [Java 集合深入理解（7）：ArrayList](https://shixin.blog.csdn.net/article/details/52853989)
* [Jdk 6260652 Bug](https://www.cnblogs.com/lsf90/p/5366325.html)
* [高并发编程之CopyOnWriteArrayList介绍](https://blog.csdn.net/weixin_42146366/article/details/88016527)
