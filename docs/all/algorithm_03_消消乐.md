<!--
date: 2021-10-27T22:34:12+08:00
lastmod: 2021-10-30T22:34:12+08:00
-->

## 消消乐

这类问题需要按顺序解除字符、或者数字等，通常用**后入先出的栈**来模拟解决这类消消乐问题。

Java的栈推荐使用ArrayDeque。

* [为什么使用ArrayDeque作为栈](https://javanote.doc.lewky.cn/#/all/algorithm_02_单调栈?id=%e4%b8%ba%e4%bb%80%e4%b9%88%e4%bd%bf%e7%94%a8arraydeque%e4%bd%9c%e4%b8%ba%e6%a0%88)

## 20. 有效的括号

题目：https://leetcode-cn.com/problems/valid-parentheses/

由题意可知，有效字符串只能是偶数长度，故奇数长度直接返回false。

括号类型必须按顺序结对才有效，换言之，将字符串中的括号按顺序结对消除，即左边的括号入栈，遇到对应的右边括号则出栈，最终若栈为空则说明是有效字符串。

```java
class Solution {
    public boolean isValid(String s) {
        // 有效字符串只能是偶数长度
        if (s.length() % 2 != 0) {
            return false;
        }

        // 括号必须结对，且要按顺序结对才有效（"([)]"是无效的）
        // 可以用栈来模拟结对消除，遍历结束后栈中为空则字符串有效
        Deque<Character> stack = new ArrayDeque<>();
        for (int i = 0; i < s.length(); i++) {
            char c = s.charAt(i);
            if (c == '(' || c == '[' || c == '{') {
                stack.push(c);
            } else if (c == ')') {
                if (stack.isEmpty() || stack.poll() != '(') {
                    return false;
                }
            } else if (c == ']') {
                if (stack.isEmpty() || stack.poll() != '[') {
                    return false;
                }
            } else if (c == '}') {
                if (stack.isEmpty() || stack.poll() != '{') {
                    return false;
                }
            }
        }

        return stack.isEmpty();
    }
}
```

<!--
## 301. 删除无效的括号

题目：https://leetcode-cn.com/problems/remove-invalid-parentheses/
-->