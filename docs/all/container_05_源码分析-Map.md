<!--
date: 2021-06-06T22:34:12+08:00
lastmod: 2021-06-28T22:38:12+08:00
-->
## 前言

本文的源码分析主要基于JDK 1.8。

## HashMap

### 概览

HashMap是一个基于键值对存储的哈希表（Hash table），它不是**线程安全**的。HashMap的底层使用`Node`数组来存储键值对，该数组长度总是2的N次幂（即2^n）：

```java
// 存储键值对结点的数组
transient Node<K,V>[] table;
```

并且维护了一个entrySet作为cache，以供`keySet()`和`values()`使用：

```java
// 存放了所有的键值对，作为cache使用
transient Set<Map.Entry<K,V>> entrySet;
```

`Node`节点是HashMap的静态内部类，除了持有着自身的hash、键值对，还持有着下一个结点的值：

```java
static class Node<K,V> implements Map.Entry<K,V> {
    final int hash;
    final K key;
    V value;
    Node<K,V> next;

    Node(int hash, K key, V value, Node<K,V> next) {
        this.hash = hash;
        this.key = key;
        this.value = value;
        this.next = next;
    }
}
```

从这里可以看出，HashMap底层是通过**数组+单链表**来实现存储的，相同hash的结点会被保存到数组的同一个下标中，即同一个桶中（bin）。此时这些发生哈希碰撞的结点会通过**尾插法**来形成一个单链表，这就是所谓的**拉链法**。拉链法是在发生哈希碰撞时的一种解决方案，将相同哈希值的元素编入同一个单链表中。虽然能节省空间，但如果这个链表太长，会影响遍历元素的效率，从O(1)退化到O(n)。为了改善这种情况，JDK 1.8引入了红黑树。

### 为什么HashMap的容量总是2的N次幂

HashMap是基于哈希表的思想实现的，在操作元素时离不开对key求hash，然后根据key的hash来求出对应的value被保存在哈希表的哪个桶中。在HashMap中，这个桶就是对应的底层`Node<K,V>[]`的下标位置。

按照普通的逻辑，就是直接拿hash对数组长度length进行取模操作，得到的余数在`0`~`length-1`之间，正好对应数组下标区间。

为了尽可能地降低这个取模操作的性能损耗，HashMap使用了位运算`(n - 1) & hash`来取代取模运算，这里的n就是数组的length。位运算的效率比取模会高效很多，因为是对内存数据进行直接操作，不需要进制转换。但是使用这个位运算的前提是，n必须是2的N次幂，比如默认的初始容量是16，其二进制形式就是`00010000`。

2的N次幂的二进制数只有最高位是1，其他都是0；将其-1后，则高位会变成0，低位全部变成1。此时再与hash做&运算，意味着只会保留最后4位，其他位都会被置为0。而这最后4位，其实就相当于用hash对length进行取模后得到的余数。

```java
// 十进制的16
int n1 = 16;
// 二进制的16
byte n2 = 0b00010000;
// 16-1
byte n3 = 0b00001111;

int hash = 17;
// 二进制的17
byte n4 = 0b00010001;

// 结果为1，正好是两数相除的余数
int index = hash & (n1 - 1);
```

另一个方面，是为了提高扩容时的效率。HashMap在扩容之后需要重新对结点计算对应的桶下标，如果容量是2的N次幂，且每次都扩容为之前的两倍，则在重新计算桶下标时会有极大的便利。

### 计算hash

HashMap在对key计算hash时，如果key是null，则返回hash为0；否则先通过`hashCode()`算出key对象本身的哈希值，然后将该哈希值的高位（前16位）通过异或扩散到低位（后16位）。

```java
static final int hash(Object key) {
    int h;
    return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
}
```

**为什么这里不直接使用key的hashCode作为哈希值，而是使用异或将hashCode的高位传播到低位？**

这主要是为了降低哈希碰撞的概率，因为HashMap中通过`(length - 1) & hash`的位运算来计算桶下标，而length一般情况下并不会很大，初始容器只有16，这意味着对于`(length - 1) & hash`的位运算来说，hash的高16位其实没有被使用到，起到决定作用的其实是低16位。所以在计算hash时，通过异或操作将高位传播到低位来一起参与桶下标的位运算，可以降低哈希碰撞的概率。

