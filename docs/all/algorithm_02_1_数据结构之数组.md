<!--
date: 2021-11-09T10:34:12+08:00
lastmod: 2021-11-22T10:34:12+08:00
-->

## 1. 两数之和

题目：https://leetcode-cn.com/problems/two-sum/

使用哈希表：

```java
class Solution {
    public int[] twoSum(final int[] nums, final int target) {
        HashMap<Integer, Integer> map = new HashMap<>();
        for (int i = 0; i < nums.length; i++) {
            if (map.get(target - nums[i]) != null) {
                return new int[] {map.get(target - nums[i]), i};
            } else {
                map.put(nums[i], i);
            }
        }

        return new int[0];
    }
}
```

## 88. 合并两个有序数组

题目：https://leetcode-cn.com/problems/merge-sorted-array/

使用两个指针，分别指向两个数组的下标；同时开始逆序遍历两个数组，遇到更大的一个元素时，对应的数组下标就向前移动一次，直到遍历完两个数组即可。

使用逆序遍历可以直接改变第一个数组的元素，而不需要额外申请内存空间。

```java
class Solution {
    public void merge(int[] nums1, final int m, final int[] nums2, final int n) {
        if (m == 0 && n == 0) {
            nums1 = new int[0];
        }

        int p1 = m - 1;
        int p2 = n - 1;
        while (p1 >= 0 || p2 >= 0) {
            if (p1 >= 0 && p2 >= 0) {
                if (nums1[p1] < nums2[p2]) {
                    nums1[p1 + p2 + 1] = nums2[p2];
                    p2--;
                } else {
                    nums1[p1 + p2 + 1] = nums1[p1];
                    p1--;
                }
            } else if (p1 >= 0) {
                p1--;
            } else if (p2 >= 0) {
                nums1[p2] = nums2[p2];
                p2--;
            }
        }
    }
}
```

## 383. 赎金信

题目：https://leetcode-cn.com/problems/ransom-note/

先统计杂志中所有字符的出现次数，然后再遍历赎金信的字符，当杂志中对应的字符的次数不够使用时则返回false。

```java
class Solution {
    public boolean canConstruct(final String ransomNote, final String magazine) {
        // 字符串都是小写字母，因此用数组来替代哈希表以统计次数，可以节省内存
        // magazine.charAt(i) - 'a'相当于快速求hash，即字符对应的数组下标
        final int[] temp = new int[26];
        for (int i = 0; i < magazine.length(); i++) {
            ++temp[magazine.charAt(i) - 'a'];
        }

        for (int i = 0; i < ransomNote.length(); i++) {
            if (--temp[ransomNote.charAt(i) - 'a'] < 0) {
                return false;
            }
        }

        return true;
    }
}
```

## 36. 有效的数独

题目：https://leetcode-cn.com/problems/valid-sudoku/

用三个数组来模拟哈希表，记录数独中每个数字在行、列和九宫格出现的次数，如果重复出现则该数独无效。

使用`board[i][j] - '1'] > 1`来快速定位到数组下标，将对应的值加一，如果增大后的值超过1说明当前数字在当前的行/列/九宫格里重复出现。

使用`i / 3 * 3 + j / 3`来计算出当前的数字属于第几个九宫格。

```java
class Solution {
    public boolean isValidSudoku(char[][] board) {
		// 记录每行出现的数字，row
        int[][] x = new int[9][9];
		// 记录每列出现的数字，column
        int[][] y = new int[9][9];
		// 记录每个九宫格出现的数字，area
        int[][] z = new int[9][9];
        for (int i = 0; i < board.length; i++) {
            for (int j = 0; j < board[i].length; j++) {
                if (board[i][j] != '.' && (++x[i][board[i][j] - '1'] > 1 || ++y[j][board[i][j] - '1'] > 1 || ++z[i / 3 * 3 + j / 3][board[i][j] - '1'] > 1)) {
                   return false;
                }
            }
        }
        
        return true;
    }
}
```

## 594. 最长和谐子序列

题目：https://leetcode-cn.com/problems/longest-harmonious-subsequence/

### 滑动窗口

先将数组排序，然后使用begin变量记录连续相同元素序列的第一个下标，遍历数组过程中i为连续相同元素序列的末尾元素的下一个相邻元素下标。当二者之间相差为1时，则可以得到两个元素之间的长度。最后就是在遍历中记录最大的序列长度即可。

```java
class Solution {
    public int findLHS(int[] nums) {
        Arrays.sort(nums);
        int begin = 0, res = 0;
        for (int i = 0; i < nums.length; i++) {
            while (nums[i] - nums[begin] > 1) {
                begin++;
            }
            if (nums[i] - nums[begin] == 1) {
                res = Math.max(res, i - begin + 1);
            }
        }
        
        return res;
    }
}
```

### 哈希表计数

可以用哈希表来记录数组中每个元素的出现次数，然后遍历哈希表，获取x和x+1的组合的个数，即为对应的和谐子序列的长度。最后在遍历中记录最大的序列长度即可。

```java
class Solution {
    public int findLHS(int[] nums) {
        HashMap <Integer, Integer> cnt = new HashMap <>();
        int res = 0;
        for (int num : nums) {
            cnt.put(num, cnt.getOrDefault(num, 0) + 1);
        }
        for (int key : cnt.keySet()) {
            if (cnt.containsKey(key + 1)) {
                res = Math.max(res, cnt.get(key) + cnt.get(key + 1));
            }
        }
        return res;
    }
}
```

## 384. 打乱数组

题目：https://leetcode-cn.com/problems/shuffle-an-array/

### 洗牌算法

本题是洗牌算法的模板题，洗牌算法实际上就是原地打乱元素顺序。对于一个数组，在每次遍历时随机选取一个元素，将其与数组中尚未洗牌的第一个元素进行置换。

这样第一个元素就完成了洗牌，剩下的元素尚待洗牌。在下次遍历时，从尚未洗牌的元素中重新随机选取一个，将其与尚未洗牌的第一个元素（也就是原数组的第二个元素）进行置换。以此类推，直到遍历结束，整个数组完成洗牌。

```java
class Solution {
    private int[] nums;
    private int[] res;
    private Random random = new Random();

    public Solution(int[] nums) {
        this.nums = nums;
        res = new int[nums.length];
        System.arraycopy(nums, 0, res, 0, nums.length);
    }
    
    public int[] reset() {
        return nums;
    }
    
    public int[] shuffle() {
        for (int i = 0; i < res.length; i++) {
            int index = i + random.nextInt(res.length - i);
            int temp = res[i];
            res[i] = res[index];
            res[index] = temp;
        }
        
        return res;
    }
}
```