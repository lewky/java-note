<!--
date: 2021-11-09T10:34:12+08:00
lastmod: 2021-11-09T10:34:12+08:00
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

