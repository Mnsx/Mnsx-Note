## [剑指 Offer 20. 表示数值的字符串](https://leetcode.cn/problems/biao-shi-shu-zhi-de-zi-fu-chuan-lcof/)

> **请实现一个函数用来判断字符串是否表示数值（包括整数和小数）。**
>
> **数值**（按顺序）可以分成以下几个部分：
>
> * 若干空格
> * 一个小数或者整数
> * （可选）一个'e'或'E'，后面跟着一个整数
> * 若干空格
>
> **小数**（按顺序）可以分成以下几个部分：
>
> * （可选）一个符号字符（'+' 或 '-'）
> * 下述格式之一：
>   * 至少一位数字，后面跟着一个点 '.'
>   * 至少一位数字，后面跟着一个点 '.' ，后面再跟着至少一位数字
>   * 一个点 '.' ，后面跟着至少一位数字
>
> **整数**（按顺序）可以分成以下几个部分：
>
> * （可选）一个符号字符（'+' 或 '-'）
> * 至少一位数字
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

## [剑指 Offer 67. 把字符串转换成整数](https://leetcode.cn/problems/ba-zi-fu-chuan-zhuan-huan-cheng-zheng-shu-lcof/)

> **写一个函数 StrToInt，实现把字符串转换成整数这个功能。**
>
> 首先，该函数会**丢弃无用的开头空格字符**，直到寻找到第一个非空格的字符为止。
>
> 当我们寻找到的第一个非空**字符为正或者负号时，作为该整数的正负号**；假如第一个非空字符是**数字，则与之后连续的数字字符组合**，形成整数；该字符串中**多余的字符，忽略**
>
> 注意：字符串中的**第一个非空格字符不是一个有效整数字符**、**字符串为空**或**字符串仅包含空白字符**时，则不需要进行转换。
>
> 若函数**不能转换时，返回 0**。
>
> **提示**
>
> 数值范围为 [−231, 231 − 1]，超过这个范围，返回  INT_MAX (231 − 1) 或 INT_MIN (−231) 。

**题目标签**

`字符串` `自动机`

**解题思路**

字符串相关题目涉及复杂的路程和条件，可以使用自动机这个概念来处理每个输入字符的情况

自动机设定程序每个时刻都有一个状态，每次有新的字符输入就会修改状态

根据题目需求设计状态转换的Hash表，不同的字符输入时转换状态

题目可能存在的状况，空格，数值，正负号，多余字符

start初始状态——

* 输入空格，继续start状态，处理所有开头的空格
* 输入数值，进入number状态，处理数值
* 输入正负号，进入sign状态，处理正负号
* 输入多余字符，进入end状态，不允许有其他字符

number数值状态——

* 输入空格，进入end状态，空格属于多余字符
* 输入数值，继续number状态，处理后续的数值
* 输入正负号，进入end状态，正负号属于多余字符
* 输入多余字符，进入end状态，不允许有其他字符

sign正负号状态——

* 输入空格，进入end状态，空格属于多余字符
* 输入数值，进入number状态，处理数值
* 输入正负号，进入end状态，正负号只能存在一个
* 输入多余字符，进入end状态，不允许有其他字符

end结束状态——

* end状态输入任何字符都继续end状态

**代码展示**

```java
class Solution {
    public static void main(String[] args) {
        Solution solution = new Solution();
        System.out.println(solution.strToInt("42"));
    }

    public int strToInt(String str) {
        Automaton automaton = new Automaton();
        for (int i = 0; i < str.length(); ++i) {
            automaton.get(str.charAt(i));
        }
        return (int) (automaton.sign == 1 ? automaton.res : -automaton.res);
    }
}

class Automaton {
    // 结果符号
    int sign = 1;
    // 结果值
    long res = 0;
    // 当前状态，初始为start
    String state = "start";

    // 自动机的状态选项
    Map<String, String[]> map = new HashMap<>();
    {
        map.put("start", new String[]{"start", "sign", "number", "end"});
        map.put("sign", new String[]{"end", "end", "number", "end"});
        map.put("number", new String[]{"end", "end", "number", "end"});
        map.put("end", new String[]{"end", "end", "end", "end"});
    }

    /**
     * 根据输入的字符，判断自动机数据修改
     * @param ch 输入字符
     */
    public void get(char ch) {
        // 根据状态获取新状态
        state = map.get(state)[getState(ch)];
        if (state.equals("number")) {
            res = res * 10 + ch - '0';
            res = sign == 1 ? Math.min(res, Integer.MAX_VALUE) : Math.min(res, -(long)Integer.MIN_VALUE);
        } else if (state.equals("sign")) {
            sign = ch == '+' ? 1 : -1;
        }
    }

    /**
     * 根据输入字符获取状态
     * @param ch 输入字符
     * @return int
     */
    public int getState(char ch) {
        // 空格返回1
        if (ch == ' ') {
            return 0;
        }
        // 正负号返回1
        else if (ch == '+' || ch == '-') {
            return 1;
        }
        // 数字返回2
        else if (Character.isDigit(ch)) {
            return 2;
        }
        // 其他字符返回3
        else {
            return 3;
        }
    }
}
```

