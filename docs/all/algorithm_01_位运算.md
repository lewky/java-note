<!--
date: 2021-10-28T11:34:12+08:00
lastmod: 2021-11-16T11:34:12+08:00
-->

## 位运算

位运算可以用来巧妙的解决多种类型场景的问题，且时间复杂度和空间复杂度都很低，比如求两数交换、2的幂、反转字符串等。

这类题目通常会用到移位、异或等操作。

## 异或运算的性质

1）相同的两个数异或结果为0<br>
2）任何数与0异或结果还是其自身<br>
3）异或运算满足交换律和结合律

## 位运算符优先级

在使用位运算时要注意优先级问题，有时候可能写漏了括号，会导致最后运算结果不同。

当然没必要刻意记住各种符合优先级，在遇到多种运算符混合使用的时候，加上小括号就足够了。

## 不申请额外空间来交换两个数的值

题目：假如有x、y两个数，如何在不占用任何额外空间的情况下交换两个数的值？

平时我们在交换两个数的值时，往往会用一个中间数temp来实现效果，现在需要不占用任何额外空间，自然就不能使用这种寻常的方法了；这里可以有两种方法来实现。

### 求和法

```java
int x = 5;
int y = 10;
x = x + y;
y = x - y;
x = x - y;
```

先将两个数之和附给x，接着x-y自然就是原本x的值，这时候赋值给y，y就拿到了x原本的值。此时x依然是两个数之和，再进行x-y自然就是原本x的值。

这种方法比较直观，也好理解，但是可能**存在溢出的情况。**

### 异或运算法

```java
int x = 5;
int y = 10;
x = x ^ y;
y = x ^ y;
x = x ^ y;
```

第二种方法利用了异或运算的性质，将x^y的结果赋予x，接着再将x与y异或，此时y的值就是`x^y^y = x^(y^y) = x`，也就是说y拿到了x原本的值。

此时x依然是两数异或的结果，而y是x原本的值，接着进行x^y就等同于`x^y^x = y`， 于是x就拿到了y原本的值。

这种方法很巧妙，也不太好理解，但是不存在溢出的情况。

## 231. 2 的幂

题目：https://leetcode-cn.com/problems/power-of-two/

### 位运算解法

2的幂是一个二进制最高位为1，之后的位全为0的数，且2的幂必定大于0。

观察2的幂的二进制形式后，可以发现，如果将这个数减去1，则其二进制原本的最高位变为0，后面的位会变成1。此时，将二者进行&运算会得到0。此外，若n是一个2的幂，还可以发现`(n & -n) == n`

基于上面的发现，有两种写法：

```java
class Solution {
    public boolean isPowerOfTwo(int n) {
        return n > 0 && (n & (n - 1)) == 0;
    }
}

class Solution {
    public boolean isPowerOfTwo(int n) {
        return n > 0 && (n & -n) == n;
    }
}
```

### 最大约数解法

题目规定了n的大小，因此最多2的幂最大为`1 << 30`。因此也可以直接拿最大的这个幂来判断输入的数是否为其约数，若是则说明输入的数也是2的幂：

```java
class Solution {
    static final int BIG = 1 << 30;

    public boolean isPowerOfTwo(int n) {
        return n > 0 && BIG % n == 0;
    }
}
```

## 344. 反转字符串

题目：https://leetcode-cn.com/problems/reverse-string/

### 异或运算

本题难点在于要求不能申请额外的内存空间，只能直接修改输入的数组，并且要求额外的空间为O(1)。在反转字符串后，会发现字符串的首尾字符互相交换了位置，也就是说，本题可以转换为上文的“交换两个数的值”。

于是就有了这样一种思路：将字符串的首尾字符通过异或运算来相互调换位置，比如第一个和最后一个字符互换位置，第二个和最后第二个互换位置，一共循环字符串长度/2的次数，这样就可以实现题目想要的效果。代码如下：

```java
class Solution {
    public void reverseString(char[] s) {
        int size = s.length;
        int a = size / 2;
        int last;
        for (int i = 0; i < a; i++) {
            last = size - 1 - i;
            s[i] ^= s[last];
            s[last] ^= s[i];
            s[i] ^= s[last];
        }
    }
}
```

### 双指针

官方题解的做法直接用一个字符来存储临时字符，用然后双指针来遍历：

```
class Solution {
    public void reverseString(char[] s) {
        int n = s.length;
        for (int left = 0, right = n - 1; left < right; ++left, --right) {
            char tmp = s[left];
            s[left] = s[right];
            s[right] = tmp;
        }
    }
}
```

## 136. 只出现一次的数字

题目：https://leetcode-cn.com/problems/single-number/