### 扩容

HashMap的扩容和几个变量有关：

```java
// 默认的负载因子
static final float DEFAULT_LOAD_FACTOR = 0.75f;
// 默认的容量为16
static final int DEFAULT_INITIAL_CAPACITY = 1 << 4;

// 已经存放的键值对的数量
transient int size;
// 发生扩容的阈值
int threshold;
// 负载因子
final float loadFactor;
```

创建HashMap时，如果不指定负载因子，则默认为`0.75f`，一般不手动指定负载因子，默认的负载因子是基于统计得到的最佳因子。若不指定容量则默认为16，如果指定了容量大小，则会将容量设为不小于指定容量的一个2的N次幂数：

```java
static final int tableSizeFor(int cap) {
    int n = cap - 1;
    n |= n >>> 1;
    n |= n >>> 2;
    n |= n >>> 4;
    n |= n >>> 8;
    n |= n >>> 16;
    return (n < 0) ? 1 : (n >= MAXIMUM_CAPACITY) ? MAXIMUM_CAPACITY : n + 1;
}
```

这个算法是基于位运算来快速得到一个不小于指定的数字的2的N次幂数，比如指定容量为8，则会得到结果8；指定容量为9，则会得到结果16。

以9为例，思路如下：
```java
// 十进制的9
int cap = 9;
// 二进制的9
byte n = 0b00001001;

// 由于2的N次幂数都是最高位为1，低位为0
// 最靠近且不小于9的2的N次幂数是16
byte target = 0b00010000;

// 如果将16-1，则二进制如下
byte temp = 0b00001111;
// 此时可以发现，这个二进制数跟指定容量的二进制数的位数一致
// 那么只需要将指定容量的二进制数的所有位都置为1，再将结果+1，既可以得到最靠近且不小于9的2的N次幂数
// 对于一个int数值，只需要进行五次无符号右移和|=运算，即可实现这个操作
// a |= b 相当于 a = a | b
n |= n >>> 1;
n |= n >>> 2;
n |= n >>> 4;
n |= n >>> 8;
n |= n >>> 16;

// 此时n就变成了temp，然后n+1，就可以得到16这个数
// 但是如果指定容量一开始就是一个2的N次幂数，比如8，上面这种算法同样会得到16的结果，而我们想要的是8
// 于是需要一开始就将指定容量-1，这样输入8就能得到8
```

既然要求容量总是2的N次幂数，那么HashMap的扩容自然也只能扩容为原本的2倍了，这样可以满足2的N次幂数要求，又不至于扩容倍数太大而浪费空间。

至于什么时候发生扩容，则取决于这个HashMap的填充程度，因为HashMap底层是数组+单链表的结构，不可能等到整个数组都放满了结点才进行扩容，这样有可能某些桶中已经堆积了很长的链表，会导致获取结点的效率大幅下降。

于是就有了负载因子的概念，HashMap的扩容阈值threshold值为容量和负载因子的乘积，当HashMap的`size > threshold`时，则发生扩容，容量变为原本两倍：`newCap = oldCap << 1`，然后对原本存储的结点进行重新计算桶下标：`resize()`。

这里着重讲下最后的单链表结点怎么重新计算桶下标:

```java
// resize()中单链表结点重新计算桶下标

Node<K,V> loHead = null, loTail = null;
Node<K,V> hiHead = null, hiTail = null;
Node<K,V> next;
do {
    next = e.next;
    if ((e.hash & oldCap) == 0) {
        if (loTail == null)
            loHead = e;
        else
            loTail.next = e;
        loTail = e;
    }
    else {
        if (hiTail == null)
            hiHead = e;
        else
            hiTail.next = e;
        hiTail = e;
    }
} while ((e = next) != null);
if (loTail != null) {
    loTail.next = null;
    newTab[j] = loHead;
}
if (hiTail != null) {
    hiTail.next = null;
    newTab[j + oldCap] = hiHead;
}
```

