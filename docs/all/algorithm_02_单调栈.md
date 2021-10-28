<!--
date: 2021-10-26T20:34:12+08:00
lastmod: 2021-10-26T20:34:12+08:00
-->

## 单调栈

单调栈指的是一个栈中元素按照单调递增或者单调递减进行存储，这是一种**专门用于解决下一个更大元素（Next Greater Number）场景**的算法。

可以把这种场景看成这样：有一群身高不同的人排成一列，从正面看，个子高的人会挡住身后个子矮的人，然后找到比某个人身后最靠近他的更高的人。

下面是例题。

## 496. 下一个更大元素 I

题目：https://leetcode-cn.com/problems/next-greater-element-i/

对于本题，若使用暴力解法，需要O(m*n)的时间。如果借助栈，同时逆序遍历第二个数组，每次遍历元素时，将栈中所有比当前元素小于或对于的元素出栈，此时若栈不为空，则剩下的栈顶元素就是最接近当前元素的下一个更大元素；若栈中为空，说明在当前元素之后不存在比之更大的元素，按照题意返回-1。

这时候再将当前元素入栈，会在栈中逐渐形成一个自底向上的递减序列。由于题中说明不存在相同的元素，因此在每次遍历时，可以将当前元素的下一个更大元素存入哈希表中，方便最后返回结果数组。当然本题只需要找到下一个更大元素的值，对于某些题型，可能需要元素的下标来计算，此时哈希表就应该存入下标。

### 使用ArrayDeque来作为栈

ArrayDeque可以作为栈使用，也可以作为队列使用。不使用Stack是因为Stack继承了Vector，这两个容器都不推荐使用了。不使用LinkedList作为栈实现，是因为其底层是链表实现的，ArrayDeque底层是数组实现的：

1）链表比数组花费更多空间<br>
2）链表的随机访问性质比数组差（虽然这个对栈来说问题不大）<br>
3）链表的每次插入和删除都涉及到一个节点对象的创建和弃用，非常低效和浪费空间，而动态数组几乎是0花费的(数组充满时重新拷贝除外)<br>
4）链表是非连续的，访问时候不能充分利用cpu cache

```java
class Solution {
    public int[] nextGreaterElement(int[] nums1, int[] nums2) {
        // 单调栈 + 哈希表
        Deque<Integer> stack = new ArrayDeque<>();
        Map<Integer, Integer> map = new HashMap<Integer, Integer>();
        for (int i = nums2.length - 1; i >= 0; i--) {
            while (!stack.isEmpty() && stack.peek() <= nums2[i]) {
                stack.pop();
            }
            map.put(nums2[i], stack.isEmpty() ? -1 : stack.peek());
            stack.push(nums2[i]);
        }
        
        int[] res = new int[nums1.length];
        for (int i = 0; i < nums1.length; i++) {
            res[i] = map.getOrDefault(nums1[i], -1);
        }

        return res;
    }
}
```

单调栈+哈希表的时间复杂度为O(m+n)，第一个for循环嵌套while循环的时间复杂度为O(m)，是因为在遍历过程中，每个元素都至多只会有一次`pop()`和`push()`操作，换言之，while循环的时间复杂度是常数，整体的复杂度还是O(m)。

## 503. 下一个更大元素 II

题目：https://leetcode-cn.com/problems/next-greater-element-ii/

本题是[496题](https://javanote.doc.lewky.cn/#/all/algorithm_02_单调栈?id=_496-%e4%b8%8b%e4%b8%80%e4%b8%aa%e6%9b%b4%e5%a4%a7%e5%85%83%e7%b4%a0-i)的升级版，要遍历的数组是一个循环数组，并且数组内的元素可以相等，这意味着不能使用哈希表来存下一个更大的值。

遍历循环数组，可以看成是遍历2n次原本的数组，比如原本的循环数组是`[1,2,1]`，相当于遍历一个普通数组`[1,2,1,1,2,1]`，这时候可以用取模操作来获取对应的元素，而不需要额外拷贝一个2倍长度的数组。

在遍历过程中，当下标在原本数组范围内时再将对应的下一个更大元素存入结果数组中即可。

```java
    public int[] nextGreaterElements(int[] nums) {
        Deque<Integer> stack = new ArrayDeque<>();
        int[] res = new int[nums.length];
        for (int i = (nums.length << 1) - 1; i >= 0; i--) {
            int num = nums[i % nums.length];
            while (!stack.isEmpty() && stack.peek() <= num) {
                stack.pop();
            }
            if (i <= nums.length - 1) {
                res[i] = stack.isEmpty() ? -1 : stack.peek();
            }
            stack.push(num);
        }

        return res;
    }
```

## 456. 132 模式

题目：https://leetcode-cn.com/problems/132-pattern/


## 42. 接雨水

题目：https://leetcode-cn.com/problems/trapping-rain-water/



## 参考链接

* [为什么JDK建议使用ArrayDeque而不是Stack和LinkedList实现栈和队列](https://www.cnblogs.com/jiading/articles/12452830.html)