数组中其他数字都出现2次，只有一个数字出现一次，且解法应该**具有线性时间复杂度，不使用额外空间来实现**，这基本可以确定是使用异或运算来解题了：

```java
class Solution {
    public int singleNumber(int[] nums) {
        int res = 0;
        for (int n : nums) {
            res ^= n;
        }
        return res;
    }
}
```

## 137. 只出现一次的数字 II

题目：https://leetcode-cn.com/problems/single-number-ii/

和上一题（136. 只出现一次的数字）的区别在于，其他重复出现的数字是3次，此时无法使用异或运算来快速得出结果。

常规解法是使用哈希表，统计所有数字出现的次数，最终得到只出现一次的数字。对于题目要求的进阶解法，依然可以用位运算来解决。

### 统计二进制位上的和并取模

对于一个数字，其二进制数在每一位上只会是0或1。如果出现3次，那么每一位的数总和就是0或3。也就是说，如果出现3或者3的倍数次，这个总和必然可以被3整除，其余数为0。

将数组中所有数的每一位的值加起来，对3取模，剩下的值自然就是只出现一次的数字在那一位上的值：0或1。因此可以通过逐位统计每一位上的取模结果，并将每一位的结果加起来，就是最终只出现一次的数字。

```java
class Solution {
    public int singleNumber(int[] nums) {
        int res = 0;
        for (int i = 0; i < 32; i++) {
            int count = 0;
            for (int n : nums) {
                count += ((n >> i) & 1);
            }
            if (count % 3 == 1) {
                // 由于第i位以外的都是0，这里可以用 |= 替代 += , 其效率更高
                res |= (1 << i);
            }
        }

        return res;
    }
}
```

## 260. 只出现一次的数字 III

题目：https://leetcode-cn.com/problems/single-number-iii/

由于数组中存在两个只出现一次的数字，所以无法像 [136. 只出现一次的数字](https://javanote.doc.lewky.cn/#/all/algorithm_04_位运算?id=_136-%e5%8f%aa%e5%87%ba%e7%8e%b0%e4%b8%80%e6%ac%a1%e7%9a%84%e6%95%b0%e5%ad%97) 那样直接异或得出结果。常规解法依然是哈希表，如果要实现进阶要求，仅使线性时间复杂度和用常数空间复杂度来实现，依然离不开位运算。

### 异或 + 分组

首先将数组全部异或一遍，会得到两个目标数字的异或结果。由于两数不相等，这个异或结果不为0，也就是说这个异或结果的二进制值，必定至少有一位的值为1，我们只需要找到任意一位即可，假设找到的是第k位。两个目标数字在第k位上必定一个是0，一个是1。

利用这第k位，将数组中所有数字的第k位分为0和1两组，各自进行异或，就能得到两个目标数字。相当于利用第k位，将数组分成了两组，目标数字被分离到了不同组，而组内只有一个数字只出现一次，其他数字都是出现两次的，这样就又变成了 `136. 只出现一次的数字` 的场景。

```java
class Solution {
    public int[] singleNumber(int[] nums) {
        int temp = 0;
        for (int i : nums) {
            temp ^= i;
        }
        
        int k = 0;
        for (int i = 0; i < 32; i++) {
            if (((temp >> i) & 1) == 1) {
                k = i;
            }
        }
        
        int[] res = new int[2];
        for (int i : nums) {
            if (((i >> k) & 1) == 0) {
                res[0] ^= i;
            } else {
                res[1] ^= i;
            }
        }
        
        return res;
    }
}
```

## 520. 检测大写字母

题目：https://leetcode-cn.com/problems/detect-capital/

这道题有点脑筋急转弯的意思，如果单纯按照题意来写出各种if分支，必然会比较多分支。分析题意，可以知道若第一个字母是小写，那么第二个字母不能是大写；此时其他的情况可以统一处理：无论第一个字母是什么情况，只需要判断第二个字母和后续的字母是否大小写相同。

而保证一个字符串的字母是否全为大写或小写，可以用位运算来简单的判断。

```java
class Solution {
    public boolean detectCapitalUse(final String word) {
        if (word.length() <= 1) {
            return true;
        }

        // 如果第一个字母是小写，需判断第二个字母是否为小写
        final boolean secondCapital = word.charAt(1) <= 'Z';
        if (word.charAt(0) > 'Z' && secondCapital) {
            return false;
        }

        // 无论第 1 个字母是否大写，其他字母必须与第 2 个字母的大小写相同
        for (int i = 2; i < word.length(); i++) {
            if (word.charAt(i) <= 'Z' ^ secondCapital) {
                return false;
            }
        }

        return true;
    }
}
```
