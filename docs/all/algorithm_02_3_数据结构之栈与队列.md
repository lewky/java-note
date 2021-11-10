<!--
date: 2021-11-09T10:34:12+08:00
lastmod: 2021-11-10T10:34:12+08:00
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