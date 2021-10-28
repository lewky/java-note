<!--
date: 2021-10-28T11:34:12+08:00
lastmod: 2021-10-28T11:34:12+08:00
-->

## 位运算

位运算可以用来巧妙的解决多种类型场景的问题，且时间复杂度和空间复杂度都很低，比如求两数交换、2的幂、反转字符串等。

这类题目通常会用到移位、异或等操作。

## 不占用任何额外空间的情况下交换两个数的值

题目：假如有x、y两个数，如何在不占用任何额外空间的情况下交换两个数的值？

平时我们在交换两个数的值时，往往会用一个中间数temp来实现效果，现在需要不占用任何额外空间，自然就不能使用这种寻常的方法了；这里可以有两种方法来实现。

### 方法一：求和

```java
int x = 5;
int y = 10;
x = x + y;
y = x - y;
x = x - y;
```

先将两个数之和附给x，接着x-y自然就是原本x的值，这时候赋值给y，y就拿到了x原本的值。此时x依然是两个数之和，再进行x-y自然就是原本x的值。

这种方法比较直观，也好理解，但是可能**存在溢出的情况。**

### 方法二：异或运算

```java
int x = 5;
int y = 10;
x = x ^ y;
y = x ^ y;
x = x ^ y;
```

第二种方法利用了异或运算的性质：

1）相同的两个数异或结果为0<br>
2）任何数与0异或结果还是其自身<br>
3）异或运算满足交换律和结合律

于是将x^y的结果赋予x，接着再将x与y异或，此时y的值就是`x^y^y = x^(y^y) = x`，也就是说y拿到了x原本的值。

此时x依然是两数异或的结果，而y是x原本的值，接着进行x^y就等同于`x^y^x = y`， 于是x就拿到了y原本的值。

这种方法很巧妙，也不太好理解，但是不存在溢出的情况。

## 231. 2 的幂

题目：https://leetcode-cn.com/problems/power-of-two/

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

此外，题目规定了n的大小，因此最多2的幂最大为`1 << 30`。因此也可以直接拿最大的这个幂来判断输入的数是否为其约数，若是则说明输入的数也是2的幂：

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
