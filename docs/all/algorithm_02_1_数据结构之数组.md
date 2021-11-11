<!--
date: 2021-11-09T10:34:12+08:00
lastmod: 2021-11-11T10:34:12+08:00
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
