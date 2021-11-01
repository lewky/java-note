<!--
date: 2021-10-28T11:34:12+08:00
lastmod: 2021-11-02T11:34:12+08:00
-->

## 排列组合

排列（Arrangement）：从n个不同元素中，任取m(m≤n,m与n均为自然数,下同）个不同的元素按照一定的顺序排成一列，叫做从n个不同元素中取出m个元素的一个排列；从n个不同元素中取出m(m≤n）个元素的所有排列的个数，叫做从n个不同元素中取出m个元素的排列数，用符号`A(n, m)`表示。

组合（Combination）：从n个不同元素中，任取m(m≤n）个元素并成一组，叫做从n个不同元素中取出m个元素的一个组合；从n个不同元素中取出m(m≤n）个元素的所有组合的个数，叫做从n个不同元素中取出m个元素的组合数。用符号`C(n,m)`表示。

排列计算公式：`A(n, m) = n(n-1)(n-2)···(n-m+1) = n! / (n-m)!`

组合计算公式：`C(n, m) = A(n, m) / n! = n! / (m!(n-m)!)`，`C(n, m) = C(n, n-m)`，其中n >= m

具体可以看：[百度百科·排列组合](https://baike.baidu.com/item/%E6%8E%92%E5%88%97%E7%BB%84%E5%90%88/706498#2_1)

## 回溯法

对于求排列组合的题目类型，通常使用**回溯法**来解决。回溯法通常用最简单的递归方法 + 深度优先遍历（Depth First Search，DFS）来实现，用来搜索一个问题的所有的解（可能找不到解，即题目没有解），因此回溯法也叫爆搜（暴力解法），其时间复杂度很高。

关于回溯法的入门可以看看这篇文章：[回溯算法入门级详解 + 练习（持续更新）](https://leetcode-cn.com/problems/permutations/solution/hui-su-suan-fa-python-dai-ma-java-dai-ma-by-liweiw/)

## 剪枝

回溯算法需要搜索所有的解，本身时间复杂度很高，如果在遍历时能提前知道当前分支无法搜索到需要的结果，就可以提前结束，这就是剪枝。有时候剪枝需要进行一些预处理工作，比如排序等，预处理同样需要消耗时间，但可以减少剪枝的耗时。

## 为什么不使用广度优先遍历

1）广度优先遍历（Breadth First Search，BFS），通常用队列来实现，可以用来找到最优解；而深度优先遍历使用栈来实现，只能找到是否有解。<br>
2）广度优先遍历虽然不需要回溯，但在遍历每一步时需要存储大量的状态变量，而真正能用上的比较少，从性能来看并不划算。<br>
3）深度优先遍历每一步的状态变化较少，因此在回溯时很容易。此外在递归调用方法时本身就是在一个方法栈中，因此无需额外创建栈来完成深度优先遍历。

## 46. 全排列

题目：https://leetcode-cn.com/problems/permutations/

数组元素是不重复的，因此可以不需要做任何剪枝，是最简单的回溯 + 深度优先遍历题目。

为了在深度遍历后能够回溯到上一层，需要记录每一层遍历时的当前深度；此外，还需要一个boolean数组记录当前使用了哪些数字参与排列；每次深度遍历得到的结果解也要一个参数来记录，最终在用深度遍历进行递归时，一共需要5个参数（并不是说深度优先遍历一定都是需要记录5个参数，只是当前这种思路需要5个参数）。

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

题目：https://leetcode-cn.com/problems/permutations-ii/

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

## 869. 重新排序得到 2 的幂

题目：https://leetcode-cn.com/problems/reordered-power-of-2/

这道题实际上就是 [231. 2 的幂](https://javanote.doc.lewky.cn/#/all/algorithm_04_位运算?id=_231-2-%e7%9a%84%e5%b9%82) + [47. 全排列 II](https://javanote.doc.lewky.cn/#/all/algorithm_05_排列组合?id=_47-%e5%85%a8%e6%8e%92%e5%88%97-ii) 的组合，题意要求重排后的数字不能有前导0，即0不能在最前面。

### 回溯法

官方题解采用的是回溯法，先将输入的正整数转化为字符数组，然后对其进行排序，便于后续在回溯过程中去除前导0和剪枝。`nums[i] - '0'`可以快速得到对应的整型值，比如`'0' - '0' = 0`,`'1' - '0' = 1`。

```java
class Solution {
    boolean[] vis;

    public boolean reorderedPowerOf2(int n) {
        char[] nums = Integer.toString(n).toCharArray();
        Arrays.sort(nums);
        vis = new boolean[nums.length];
        return backtrack(nums, 0, 0);
    }

    public boolean backtrack(char[] nums, int idx, int num) {
        if (idx == nums.length) {
            return isPowerOfTwo(num);
        }
        for (int i = 0; i < nums.length; ++i) {
            // 不能有前导零
            if ((num == 0 && nums[i] == '0') || vis[i] || (i > 0 && !vis[i - 1] && nums[i] == nums[i - 1])) {
                continue;
            }
            vis[i] = true;
            if (backtrack(nums, idx + 1, num * 10 + nums[i] - '0')) {
                return true;
            }
            vis[i] = false;
        }
        return false;
    }

    public boolean isPowerOfTwo(int n) {
        return (n & (n - 1)) == 0;
    }
}
```

### 打表 + 词频统计

回溯法的时间复杂度较高，由于输入的n是一个正整数，范围在int之内，因此最多只会有31个2的幂。可以提前将这些幂存入HashSet中，然后统计输入的正整数中每个数字出现的频次，存放到一个数组中。将该数组和所有的2的幂的词频对比，只要词频一致，必然能通过重排列变成2的幂。

这种做法的好处是用空间换时间，大大降低了时间复杂度，同时也不存在前导0的问题。

```java
class Solution {

    static HashSet<Integer> set = new HashSet<>();
    static {
        for (int i = 0; i < 31; i++) {
            set.add(1 << i);
        }
    }
    
    public boolean reorderedPowerOf2(int n) {
        if (set.contains(n)) {
            return true;
        } else if (n < 10) {
            return false;
        }
        
        // 统计词频
        int[] count = new int[10];
        char[] nums = Integer.toString(n).toCharArray();
        for (char c : nums) {
            count[c - '0']++;
        }
        
        for (int target : set) {
            char[] targetNums = Integer.toString(target).toCharArray();
            int[] temp = new int[10];
            for (char c : targetNums) {
                temp[c - '0']++;
            }
            if (checkArray(count, temp)) {
                return true;
            }
        }
        
        return false;
    }
    
    private boolean checkArray(int[] count, int[] temp) {
        for (int i = 0; i < count.length; i++) {
            if (count[i] != temp[i]) {
                return false;
            }
        }
        
        return true;
    }
    
}
```


## 77. 组合

题目：https://leetcode-cn.com/problems/combinations/

### 回溯法

和全排列属于一个类型的题目，本题参与组合的元素不会重复，深度优先遍历时会简单一些。

在每次遍历时，只需要记录5个参数：要遍历的数组，遍历数组时的起始下标，遍历的结果解，最终要返回的结果解，需要参与组合的数字个数。

```java
class Solution {
    public List<List<Integer>> combine(final int n, final int k) {
        final List<List<Integer>> res = new ArrayList<List<Integer>>();
        final int[] nums = new int[n];
        for (int i = 0; i < n; i++) {
            nums[i] = i + 1;
        }

        dfs(0, nums, new ArrayList<>(), res, k);

        return res;
    }

    private void dfs(final int index, final int[] nums, final List<Integer> subRes, final List<List<Integer>> res, final int k) {
        if (subRes.size() == k) {
            res.add(new ArrayList<>(subRes));
            return;
        }

        for (int i = index; i < nums.length; i++) {
            subRes.add(nums[i]);
            dfs(i + 1, nums, subRes, res, k);
            subRes.remove(subRes.size() - 1);
        }
    }
}
```

这里为了写得可以更通用些，所以先构造了一个数组来存储所有用于遍历的数字。这样就算题目改成输入一个元素不重复的数组，以及参与组合的数字个数，也可以不需要修改代码直接使用。

如果就本题而言，可以不用构造数组来遍历，这样会省空间。

### 回溯法 + 剪枝

上诉解法没有使用剪枝技巧，因此会老老实实遍历完所有的情况，这样时间复杂度较高。

实际上分析这道题，从n个数字中取出k个组成组合，这意味着在遍历时，要满足一个条件：被遍历的数组需要有k个数字。也就是说，如果在每次遍历的过程中，如果我们知道剩下的还未遍历的数字，不足以和当前的结果解构成k个数字，那也就没有继续遍历下去的必要了。

比如说数组中一共有5个不同的数字，需要取出其中3个构成组合。如果某次遍历的下标从第四个数字开始，当前的结果解是`[4]`，还需要两个数字构成组合，但继续深度遍历下去，也只有第五个数字还未遍历，因此当前遍历分支可以减去。

在未剪枝的版本中，遍历数字的条件是`i < nums.length`，现在进行剪枝，给i设置一个新的上界：`i <= nums.length - (k - subRes.size())`，以此减少遍历的深度。

```java
class Solution {
    public List<List<Integer>> combine(final int n, final int k) {
        final List<List<Integer>> res = new ArrayList<List<Integer>>();
        final int[] nums = new int[n];
        for (int i = 0; i < n; i++) {
            nums[i] = i + 1;
        }

        dfs(0, nums, new ArrayList<>(), res, k);

        return res;
    }

    private void dfs(final int index, final int[] nums, final List<Integer> subRes, final List<List<Integer>> res, final int k) {
        if (subRes.size() == k) {
            res.add(new ArrayList<>(subRes));
            return;
        }

        for (int i = index; i <= nums.length - (k - subRes.size()); i++) {
            subRes.add(nums[i]);
            dfs(i + 1, nums, subRes, res, k);
            subRes.remove(subRes.size() - 1);
        }
    }
}
```

## 参考链接

* [百度百科·排列组合](https://baike.baidu.com/item/%E6%8E%92%E5%88%97%E7%BB%84%E5%90%88/706498#2_1)
* [回溯算法入门级详解 + 练习（持续更新）](https://leetcode-cn.com/problems/permutations/solution/hui-su-suan-fa-python-dai-ma-java-dai-ma-by-liweiw/)
* [77. 组合 · 题解 · 回溯算法 + 剪枝（Java）](https://leetcode-cn.com/problems/combinations/solution/hui-su-suan-fa-jian-zhi-python-dai-ma-java-dai-ma-/)