由于容量总为2的N次幂，当扩容两倍后，相当于左移一位，前文已经讲了桶下标的计算方式，是通过`(length - 1) & hash`的位运算来取代取模运算。扩容之后，hash不变，容量变为两倍，`newCap - 1`的二进制结果跟原本比起来，相当于比`oldCap - 1`多了一个`1`的最高位。

这意味着，重新计算桶下标时，只需要比较`(newCap - 1) & hash`的最高位是否为1，就可以判断出桶下标是否需要发生变化。而`hash & oldCap`，正好可以得到这个最高位的值。

如果最高位为0，则结果和之前一样，无需改变结点位置，新的桶下标跟原本一样：`newTab[j]`。

如果最高位为1，则意味着余数比之前变大了旧容量的大小，也就是说，新的桶下标等于原本的桶下标+旧容量：`newTab[j + oldCap]`。

可以看到，为了尽可能提高HashMap的性能，底层使用了各种位运算和巧妙的设计，真的是玩出花来。

### 为什么jdk 1.8的HashMap要引入红黑树

当某个桶的单链表太长时，获取该桶上的结点时间复杂度将会退化为O(n)，为了提高性能，当**链表结点大于等于8个，且数组长度大于等于64时，链表会转化为红黑树。**

```java
// 转化为红黑树的链表结点数阈值
static final int TREEIFY_THRESHOLD = 8;
// 退化回链表的结点数阈值
static final int UNTREEIFY_THRESHOLD = 6;
// 转化为红黑树的容器最小阈值
static final int MIN_TREEIFY_CAPACITY = 64;
```

为什么要规定单链表结点阈值为8呢？这个数字同样是统计出来的，假定哈希算法分布均匀，按照泊松分布模型（Poisson_distribution），单链表的结点数超过8的概率少于千万分之一：

```java
/*
 * Ideally, under random hashCodes, the frequency of
 * nodes in bins follows a Poisson distribution
 * (http://en.wikipedia.org/wiki/Poisson_distribution) with a
 * parameter of about 0.5 on average for the default resizing
 * threshold of 0.75, although with a large variance because of
 * resizing granularity. Ignoring variance, the expected
 * occurrences of list size k are (exp(-0.5) * pow(0.5, k) /
 * factorial(k)). The first values are:
 *
 * 0:    0.60653066
 * 1:    0.30326533
 * 2:    0.07581633
 * 3:    0.01263606
 * 4:    0.00157952
 * 5:    0.00015795
 * 6:    0.00001316
 * 7:    0.00000094
 * 8:    0.00000006
 * more: less than 1 in ten million
 */
```

从源码注释可以看出，实际上HashMap是不太希望会生成红黑树的。因为转化为红黑树、红黑树的旋转这些操作，都是很耗费性能的。这也是为什么除了规定单链表结点数要大于等于8之外，还要求当前容量应当大于等于64。但容量小于64时，说明是容量不足导致的哈希碰撞严重，此时不应该生成红黑树，而是应该进行扩容。

```java
final void treeifyBin(Node<K,V>[] tab, int hash) {
    int n, index; Node<K,V> e;
    if (tab == null || (n = tab.length) < MIN_TREEIFY_CAPACITY)
        resize();
    else if ((e = tab[index = (n - 1) & hash]) != null) {
        TreeNode<K,V> hd = null, tl = null;
        do {
            TreeNode<K,V> p = replacementTreeNode(e, null);
            if (tl == null)
                hd = p;
            else {
                p.prev = tl;
                tl.next = p;
            }
            tl = p;
        } while ((e = e.next) != null);
        if ((tab[index] = hd) != null)
            hd.treeify(tab);
    }
}
```

当红黑树中的结点数少于等于6时，会退化回链表，可以看到红黑树的生成和退化这两个阈值并不是连续的，这是为了留一个缓冲的空间，避免刚生成红黑树后因为被移除了一个树结点就马上退化为链表，如果正巧又有一个结点添加到这个链表就会再次转化为红黑树，这样会很耗费性能。

### jdk 1.7的HashMap链表头插法导致的死循环

在jdk 1.7中，HashMap使用了头插法来给单链表插入新的结点，因为新创建的结点往往更常被使用，使用头插法可以更快获取到最新插入的结点。

