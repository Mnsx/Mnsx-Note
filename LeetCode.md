## [剑指 Offer 20. 表示数值的字符串](https://leetcode.cn/problems/biao-shi-shu-zhi-de-zi-fu-chuan-lcof/)

> 请实现一个函数用来判断字符串是否表示数值（包括整数和小数）。
>
> 数值（按顺序）可以分成以下几个部分：
>
> * 若干空格
> * 一个小数或者整数
> * （可选）一个'e'或'E'，后面跟着一个整数
> * 若干空格
>
> 小数（按顺序）可以分成以下几个部分：
>
> * （可选）一个符号字符（'+' 或 '-'）
> * 下述格式之一：
>   * 至少一位数字，后面跟着一个点 '.'
>   * 至少一位数字，后面跟着一个点 '.' ，后面再跟着至少一位数字
>   * 一个点 '.' ，后面跟着至少一位数字
>
> 整数（按顺序）可以分成以下几个部分：
>
> * （可选）一个符号字符（'+' 或 '-'）
> * 至少一位数字
>
> 部分数值列举如下：
>
> ["+100", "5e2", "-123", "3.1416", "-1E-16", "0123"]
> 部分非数值列举如下：
>
> ["12e", "1a3.14", "1.2.3", "+-5", "12e+5.4"]
>
> **提示**
>
> - 1 <= s.length <= 20
> - s 仅含英文字母（大写和小写），数字（0-9），加号 '+' ，减号 '-'，空格 ' ' 或者点 '.' 。

**题目标签**

`字符串` `模拟`

**解题思路**

遍历字符串，针对字符的不同可能进行操作

* 正负号
* 数值
* 科学计数法符号
* 小数点

整数处理分为，带符号整数和不带符号整数

小数点处理，小数点需要前有整数或后必须有无符号整数，小数只能出现一次

科学计数法处理，符号前必须有整数或小数，符号后必须有无符号整数，科学计数法符号只能出现一次

由此可以得出

* 小数点处理一定先于科学计数法符号处理
* 小数点前不一定需要整数，但是科学计数法符号前一定需要整数
* 小数点后需要进行无符号整数处理
* 科学计数法符号后需要进行整数处理

**代码展示**

```java
class Solution {
    private int currentIndex;

    public boolean isNumber(String s) {
        // 排除字符串前后的空格
        s = s.trim();
        // 处理空串
        if (s.length() == 0) {
            return false;
        }
        
       	// 整数处理
        boolean flag = findInteger(s);

        // 针对小数点处理
        if (currentIndex < s.length() && s.charAt(currentIndex) == '.') {
            currentIndex++;
            // 因为小数点前不一定需要整数，所以为或
            flag = findUnsignedInteger(s) || flag;
        }

        // 针对科学计数法符号处理
        if (currentIndex < s.length() && (s.charAt(currentIndex) == 'E' || s.charAt(currentIndex) == 'e')) {
            currentIndex++;
            // 因为科学计数法符号前异地那个需要整数，所以为与
            flag = flag && findInteger(s);
        }

        return flag && currentIndex == s.length();
    }

    /**
     * 搜查后续的整数
     * @param str 字符串
     * @return 是否找到整数
     */
    public boolean findInteger(String str) {
        int n = str.length();

        // 忽略数值的正负
        if (currentIndex < n && (str.charAt(currentIndex) == '+' || str.charAt(currentIndex) == '-')) {
            currentIndex++;
        }
        return findUnsignedInteger(str);
    }

    /**
     * 搜查后续的数值
     * @param str 字符串
     * @return 是否找到数值
     */
    public boolean findUnsignedInteger(String str) {
        int n = str.length();
        int beforeIndex = currentIndex;

        // 忽略数字
        while (currentIndex < n && Character.isDigit(str.charAt(currentIndex))) {
            currentIndex++;
        }

        return currentIndex > beforeIndex;
    }
}
```



