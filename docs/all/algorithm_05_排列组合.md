<!--
date: 2021-10-28T11:34:12+08:00
lastmod: 2021-10-28T11:34:12+08:00
-->

## 排列组合

排列（Arrangement）：从n个不同元素中，任取m(m≤n,m与n均为自然数,下同）个不同的元素按照一定的顺序排成一列，叫做从n个不同元素中取出m个元素的一个排列；从n个不同元素中取出m(m≤n）个元素的所有排列的个数，叫做从n个不同元素中取出m个元素的排列数，用符号`A(n, m)`表示。

组合（Combination）：从n个不同元素中，任取m(m≤n）个元素并成一组，叫做从n个不同元素中取出m个元素的一个组合；从n个不同元素中取出m(m≤n）个元素的所有组合的个数，叫做从n个不同元素中取出m个元素的组合数。用符号`C(n,m)`表示。

排列计算公式：`A(n, m) = n(n-1)(n-2)···(n-m+1) = n! / (n-m)!`

组合计算公式：`C(n, m) = A(n, m) / n! = n! / (m!(n-m)!)`，`C(n, m) = C(n, n-m)`，其中n >= m

具体可以看：[百度百科·排列组合](https://baike.baidu.com/item/%E6%8E%92%E5%88%97%E7%BB%84%E5%90%88/706498#2_1)

## 回溯法

对于求排列组合的题目类型，通常使用**回溯法**来解决。回溯法通常用最简单的递归方法 + 深度优先遍历（Depth-First-Search，DFS）来实现，用来搜索一个问题的所有的解（可能找不到解，即题目没有解），因此回溯法也叫爆搜（暴力解法），其时间复杂度很高。

关于回溯法的入门可以看看这篇文章：[回溯算法入门级详解 + 练习（持续更新）](https://leetcode-cn.com/problems/permutations/solution/hui-su-suan-fa-python-dai-ma-java-dai-ma-by-liweiw/)

## 剪枝

## 为什么不使用广度优先遍历

## 46. 全排列

题目：https://leetcode-cn.com/problems/permutations/

数组元素是不重复的，因此可以不需要做任何剪枝，是最简单的回溯 + 深度优先遍历题目。

为了在深度遍历后能够回溯到上一层，需要记录每一层遍历时的当前深度；此外，还需要一个boolean数组记录当前使用了哪个数字参与排列；每次深度遍历的解也要一个参数来记录，最终在用深度遍历进行递归时，一共需要5个参数。

```java
class Solution {
    public List<List<Integer>> permute(final int[] nums) {
        final List<List<Integer>> res = new ArrayList<>();
        final List<Integer> subRes = new ArrayList<>();
        if (nums.length < 1) {
            return res;
        }
        final boolean[] flag = new boolean[nums.length];
        dfs(nums, 0, flag, res, subRes);

        return res;
    }

    private void dfs(final int[] nums, final int depth, final boolean[] flag, final List<List<Integer>> res, final List<Integer> subRes) {
        if (depth == nums.length) {
            // 这里不能直接把遍历结果添加到集合中，因为一次遍历结果是一个对象，在回溯时会被清空
            // 这里需要拷贝一个新的集合并添加到结果中
            res.add(new ArrayList<>(subRes));
            
            // 一次深度遍历结束，回溯到上一层
            return;
        }

        for (int i = 0; i < nums.length; i++) {
            if (!flag[i]) {
                flag[i] = true;
                subRes.add(nums[i]);
                dfs(nums, depth + 1, flag, res, subRes);

                // 开始回溯，复原步骤
                flag[i] = false;
                subRes.remove(subRes.size() - 1);
            }
        }
    }
}
```

## 47. 全排列 II

本题是上一题（46. 全排列）的进阶版，输入的数组元素可以有重复的，因此本题在上一题的解法思路之上，还需要在遍历每一层时额外使用一个HashSet来进行去重操作：

```java
class Solution {
    public List<List<Integer>> permuteUnique(final int[] nums) {
        final List<List<Integer>> res = new ArrayList<>();
        final List<Integer> subRes = new ArrayList<>();
        if (nums.length < 1) {
            return res;
        }
        final boolean[] flag = new boolean[nums.length];
        dfs(nums, 0, flag, res, subRes);

        return res;
    }

    private void dfs(final int[] nums, final int depth, final boolean[] flag, final List<List<Integer>> res, final List<Integer> subRes) {
        if (depth == nums.length) {
            res.add(new ArrayList<>(subRes));
            // 一次深度遍历结束，回溯到上一层
            return;
        }

        // 在每层遍历时要记录当前使用了哪些数字，用以去重
        final HashSet<Integer> set = new HashSet<>();
        for (int i = 0; i < nums.length; i++) {
            if (!flag[i] && !set.contains(nums[i])) {
                    flag[i] = true;
                    subRes.add(nums[i]);

                    set.add(nums[i]);
                    dfs(nums, depth + 1, flag, res, subRes);

                    // 开始回溯，复原步骤
                    flag[i] = false;
                    subRes.remove(subRes.size() - 1);
            }
        }
    }
}
```