在扩容时生成的新链表同样通过头插法来生成，不过由于是遍一边历原链表结点一边插入到新链表中，相当于生成了原链表的反序链表。而HashMap又是线程不安全的，这导致了在高并发场景下很容易在扩容时产生环形链表，有时候甚至会导致死循环。

```java
void transfer(Entry[] newTable, boolean rehash) {
    int newCapacity = newTable.length;
    for (Entry<K,V> e : table) {
        while(null != e) {
            Entry<K,V> next = e.next;
            if (rehash) {
                e.hash = null == e.key ? 0 : hash(e.key);
            }
            int i = indexFor(e.hash, newCapacity);
            e.next = newTable[i];
            newTable[i] = e;
            e = next;
        }
    }
}
```

假设原链表中的结点为：`Node(A) -> Node(B) -> null`

在扩容时，遍历原链表，将原结点用头插法插入到新链表中，最终生成：`Node(B) -> Node(A) -> null`

如果在并发场景中同时有两个线程进行扩容，线程A执行到`Entry<K,V> next = e.next;`挂起，此时e为`Node(A)`，`next`为`Node(B)`。而table是底层存储键值对的数组，属于线程共享的内存，被遍历的原链表同样是被线程共享的，此时`Node(A) -> Node(B)`。

然后在线程A挂起的期间，线程B执行完扩容，生成了新的反序链表：`Node(B) -> Node(A) -> null`。此时线程A被唤醒，继续执行，`newTable[i]`的链表为：`Node(A) -> null`

接着进入第二次循环，由于`Node(B)`被线程B修改了，从原本的指向null变成了指向`Node(A)`，这意味着原本只需遍历二次，此时因为存在下一个结点，需要进行第三次循环。而经过第二次循环，`newTable[i]`的链表为：`Node(B) -> Node(A) -> null`

接着进入第三次循环，此时`next`为null，不会有下一次循环了。接着通过头插法将`Node(A)`插入到新链表头部，于是就形成了环形链表：`Node(A) -> Node(B) -> Node(A)`。

接下来，一旦需要遍历该链表，就会陷入死循环。当然，HashMap本身就不是线程安全的，在多线程场景中应该使用其他容器。

在jdk 1.8中，使用的是尾插法，扩容时生成的新链表不再是原链表的反序，避免了多线程时形成环形链表而引发死循环，但在多线程场景中依然存在其他的问题。

### HashMap和Hashtable的比较

1）Hashtable使用synchronized进行同步<br>
2）Hashtable不允许key、value为null，HashMap则都可以为null<br>
3）HashMap的迭代器是fail-fast迭代器<br>
4）Hashtable的扩容是原来容量的二倍加1（2n+1），HashMap每次扩容为之前的两倍<br>
5）Hashtable在计算桶下标时使用除法运算，效率较低；HashMap使用位运算来求桶下标

## ConcurrentHashMap

实现和HashMap类似，但ConcurrentHashMap是线程安全的。

在jdk 1.7中，ConcurrentHashMap则是使用的分段锁（Segment）来进行并发更新操作，每个分段锁维护着几个桶（HashEntry），多个线程可以同时访问不同分段锁上的桶，从而使其并发度更高（并发度就是 Segment 的个数）。

Segment继承自ReentrantLock，默认的并发级别为16，即默认创建16个Segment。由于Segment的存在，在获取当前的键值对总数size时，需要遍历每个Segment上的键值对数量并累计起来。

在执行`size()`时先尝试在不加锁的情况下累计所有Segment的键值对数量`count`，同时也会累计所有Segment的数据结构改动次数`modCount`。如果连续两次不加锁的`modCount`统计结果一致，则认为此时统计得到的size是正确的。

但尝试次数超过3次时，会对所有Segment加锁再继续统计，在连续两次统计结果一致时，返回size并对所有Segment解锁。

而在jdk 1.8中，Segment已被CAS + synchronized取代。在添加新的键值对时，如果对应的桶下标不存在结点则会用CAS进行更新；如果已存在结点则会用synchronized进行同步，将新结点添加到原结点链表的末尾或者是插入到红黑树中。

