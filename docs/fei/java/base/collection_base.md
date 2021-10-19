# 容器类

## 容器类分类

* `Collection`：一组元素的序列
  * `List`
  * `Set`
  * `Queue`

* `Map`：一组键值对的集合

容器类在判断对象之间是否相等时，会利用对象的`equals()`或`hashCode()`进行判断

容器类在排序对象时，该对象必须是可排序的(即实现了`Comparable`接口)

## List

一组有序的元素序列

* `ArrayList`

  以数组作为内部结构，随机访问元素快，中间插入和移动元素慢

* `LinkedList`

  以链表作为内部结构，随机访问元素慢，中间插入和移动元素快

### Stack

栈，具有后进先出(`LIFO`)特点的序列

JDK中的`Stack`继承自`Vector`类

## Set

一组无序且元素唯一的序列

* `HashSet`

  以散列表作为内部结构，查询元素快

* `TreeSet`

  以红黑树作为内部结构，保证了元素有序

* `LinkedHashSet`

  以散列表和链表作为内部结构，利用散列表提高了查询元素的效率，利用链表来维护元素的插入顺序

* `SortedSet`

  按对象中`Comparable`接口的实现来进行排序

## Queue

队列，具有先进先出(`FIFO`)特点的序列

> `LinkedList`实现了`Deque`接口，所以它也可以当成是`Queue`

### PriorityQueue

优先队列，下一个弹出的元素是优先级最高的，而不是等待时间最长的(`FIFO`)

优先队列在插入元素时，会根据对象中`Comparable`接口的实现来进行排序

## Map

* `HashMap`

  以散列表作为内部结构，查询元素快

* `LinkedHashMap`

  以散列表和链表作为内部结构，利用散列表提高了查询元素的效率，利用链表来维护元素的插入顺序

* `TreeMap`

  以红黑树作为内部结构，保证了元素有序

* `ConcurrentHashMap`

  线程安全的`HashMap`

### 散列

* 散列的目的

  想要使用一个对象来查找另一个对象

* 散列的原理

  将键用散列函数计算出散列码，该散列码作为数组的下标

  为了解决数组容量固定的问题，不同的键可以计算出相同的散列码，这种情况称为散列冲突(完美的散列函数不会产生冲突)

  由于存在散列冲突，数组(存储一组元素最快的数据结构)并不会存储单个值，而是存储值的集合`List`(线性查询时，会对`List`中的值使用`equals()`进行判断)

```java
class SimpleHashMap<K, V> extends AbstractMap<K, V> {

    static class SimpleEntry<K, V> implements Map.Entry<K, V> {

        K key;
        V value;

        SimpleEntry(K key, V value) {
            this.key = key;
            this.value = value;
        }

        @Override
        public K getKey() {
            return key;
        }

        @Override
        public V getValue() {
            return value;
        }

        @Override
        public V setValue(V value) {
            this.value = value;
            return value;
        }
    }

    static final int SIZE = 10;

    // buckets为散列表，bucket为其中的槽位slot
    LinkedList<SimpleEntry<K, V>>[] buckets = new LinkedList[SIZE];

    @Override
    public V put(K key, V value) {
        V oldValue = null;
        int index = Math.abs(key.hashCode()) % SIZE;
        if (buckets[index] == null) {
            buckets[index] = new LinkedList<>();
        }
        LinkedList<SimpleEntry<K, V>> bucket = buckets[index];
        SimpleEntry<K, V> pair = new SimpleEntry<>(key, value);
        boolean found = false;
        ListIterator<SimpleEntry<K, V>> iterator = bucket.listIterator();
        SimpleEntry<K, V> temp;
        while (iterator.hasNext()) {
            temp = iterator.next();
            if (temp.getKey().equals(key)) {
                oldValue = temp.getValue();
                iterator.set(pair);
                found = true;
                break;
            }
        }
        if (!found) {
            bucket.add(pair);
        }
        return oldValue;
    }

    @Override
    public V get(Object key) {
        int index = Math.abs(key.hashCode()) % SIZE;
        if (buckets[index] == null) {
            return null;
        }
        for (SimpleEntry<K, V> entry : buckets[index]) {
            if (entry.getKey().equals(key)) {
                return entry.getValue();
            }
        }
        return null;
    }

    @Override
    public Set<Entry<K, V>> entrySet() {
        Set<Entry<K, V>> set = new HashSet<>();
        for (LinkedList<SimpleEntry<K, V>> bucket : buckets) {
            if (bucket == null) {
                continue;
            }
            for (SimpleEntry<K, V> entry : bucket) {
                set.add(entry);
            }
        }
        return set;
    }

}

public class Test {
    public static void main(String[] args) {
        SimpleHashMap<String, String> map = new SimpleHashMap<>();
        map.put("f", "1");
        map.put("j", "2");
        map.put("h", "3");
        map.put("f", "4");
        map.forEach((k, v) -> System.out.println(k + ":" + v));
    }
}
```

### HashMap的性能因子

* 容量：表(散列表)中的桶位数

* 初始容量：表在创建时所拥有的桶位数

* 尺寸：表中当前存储的项数

* 负载因子：尺寸/容量

负载轻的表产生冲突的可能性小

`HashMap`能指定负载因子，当负载情况达到负载因子的水平时，容器将自动增加其容量(桶位数)，并将现有的存储数据重新分配到新的桶中，这个重新分配的过程被称为再散列

`HashMap`默认的负载因子为`0.75`，创建一个具有恰当大小的初始容器能避免自动再散列，导致消耗性能
