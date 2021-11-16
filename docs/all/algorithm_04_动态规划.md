<!--
date: 2021-10-26T08:34:12+08:00
lastmod: 2021-11-17T08:34:12+08:00
-->

## 动态规划Dynamic Programming

动态规划属于分治法的一种，分治法是自顶向下的，动态规划与之相反，是自底向上的。动态规划最重要的就是**找到状态转移方程**（类似于归纳法），这也是最难的地方。

自顶向下的分治法往往采用递归的写法，自底向上的动态规划则是采用迭代的写法。一个递归实现的分治法，有的可以写成一个带有备忘录的迭代实现的动态规划。

为了提高性能，降低时间损耗，往往会使用数组作为备忘录，以较小的空间存储之前的状态。入门可以看这篇文章：[动态规划解题核心框架](https://labuladong.gitee.io/algo/3/21/61/)

## 121. 买卖股票的最佳时机

题目：https://leetcode-cn.com/problems/best-time-to-buy-and-sell-stock/

### 动态规划法

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

## 118. 杨辉三角

题目：https://leetcode-cn.com/problems/pascals-triangle/

这道题的状态转移方程一目了然，跟斐波那契数列差不多，很轻松就可以用动态规划法做出来。

此外，可以发现杨辉三角的边缘都是1，且每一行的数字都是左右对称的，因此可以利用这个性质只计算前半截的数字，后半截可以直接获取到对称的值，这样可以提高时间效率。

```java
class Solution {
    public List<List<Integer>> generate(final int numRows) {
        final List<List<Integer>> res = new ArrayList<>();

        for (int i = 0; i < numRows; i++) {
            final List<Integer> subRes = new ArrayList<>();
            subRes.add(1);
            for (int j = 1; j < i + 1; j++) {
                if (j < (i + 2) / 2) {
                    if (i > 0) {
                        final int temp = res.get(i - 1).get(j) + res.get(i - 1).get(j - 1);
                        subRes.add(temp);
                    }
                } else {
                    final int temp = subRes.get(i - j);
                    subRes.add(temp);
                }
            }

            res.add(subRes);
        }

        return res;
    }
}
```

## 375. 猜数字大小 II

题目：https://leetcode-cn.com/problems/guess-number-higher-or-lower-ii/

### 极小化极大算法

Minimax算法（亦称 MinMax or MM）又名极小化极大算法，是一种找出失败的最大可能性中的最小值的算法。具体可以参考百度百科：[极小化极大算法](https://baike.baidu.com/item/%E6%9E%81%E5%B0%8F%E5%8C%96%E6%9E%81%E5%A4%A7%E7%AE%97%E6%B3%95/1351828?fr=aladdin)

本题可以用一个二维数组dp[i][j]来表示从i到j之间的可以确保获胜的最小现金数，在i和j之间猜数字k，k的范围在i和j之间，会有三种情况：

1）猜对数字，此时无需支付现金<br>
2）猜的数字小了，即k小于实际数字，此时需要缩小下一次猜测的范围，即i到k-1<br>
3）猜的数字大了，即k大于实际数字，此时需要扩大下一次猜测的范围，即k+1到j

为了确保获胜，需要考虑最差情况，也就是取上述三种情况中的最大值现金数，也就是取局部极大值。

然后在i和j之间猜数字，需要遍历所有数字，也就是逐个猜i和j之间的数字，来找到所有局部极大值中的最小值。综合起来，就是极小化极大算法。

由于是二维数组，对于不满足猜测范围的数字，可以认为是无需猜测就能得到答案，此时无需支付现金，也就是说值是0。

```java
class Solution {
    public int getMoneyAmount(final int n) {
        final int[][] dp = new int[n + 1][n + 1];
        // 逆序遍历是为了找dp值时，被依赖的dp值还未计算出来
        for (int i = n - 1; i >= 1; i--) {
            for (int j = i + 1; j <= n; j++) {
                // 为了避免k取j值时越界，先记录k为j值的dp值
                dp[i][j] = j + dp[i][j - 1];
                for (int k = i; k < j; k++) {
                    // 找到所有k值时的最小值
                    // dp值需要考虑最大值（即最差情况）
                    dp[i][j] = Math.min(dp[i][j], k + Math.max(dp[k + 1][j], dp[i][k - 1]));
                }

            }
        }

        return dp[1][n];
    }
}
```