## LinkedHashMap

### 概览

继承自HashMap，在HashMap的基础之上，额外维护了一个双向链表，用来维护插入顺序或访问顺序（LRU）：

```java
static class Entry<K,V> extends HashMap.Node<K,V> {
    Entry<K,V> before, after;
    Entry(int hash, K key, V value, Node<K,V> next) {
        super(hash, key, value, next);
    }
}

/**
 * The head (eldest) of the doubly linked list.
 */
transient LinkedHashMap.Entry<K,V> head;

/**
 * The tail (youngest) of the doubly linked list.
 */
transient LinkedHashMap.Entry<K,V> tail;
```

成员变量`accessOrder`决定了维护的顺序，true表示访问顺序，false表示插入顺序。默认维护的是插入顺序：

```java
final boolean accessOrder;
```

在HashMap中可以看到三个空实现的方法，LinkedHashMap对其进行了重写：

```java
// Callbacks to allow LinkedHashMap post-actions
void afterNodeAccess(Node<K,V> p) { }
void afterNodeInsertion(boolean evict) { }
void afterNodeRemoval(Node<K,V> p) { }
```

`afterNodeAccess`和`afterNodeInsertion`这两个方法负责维护双向链表中结点的顺序，会在`put`、`get`等方法中被调用。

### afterNodeAccess方法

当一个结点被访问时，如果`accessOrder`为true，即维护的是访问顺序时，若该结点不是尾结点，则将其移动至双向链表的末尾。此时，链表的头结点即为最近最少被使用的结点，因此可以利用这种特性来实现LRU缓存。

```java
void afterNodeAccess(Node<K,V> e) { // move node to last
    LinkedHashMap.Entry<K,V> last;
    if (accessOrder && (last = tail) != e) {
        LinkedHashMap.Entry<K,V> p =
            (LinkedHashMap.Entry<K,V>)e, b = p.before, a = p.after;
        p.after = null;
        if (b == null)
            head = a;
        else
            b.after = a;
        if (a != null)
            a.before = b;
        else
            last = b;
        if (last == null)
            head = p;
        else {
            p.before = last;
            last.after = p;
        }
        tail = p;
        ++modCount;
    }
}
```

### afterNodeInsertion方法

当一个结点插入时，会判断是否移除最旧的结点，即移除头结点：

```java
void afterNodeInsertion(boolean evict) { // possibly remove eldest
    LinkedHashMap.Entry<K,V> first;
    if (evict && (first = head) != null && removeEldestEntry(first)) {
        K key = first.key;
        removeNode(hash(key), key, null, false, true);
    }
}
```

这里的`evict`参数只有在构建map时才是false，`removeEldestEntry`方法则是判断是否满足移除头结点的条件，默认实现是返回false。

```java
protected boolean removeEldestEntry(Map.Entry<K,V> eldest) {
    return false;
}
```

如果要用LinkedHashMap来实现LRU缓存，则需要重写`removeEldestEntry`方法，这样才能自动移除最旧的结点，避免内存不足，且驻留在缓存中的都是热点数据。

### 一个简单的LRU缓存实现

```java
public class LRUCache<K, V> extends LinkedHashMap<K, V> {

    private int size;

    public LRUCache(int size) {
        // 16是初始化容量，0.75是负载因子，true表示按照访问排序
        super(16, (float) 0.75, true);
        this.size = size;
    }

    @Override
    // 该方法会在`put`和`putAll`插入元素之后自行调用，返回true表示应该删除最旧的元素。
    protected boolean removeEldestEntry(java.util.Map.Entry<K, V> eldest) {
        return size() > size;
    }
}
```

## WeakHashMap

### 概览

WeakHashMap继承自AbstractMap，实现了Map接口，这是一个使用弱引用对象来包装key的map容器。（JDK 1.2扩充了引用的概念，细分为强引用、软引用、弱引用、虚引用四种引用。）

WeakHashMap的Entry继承自WeakReference，底层使用该Entry数组来存储键值对。在下一次垃圾回收时弱引用对象中的key会被回收，然后该弱引用对象Entry会被放到一个队列中。