## [剑指 Offer 24. 反转链表](https://leetcode.cn/problems/fan-zhuan-lian-biao-lcof/)

> **定义一个函数，输入一个链表的头节点，反转该链表并输出反转后链表的头节点。**

**题目标签**

`链表` `递归`

**解题思路**

* **遍历**

  定义一个新的链表，遍历每次将新的元素，作为新链表的头部即可（见代码）

* **递归**

  递归到最后一个元素，将其作为头部，作为新链表返回，每次返回添加新的元素即可（见代码）

**代码展示**

* **遍历**

```java
class Solution {
    public ListNode reverseList(ListNode head) {
        // 新链表头节点
        ListNode pre = null;
        // 当前节点
        ListNode cur = head;
        // 反转时，头节点的后继节点会丢失，使用临时节点存储
        ListNode next;
        while (cur != null) {
            // 临时节点存放，当前节点的后继节点
            next = cur.next;
            // 当前节点作为新链表头节点加入
            cur.next = pre;
            // 当前节点作为新链表头节点
            pre = cur;
            // 继续遍历
            cur = next;
        }
        return pre;
    }
}
```

* **递归**

```java
class Solution {
    public ListNode reverseList(ListNode head) {
        // 排除链表为空，一直遍历到最后一个元素停止
        if (head == null || head.next == null) {
            return head;
        }
        // 将返回值作为新的链表
        ListNode newList = reverseList(head.next);
        // head-newList-head
        head.next.next = head;
        // newList-head
        head.next = null;
        return newList;
    }
}
```

## [2681. 英雄的力量](https://leetcode.cn/problems/power-of-heroes/)

> 给你一个下标从 0 开始的整数数组 nums ，它表示英雄的能力值。如果我们选出一部分英雄，这组英雄的 力量 定义为：
>
> * i0, i1, ... ik 表示这组英雄在数组中的下标，那么这组英雄的力量为`max(nums[i0], nums[i1], ... nums[ik])^2 * min(nums[i0], nums[i1], ... nums[ik])`
>
> **请你返回所有可能的非空英雄组的力量之和**
>
> **提示**
>
> 由于答案可能非常大，请你将结果对10^9 + 7 取余

**题目标签**

`动态规划` `前缀和`

**解题思路**



**代码展示**

```java
class Solution {
    public int sumOfPower(int[] nums) {
        // 排序，保证nums[i]为最大
        Arrays.sort(nums);
        int n = nums.length;
        // dp保存的是以nums[i]为结尾的序列的最小值
        int[] dp = new int[n];
        // dp[1].....dp[i - 1]相加的值
        int[] preSum = new int[n + 1];
        int res = 0;
        int mod = (int) (1e9 + 7);
        // 遍历，模拟以nums[i]为结尾的序列
        for (int i = 0; i < n; ++i) {
            // 见解题思路
            dp[i] = (preSum[i] + nums[i]) % mod;
            // preSum[i + 1]代表前i个的dp相加
            preSum[i + 1] = (preSum[i] + dp[i]) % mod;
            // 因为有序，所以nums[i]就是最大值，dp是最小值代入方程
            res = (int) ((res + (long) nums[i] * nums[i] % mod * dp[i]) % mod);
            // 因为强转int，可能超出最大值
            if (res < 0) {
                res += mod;
            }
        }
        return res;
    }
}
```



