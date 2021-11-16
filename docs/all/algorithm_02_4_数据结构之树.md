<!--
date: 2021-11-09T10:34:12+08:00
lastmod: 2021-11-13T10:34:12+08:00
-->

## 374. 猜数字大小

题目：https://leetcode-cn.com/problems/guess-number-higher-or-lower/

虽然没用到树，但可以用二分查找来解决，可以看成是查找一颗二叉树的结点：

```java
public class Solution extends GuessGame {
    public int guessNumber(int n) {
        int left = 1;
        int right = n;
        while (left < right) {
            // 求中间数时要防止溢出，也可以选择用long类型
            int mid = left + (right - left) / 2;
            int flag = guess(mid);
            if (flag == 0) {
                return mid;
            } else if (flag < 0) {
                right = mid - 1;
            } else {
                left = mid + 1;
            }

        }

        return left;
    }
}
```