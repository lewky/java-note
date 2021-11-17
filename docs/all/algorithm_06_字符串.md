<!--
date: 2021-11-17T11:34:12+08:00
lastmod: 2021-11-17T11:34:12+08:00
-->

## 412. Fizz Buzz

题目：https://leetcode-cn.com/problems/fizz-buzz/

### 取模

常规思路是用取模操作来分类判断输出对应的字符串：

```java
class Solution {
    public List<String> fizzBuzz(int n) {
        List<String> list = new ArrayList<>(n);
        for (int i = 1; i <= n; i++) {
            if (i % 15 == 0) {
                list.add("FizzBuzz");
            } else if (i % 3 == 0) {
                list.add("Fizz");
            } else if (i % 5 == 0) {
                list.add("Buzz");
            } else {
                list.add(i + "");
            }
        }
        return list;
    }
}
```

### 加法运算取代取模操作

但是考虑到CPU取模运算相对来说效率很低，如果可以避免使用大量的取模操作，可以提升程序的性能。因此可以用下面的写法来巧妙地取代取模：

```java
class Solution {
    public List<String> fizzBuzz(int n) {
        List<String> result = new ArrayList<>();
        for(int i = 1, fizz = 3, buzz = 5; i <= n; i++) {
            if(i == fizz && i == buzz) {
                fizz += 3;
                buzz += 5;
                result.add("FizzBuzz");
            } else if(i == fizz) {
                fizz += 3;
                result.add("Fizz");
            } else if(i == buzz) {
                buzz += 5;
                result.add("Buzz");
            } else
                result.add(String.valueOf(i));
        }

        return result;
    }
}
```

## 709. 转换成小写字母

题目：https://leetcode-cn.com/problems/to-lower-case/

题目本身很简单，只是有两点需要注意：

1）为什么大小写字母的字符差距是32而不是26（一共26个字母）？<br>
2）字符和整型数字运算时要注意返回值

第一点很简单，在观察ASCII表中大小写字母的二进制时，会发现同一个字母的大小写字符，对应的八位二进制数字实际上只有一位是不一样的，其他位都是一样的。显然这是为了方便进行大小写字母之间的对照和转化，那为什么是32呢？

从二进制的角度来考虑，现在只有一位数字不同，那必然要大于26才行，于是就选择了最贴近的32；换言之，同一个字母的小写会比大写大上32，比如`a`字符的二进制是`0110 0001`，而`A`字符是`0100 0001`。因此只需要记住，**在ASCII表中出于书写习惯，大写字母先出现（即比小写字母小），然后小写字母比对应的大写要大32（即`0010 0000`）。**

第二点则是因为字符和整型运算时，字符会自动向上转型为整型，导致结果为整型，如果直接在拼接字符串时不注意这点，会导致输出的字符串是一个整型数字，而不是字符。

```java
class Solution {
    public String toLowerCase(String s) {
        StringBuilder sb = new StringBuilder();
        for (int i = 0; i < s.length(); i++) {
            char c = s.charAt(i);
            if (c >= 'A' && c <= 'Z') {
				// +=操作符自带类型转换
                c += 32;
            }
            sb.append(c);
        }
        return sb.toString();
    }
}
```