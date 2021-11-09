<!--
date: 2021-10-26T08:34:12+08:00
lastmod: 2021-10-30T08:34:12+08:00
-->

## 动态规划Dynamic Programming

动态规划属于分治法的一种，分治法是自顶向下的，动态规划与之相反，是自底向上的。动态规划最重要的就是**找到状态转移方程**（类似于归纳法），这也是最难的地方。

自顶向下的分治法往往采用递归的写法，自底向上的动态规划则是采用迭代的写法。一个递归实现的分治法，有的可以写成一个带有备忘录的迭代实现的动态规划。

为了提高性能，降低时间损耗，往往会使用数组作为备忘录，以较小的空间存储之前的状态。入门可以看这篇文章：[动态规划解题核心框架](https://labuladong.gitee.io/algo/3/21/61/)

## 121. 买卖股票的最佳时机

题目：https://leetcode-cn.com/problems/best-time-to-buy-and-sell-stock/

### 动态规划解法

求某段时间内的最高利润，如果将每天的股价数组转为每天的价格波动（即利润数组），则可以转化为一个动态规划问题，即求一个数字序列的连续子序列最大的和。

状态转移方程：以第i个数字结尾的连续子序列的最大和，等于第i-1个数字结尾的最大和加上第i个数字的和，或者等于第i个数字。这取决于第i-1个数字结尾的最大和是否大于0。

```java
class Solution {
    public int maxProfit(final int[] prices) {
        final int[] dp = new int[prices.length];
        int max = 0;
        for (int i = 1; i < prices.length; i++) {
            int count = prices[i] - prices[i - 1];
            if (dp[i - 1] > 0) {
                dp[i] = dp[i - 1] + count;
            } else {
                dp[i] = count;
            }
            
            if (dp[i] > max) {
                max = dp[i];
            }
        }

        return max;
    }
}
```

### 求差法

对于本题而言，用动态规划法实际上不是最佳的方法。这是买卖股票，那么对于第i天而言，如果想利润最高，必然要在第1~i天期间的最低价买入，这样在第i天卖出才能获取最高利润。

于是就有了下面更快的做法：直接算出第1~i天期间的最低价，并算出最大的利润。

```java
class Solution {
    public int maxProfit(final int[] prices) {
        int minPrice = Integer.MAX_VALUE;
        int max = 0;
        for (int i : prices) {
            if (i < minPrice) {
                minPrice = i;
            } else if (i - minPrice > max) {
                max = i - minPrice;
            }
        }

        return max;
    }
}
```