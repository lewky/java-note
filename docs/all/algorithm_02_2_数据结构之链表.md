<!--
date: 2021-11-09T10:34:12+08:00
lastmod: 2021-11-17T10:34:12+08:00
-->

## 141. 环形链表

题目：https://leetcode-cn.com/problems/linked-list-cycle/

题目本身很简单，难的是进阶要求：使用 O(1)（即，常量）内存解决此问题。常规解法，不考虑进阶要求，借助HashSet即可找出链表是否出现过重复节点。

### 哈希表

```java
public class Solution {
    public boolean hasCycle(ListNode head) {
        Set<ListNode> seen = new HashSet<ListNode>();
        while (head != null) {
            if (!seen.add(head)) {
                return true;
            }
            head = head.next;
        }
        return false;
    }
}
```

### 快慢指针

为了实现进阶要求，需要使用到两个遍历速度不同的指针。一个指针遍历速度快，一个遍历速度慢，如果是一个环形链表，那么遍历速度快的指针最终必定能追上慢指针。

这就是所谓的龟兔赛跑算法（即`Floyd 判圈算法`），如果不是环形链表，那么快指针很快就会遍历到链表的尾节点，然后退出循环。

```java
public class Solution {
    public boolean hasCycle(final ListNode head) {
        if (head == null || head.next == null) {
            return false;
        }
        ListNode slow = head;
        ListNode fast = head.next;
        while (slow != fast) {
            // 快指针遍历速度更快，因此只需要判断快指针和其下一个节点是否为null
            if (fast == null || fast.next == null) {
                return false;
            }
            slow = slow.next;
            fast = fast.next.next;
        }

        return true;
    }
}
```

## 206. 反转链表

题目：https://leetcode-cn.com/problems/reverse-linked-list/

### 迭代法

双指针记录当前结点和上一个结点，迭代过程中逐渐反转链表结点。

```java
class Solution {
    public ListNode reverseList(final ListNode head) {
        ListNode cur = head;
        ListNode pre = null;
        while (cur != null) {
            ListNode next = cur.next;
            cur.next = pre;
            pre = cur;
            cur = next;
        }

        return pre;
    }
}
```

### 递归法

递归法比较复杂，在递归反转结点的过程中，当前结点的下一个结点的下一个结点需要指向当前结点，即：

原本链表： a -> b -> c

反转之后： a <- b <- c

需要注意的是，第一个结点的下一个结点必须指向null，否则最终会形成环形链表。

```java
class Solution {
    public ListNode reverseList(ListNode head) {
        if (head == null || head.next == null) {
            return head;
        }
        ListNode newHead = reverseList(head.next);
        head.next.next = head;
		// 避免递归返回到第一个结点时，形成环，即第一个结点指向第二个结点，第二个结点又指向第一个结点
        head.next = null;
        return newHead;
    }
}
```

## 203. 移除链表元素

题目：https://leetcode-cn.com/problems/remove-linked-list-elements/

### 双指针

常规思路是使用两个指针，分别记录当前结点和上一个结点，在遍历删除的过程中记录新的头结点：

```java
class Solution {
    public ListNode removeElements(ListNode head, int val) {
        ListNode cur = head;
        ListNode pre = null;
        ListNode newHead = null;
        while (cur != null) {
            if (cur.val == val) {
                ListNode next = cur.next;
                cur.next = null;
                cur = next;
                if (pre != null) {
                    pre.next = cur;
                }
            } else {
                if (newHead == null) {
                    newHead = cur;
                }
                pre = cur;
                cur = cur.next;
            }
        }

        return newHead;
    }
}
```

### 虚拟头结点

注意到上一个思路的条件分支比较多，原因是需要考虑到头结点需要删除的情况。如果我们创建一个假的头结点，然后指向真正的头结点，那么就可以无需写这么多的条件分支。

这种做法巧妙地将头结点需要删除的特殊情况，转换成了普通结点需要删除的场景。

```java
class Solution {
    public ListNode removeElements(ListNode head, int val) {
        ListNode dummyHead = new ListNode(0, head);
        ListNode cur = dummyHead;
        while (cur.next != null) {
            if (cur.next.val == val) {
                cur.next = cur.next.next;
            } else {
                cur = cur.next;
            }
        }

        return dummyHead.next;
    }
}
```

## 146. LRU 缓存机制

题目：https://leetcode-cn.com/problems/lru-cache/

实际上就是使用哈希表 + 双向链表来实现，哈希表负责存键值对，双向链表负责维护节点顺序，这里维护的是最近使用的顺序。Java中已经提供了这样的实现，即LinkedHashMap，有兴趣的也可以看看源码，自己手写一个。

```java
class LRUCache extends LinkedHashMap<Integer, Integer> {
    
    private int capacity;

    public LRUCache(final int capacity) {
        super(capacity, 0.75f, true);
        this.capacity = capacity;
    }

    public int get(final int key) {

        return super.getOrDefault(key, -1);
    }

    public void put(final int key, final int value) {
        super.put(key, value);
    }
    
    @Override
    // 该方法会在`put`和`putAll`插入元素之后自行调用，返回true表示应该删除最旧的元素。
    protected boolean removeEldestEntry(java.util.Map.Entry<Integer, Integer> eldest) {        
        return size() > capacity;
    }
}
```
