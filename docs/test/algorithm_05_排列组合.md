修改：

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

* [77. 组合 · 题解 · 回溯算法 + 剪枝（Java）](https://leetcode-cn.com/problems/combinations/solution/hui-su-suan-fa-jian-zhi-python-dai-ma-java-dai-ma-/)