```java
private static class Entry<K,V> extends WeakReference<Object> implements Map.Entry<K,V> {
    V value;
    final int hash;
    Entry<K,V> next;

    /**
     * Creates new entry.
     */
    Entry(Object key, V value,
          ReferenceQueue<Object> queue,
          int hash, Entry<K,V> next) {
        super(key, queue);
        this.value = value;
        this.hash  = hash;
        this.next  = next;
    }
}
```

每当发生垃圾回收后，ReferenceQueue中会存放被key被set成null的Entry对象，在调用`size()`、`resize()`、`get()`、`put()`等方法时，都会调用一个`expungeStaleEntries()`方法来移除queue中被回收了key的Entry对象：

```java
private void expungeStaleEntries() {
    for (Object x; (x = queue.poll()) != null; ) {
        synchronized (queue) {
            @SuppressWarnings("unchecked")
                Entry<K,V> e = (Entry<K,V>) x;
            int i = indexFor(e.hash, table.length);

            Entry<K,V> prev = table[i];
            Entry<K,V> p = prev;
            while (p != null) {
                Entry<K,V> next = p.next;
                if (p == e) {
                    if (prev == e)
                        table[i] = next;
                    else
                        prev.next = next;
                    // Must not null out e.next;
                    // stale entries may be in use by a HashIterator
                    e.value = null; // Help GC
                    size--;
                    break;
                }
                prev = p;
                p = next;
            }
        }
    }
}
```

利用这种弱引用对象的特性，可以用WeakHashMap来实现缓存。但是需要注意的是，不能使用及基础类型的包装类作为Key，因为包装类存在着缓存，这些缓存是不会被回收的，这会导致由缓存作为的Key所对应的Value对象永远不会被GC自动回收。这样就失去了使用WeakHashMap的意义。

### ConcurrentCache

Tomcat中的ConcurrentCache就使用了WeakHashMap来实现缓存功能。

ConcurrentCache采取的是分代缓存：

● 经常使用的对象放入eden（伊甸园）中，eden使用ConcurrentHashMap实现，不用担心会被回收。<br>
● 不常用的对象放入longterm，longterm使用WeakHashMap实现，这些老对象会被垃圾收集器回收。<br>
● 当调用get()方法时，会先从eden区获取，如果没有找到的话再到longterm获取，当从longterm获取到就把对象放入eden中，从而保证经常被访问的节点不容易被回收。<br>
● 当调用put()方法时，如果eden的大小超过了size，那么就将eden中的所有对象都放入longterm中，利用虚拟机回收掉一部分不经常使用的对象。

```java
public final class ConcurrentCache<K, V> {

    private final int size;

    private final Map<K, V> eden;

    private final Map<K, V> longterm;

    public ConcurrentCache(int size) {
        this.size = size;
        this.eden = new ConcurrentHashMap<>(size);
        this.longterm = new WeakHashMap<>(size);
    }

    public V get(K k) {
        V v = this.eden.get(k);
        if (v == null) {
            v = this.longterm.get(k);
            if (v != null)
                this.eden.put(k, v);
        }
        return v;
    }

    public void put(K k, V v) {
        if (this.eden.size() >= size) {
            this.longterm.putAll(this.eden);
            this.eden.clear();
        }
        this.eden.put(k, v);
    }
}
```

## 参考链接

* [Java 容器](http://www.cyc2018.xyz/Java/Java%20%E5%AE%B9%E5%99%A8.html)
* [hashmap为什么长度要是2的n次幂](https://blog.csdn.net/darkness0604/article/details/105020267)
* [HashMap之tableSizeFor方法图解](https://www.cnblogs.com/xiyixiaodao/p/14483876.html)
* [聊聊经典数据结构HashMap,逐行分析每一个关键点](https://baijiahao.baidu.com/s?id=1679219630514764243&wfr=spider&for=pc)
* [jdk1.7HashMap链表头插法导致的死循环](https://blog.csdn.net/thqtzq/article/details/90485663)
* [一文搞懂WeakHashMap工作原理（java后端面试高薪必备知识点）](https://baijiahao.baidu.com/s?id=1666368292461068600&wfr=spider&for=pc)
