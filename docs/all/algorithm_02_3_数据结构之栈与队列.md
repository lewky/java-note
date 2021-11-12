<!--
date: 2021-11-09T10:34:12+08:00
lastmod: 2021-11-12T10:34:12+08:00
-->

# 栈

由于栈**后入先出**的特性，通常用来处理一些特别的场景问题，比如消消乐问题，这类问题需要按顺序消除字符、或者数字等。

Java的栈推荐使用ArrayDeque。

* [为什么使用ArrayDeque作为栈](https://javanote.doc.lewky.cn/#/all/algorithm_03_单调栈?id=%e4%b8%ba%e4%bb%80%e4%b9%88%e4%bd%bf%e7%94%a8arraydeque%e4%bd%9c%e4%b8%ba%e6%a0%88)

## 20. 有效的括号

题目：https://leetcode-cn.com/problems/valid-parentheses/

由题意可知，有效字符串只能是偶数长度，故奇数长度直接返回false。

括号类型必须按顺序结对才有效，换言之，将字符串中的括号按顺序结对消除，即左边的括号入栈，遇到对应的右边括号则出栈，最终若栈为空则说明是有效字符串。

```java
class Solution {
    public boolean isValid(String s) {
        // 有效字符串只能是偶数长度
        int n = s.length();
        if (n % 2 == 1) {
            return false;
        }

        // 括号必须结对，且要按顺序结对才有效（"([)]"是无效的）
        // 可以用栈来模拟结对消除，遍历结束后栈中为空则字符串有效
        Map<Character, Character> pairs = new HashMap<Character, Character>() {{
            put(')', '(');
            put(']', '[');
            put('}', '{');
        }};
        Deque<Character> stack = new LinkedList<Character>();
        for (int i = 0; i < n; i++) {
            char ch = s.charAt(i);
            if (pairs.containsKey(ch)) {
                if (stack.isEmpty() || stack.peek() != pairs.get(ch)) {
                    return false;
                }
                stack.pop();
            } else {
                stack.push(ch);
            }
        }
        return stack.isEmpty();
    }
}
```

上面代码在创建HashMap时，采用了匿名类的写法来快速初始化容器。第一对大括号表明这是一个继承自HashMap的匿名类，第二对大括号表明这是一个构造代码块。

* [各种代码块的初始化顺序](http://localhost:3000/#/all/basic_05_关键字?id=%e5%88%9d%e5%a7%8b%e5%8c%96%e9%a1%ba%e5%ba%8f)

<!--
## 301. 删除无效的括号

题目：https://leetcode-cn.com/problems/remove-invalid-parentheses/
-->

# 队列

队列用来处理先入先出的场景问题，在增删元素比较频繁的场景，同样可以用ArrayDeque来实现队列。

## 232. 用栈实现队列

题目：https://leetcode-cn.com/problems/implement-queue-using-stacks/

### 常规思路

用两个栈来实现队列的四个操作：队尾入队，队头出队，查看队头，是否为空。

通常思路是利用一个栈来存储元素，另一个栈作为临时容器以反转存储元素的顺序。在push操作时，如果s1栈中元素为空，则记录入栈的元素为队头元素。在入栈之前，先将s1栈中元素搬到另一个栈s2，然后新元素入栈到s2中。接着把栈s2的元素搬到栈s1中，这样**栈s1中的元素就跟队列先入先出一样的存储顺序了**。

此时，由于`push()`操作需要调整元素的存储顺序，时间复杂度为O(n)；另外三个操作都跟栈原本操作一样，时间复杂度是O(1)。

```java
class MyQueue {

    Integer front;
    ArrayDeque<Integer> s1 = new ArrayDeque<>();
    ArrayDeque<Integer> s2 = new ArrayDeque<>();

    public MyQueue() {

    }

    public void push(final int x) {
        if (s1.isEmpty()) {
            front = x;
        }
        while (!s1.isEmpty()) {
            s2.push(s1.pop());
        }
        s2.push(x);
        while (!s2.isEmpty()) {
            s1.push(s2.pop());
        }
    }

    public int pop() {
        return s1.pop();
    }

    public int peek() {
        return s1.peek();
    }

    public boolean empty() {
        return s1.isEmpty();
    }
}
```

### 均摊每个操作的时间复杂度

本题有个进阶要求，均摊每个操作的时间复杂度为O(1)。因此，可以将上述思路中的最耗时间的`push()`操作的时间复杂度，分摊到`pop()`中，即同时使用两个栈来存储元素。经过摊还时间复杂度，每个操作都为O(1)。

push的操作中，不再借助另一个栈来调整元素的存储顺序，而是直接将元素入栈s1。如果s1栈中元素为空，则记录入栈的元素为队头元素front。

在pop操作时，如果栈s2为空，则将元素从栈s1中搬到栈s2中来，此时最顶部的元素是最早入栈的元素，即队头元素，此时直接将栈s2的元素出栈即可。

在peek操作时，如果栈s2不为空，则栈顶部的元素是队头元素；否则返回之前记录的front元素。

在empty操作时，需要同时判断两个栈中元素是否为空。

```java
class MyQueue {

    Integer front;
    ArrayDeque<Integer> s1 = new ArrayDeque<>();
    ArrayDeque<Integer> s2 = new ArrayDeque<>();

    public MyQueue() {

    }

    public void push(final int x) {
        if (s1.isEmpty()) {
            front = x;
        }
        s1.push(x);
    }

    public int pop() {
        if (s2.isEmpty()) {
            while (!s1.isEmpty()) {
                s2.push(s1.pop());
            }
        }

        return s2.pop();
    }

    public int peek() {
        if (!s2.isEmpty()) {
            return s2.peek();
        }

        return front;
    }

    public boolean empty() {

        return s1.isEmpty() && s2.isEmpty();
    }
}
```

## 参考链接

* [用栈实现队列官方题解](https://leetcode-cn.com/problems/implement-queue-using-stacks/solution/yong-zhan-shi-xian-dui-lie-by-leetcode/)