# 整数

## 整数的基本知识

### 整数除法 [goto](https://leetcode.cn/problems/xoh6Oh/)

```java
class Solution {
    public int divide(int a, int b) {
        if (a == Integer.MIN_VALUE && b == -1) {
            return Integer.MAX_VALUE;
        }
        int flag = 2;
        if (a > 0) {
            flag--;
            a = -a;
        }
        if (b > 0) {
            flag--;
            b = -b;
        }
        
        int result = 0;
        while (a <= b) {
            int x = b;
            int y = 1;
            while (x >= 0xc0000000 && a <= x + x) {
                y += y;
                x += x;
            }
            result += y;
            a -= x;
        }

        return flag == 1 ? -result : result;
    }
}
```

![image-20221017220529869](D:\WorkSpace\Note\Picture\image-20221017220529869.png)

## 二级制

### 二进制加法 [goto](https://leetcode.cn/problems/JFETK5/)

```java
class Solution {
    public String addBinary(String a, String b) {
        int up = 0;
        int n = a.length() - 1;
        int m = b.length() - 1;

        StringBuffer sb = new StringBuffer();
		
        while (n >= 0 || m >= 0) {
            int x = (n >= 0 ? a.charAt(n--) - '0' : 0);
            int y = (m >= 0 ? b.charAt(m--) - '0' : 0);
            int sum = x + y + up;
            up = sum >= 2 ? 1 : 0;
            
            sb.append(sum >= 2 ? sum -= 2 : sum);
        }
		
        if (up == 1) {
            sb.append(up);
        }

        return sb.reverse().toString();
    }
}
```

![image-20221017220551021](D:\WorkSpace\Note\Picture\image-20221017220551021.png)

### 前n个数字二进制形式中1的个数 [goto](https://leetcode.cn/problems/w3tCBm/submissions/)

```java
class Solution {
    public int[] countBits(int n) {
        int[] result = new int[n + 1];
        for (int i = 1; i <= n; ++i) {
            result[i] = result[i >> 1] + (i & 1);
        }
        return result;
    }
}
```

![image-20221017220730963](D:\WorkSpace\Note\Picture\image-20221017220730963.png)

### 只出现一次的数字 [goto](https://leetcode.cn/problems/WGki4K/)

```java
class Solution {
    public int singleNumber(int[] nums) {
        int[] bit = new int[32];
        for (int num : nums) {
            for (int i = 0; i < 32; ++i) {
                bit[i] += ((num >> (31 - i)) & 1); 
            }
        }

        int result = 0;
        for (int i = 0; i < 32; ++i) {
            result = (result <<1) + (bit[i] % 3);
        }

        return result;
    }
}
```

![image-20221026094106186](D:\WorkSpace\Note\Picture\image-20221026094106186.png)

### 单词长度的最大乘积 [goto](https://leetcode.cn/problems/aseY1I/)

```java
class Solution {
    public int maxProduct(String[] words) {
        int[] arr = new int[words.length];
        for (int i = 0; i < words.length; ++i) {
            for (int j = 0; j < words[i].length(); ++j) {
                arr[i] |= 1 << (words[i].charAt(j) - 'a');
            }
        }

        int result = 0;
        for (int i = 0; i < words.length; ++i) {
            for (int j = i + 1; j < words.length; ++j) {
                if ((arr[i] & arr[j]) == 0) {
                    int temp = words[j].length() * words[i].length();
                    result = Math.max(temp, result);
                }
            }
        }

        return result;
    }
}
```

![image-20221027090129429](D:\WorkSpace\Note\Picture\image-20221027090129429.png)

# 数组

## 双指针

### 排序数组中的两个数字之和 [goto](https://leetcode.cn/problems/kLl5u1/)

```java
class Solution {
    public int[] twoSum(int[] numbers, int target) {
        int i = 0, j = numbers.length - 1;
        while (i < j) {
            if (target == numbers[i] + numbers[j]) {
                break;
            } else if (target > numbers[i] + numbers[j]) {
                i++;
            } else {
                j--;
            }
        }
        return new int[]{i, j};
    }
}
```

​	![image-20221027090827750](D:\WorkSpace\Note\Picture\image-20221027090827750.png)

### 数组中和为0的三个数 [goto]()

```java
class Solution {
    public List<List<Integer>> threeSum(int[] nums) {
        Arrays.sort(nums);
        List<List<Integer>> result = new ArrayList<>();
        if (nums.length < 3) {
            return result;
        }
        int i = 0;
        while (i < nums.length - 2) {
            twoSum(result, i, nums);
            
            int temp = nums[i];
            while (i < nums.length && temp == nums[i]) {
                i++;
            }
        }
        return result;
    }

    public void twoSum(List<List<Integer>> result, int i, int[] nums) {
        int l = i + 1;
        int r = nums.length - 1;
        while (l < r) {
            if (nums[i] + nums[l] + nums[r] == 0) {
                List<Integer> list = new ArrayList<>(Arrays.asList(nums[i], nums[l], nums[r]));
                result.add(list);

                while (l < r && temp == nums[l]) {
                    l++;
                }
            } else if (nums[i] + nums[l] + nums[r] > 0) {
                r--;
            } else {
                l++;
            }
        }
    }
}
```

![image-20221027094357966](D:\WorkSpace\Note\Picture\image-20221027094357966.png)

### 和大于等于target的最短数组 [goto](https://leetcode.cn/problems/2VG8Kg/)

```java
class Solution {
    public int minSubArrayLen(int target, int[] nums) {
        int l = 0;
        int sum = 0;
        int min = Integer.MAX_VALUE;
        for (int r = 0; r < nums.length; r++) {
            sum += nums[r];
            while (l <= r && sum >= target) {
                min = Math.min(min, (r - l + 1));
                sum -= nums[l++];
            }
        }
        return min == Integer.MAX_VALUE ? 0 : min;
    }
}
```

![image-20221027100117096](D:\WorkSpace\Note\Picture\image-20221027100117096.png)

### 乘积小于k的子数组 [goto](https://leetcode.cn/problems/ZVAVXX/)

```java
class Solution {
    public int numSubarrayProductLessThanK(int[] nums, int k) {
        if (k <= 1) {
            return 0;
        }
        int left = 0;
        int result = 0;
        for (int right = 0, cur = 1; right < nums.length; ++right) {
            cur *= nums[right];
            while (cur >= k) {
                cur /= nums[left++];
            }
            result += right - left + 1;
        }
        return result;
    }
}
```

![image-20221028174145756](D:\WorkSpace\Note\Picture\image-20221028174145756.png)

## 累加数组数字求子数组之和

### 和为k的子数组 [goto](https://leetcode.cn/problems/QTMn0o/)

```java
class Solution {
    public int subarraySum(int[] nums, int k) {
        HashMap<Integer, Integer> map = new HashMap<>();
        map.put(0, 1);
        int sum = 0;
        int count = 0;
        for (int num : nums) {
            sum += num;
            count += map.getOrDefault(sum - k, 0);
            map.put(sum, map.getOrDefault(sum, 0) + 1);   
        }
        return count;
    }
}
```

![image-20221028191105837](D:\WorkSpace\Note\Picture\image-20221028191105837.png)

### 0和1个数相同的子数组 [goto](https://leetcode.cn/problems/A1NYOS/)

```java
class Solution {
    public int findMaxLength(int[] nums) {
        HashMap<Integer, Integer> map = new HashMap<>();
        int sum = 0;
        int max = 0;
        map.put(0, -1);
        for (int i = 0; i < nums.length; ++i) {
            sum += (nums[i] == 0 ? -1 : 1);
            if (map.containsKey(sum)) {
                max = Math.max(max, i - map.get(sum));
            } else {
                map.put(sum, i);
            }
        }
        return max;
    }
}
```

![image-20221028211412918](D:\WorkSpace\Note\Picture\image-20221028211412918.png)

### 左右两边子数组的和相等 [goto](https://leetcode.cn/problems/tvdfij/)

```java
class Solution {
    public int pivotIndex(int[] nums) {
        int sum = 0;
        for (int num : nums) {
            sum += num;
        }
        int curSum = 0;
        for (int i = 0; i < nums.length; ++i) {
            if (i != 0) {
                curSum += nums[i - 1]; 
            }
            if (curSum * 2 + nums[i] == sum) {
                return i;
            }
        }
        return -1;
    }
}
```

![image-20221029140644096](D:\WorkSpace\Note\Picture\image-20221029140644096.png)

### 二维子矩阵的数字之和 [goto]()

```java
class NumMatrix {

    private int[][] sum;

    public NumMatrix(int[][] matrix) {
        sum = new int[matrix.length + 1][matrix[0].length + 1];
        for (int i = 1; i <= matrix.length; ++i) {
            for (int j = 1; j <= matrix[0].length; ++j) {
                sum[i][j] = sum[i - 1][j] + sum[i][j - 1] - sum[i - 1][j - 1] + matrix[i - 1][j - 1];
            }
        }
        System.out.println(Arrays.deepToString(sum));
    }
    
    public int sumRegion(int row1, int col1, int row2, int col2) {
        return sum[row2 + 1][col2 + 1] + sum[row1][col1] - sum[row2 + 1][col1] - sum[row1][col2 + 1];
    }
}
```

![image-20221029151115328](D:\WorkSpace\Note\Picture\image-20221029151115328.png)

# 字符串

## 双指针

### 字符串中的变位词 [goto]()

```java
class Solution {
    public boolean checkInclusion(String s1, String s2) {
        if (s2.length() < s1.length()) {
            return false;
        }

        int[] counts = new int[26];
        for (int i = 0; i < s1.length(); ++i) {
            counts[s1.charAt(i) - 'a']++;
            counts[s2.charAt(i) - 'a']--;
        }

        if (ifOk(counts)) {
            return true;
        }

        for (int i = s1.length(); i < s2.length(); ++i) {
            counts[s2.charAt(i) - 'a']--;
            counts[s2.charAt(i - s1.length()) - 'a']++; 
            if (ifOk(counts)) {
                return true;
            }
        }

        return false;
    }

    public boolean ifOk(int[] counts) {
        for (int count : counts) {
            if (count != 0) {
                return false;
            }
        }
        return true;
    }
}
```

![image-20221029154911754](D:\WorkSpace\Note\Picture\image-20221029154911754.png)

### 字符串中的所有变位词 [goto](https://leetcode.cn/problems/VabMRr/)

```java
class Solution {
    public List<Integer> findAnagrams(String s1, String s2) {
        List<Integer> result = new ArrayList<>();
        if (s1.length() < s2.length()) {
            return result;
        }

        int[] arr = new int[26];
        int cur = 0;

        for (int i = 0; i < s2.length(); ++i) {
            arr[s1.charAt(i) - 'a']--;
            arr[s2.charAt(i) - 'a']++;
        }

        if (doFun(arr)) {
            result.add(cur);
        }

        for (int i = s2.length(); i < s1.length(); ++i) {
            cur = i - s2.length() + 1;
            arr[s1.charAt(i) - 'a']--;
            arr[s1.charAt(i - s2.length()) - 'a']++;
            if (doFun(arr)) {
                result.add(cur);
            }
        }
        
        return result;
    }

    public boolean doFun(int[] arr) {
        for (int num : arr) {
            if (num != 0) {
                return false;
            }
        }
        return true;
    }
}
```

![image-20221030111508526](D:\WorkSpace\Note\Picture\image-20221030111508526.png)

### 不包含字符的最长字符串 [goto](https://leetcode.cn/problems/wtcaE1/)

```java
class Solution {
    public int lengthOfLongestSubstring(String s) {
        if (s.length() == 0) {
            return 0;
        }

        int[] arr = new int[256];
        int i = 0, j = -1, max = 1, temp = 0;
        for (; i < s.length(); ++i) {
            arr[s.charAt(i)]++;

            if (arr[s.charAt(i)] == 2) {
                temp++;
            }

            while (temp > 0) {
                ++j;
                arr[s.charAt(j)]--;
                if (arr[s.charAt(j)] == 1) {
                    temp--;
                }
            }

            max = Math.max(max, i - j);
        }

        return max;
    }
}
```

![image-20221030115601775](D:\WorkSpace\Note\Picture\image-20221030115601775.png)

### 含有所有字符的最短字符串 [goto](https://leetcode.cn/problems/M1oyTv/)

```java
class Solution {
    public String minWindow(String s, String t) {
        Map<Character, Integer> map = new HashMap<>();

        for (char ch : t.toCharArray()) {
            map.put(ch, map.getOrDefault(ch, 0) + 1);
        }

        int count = map.size(), start = 0, end = 0, minStart = 0, minEnd = 0, minLength = Integer.MAX_VALUE;

        while (end < s.length() || (count == 0 &&  end == s.length())) {
            if (count > 0) {
                char endCh = s.charAt(end);
                if (map.containsKey(endCh)) {
                    map.put(endCh, map.getOrDefault(endCh, 0) - 1);
                    if (map.get(endCh) == 0) {
                        count--;
                    }
                }
                end++;
            } else {
                if (end - start < minLength) {
                    minLength = end - start;
                    minEnd = end;
                    minStart = start;
                }
                char startCh = s.charAt(start);
                if (map.containsKey(startCh)) {
                    map.put(startCh, map.getOrDefault(startCh, 0) + 1);
                    if (map.get(startCh) == 1) {
                        count++;
                    }
                }
                start++;
            }
        }
        
        return minLength < Integer.MAX_VALUE ? s.substring(minStart, minEnd) : "";
    }
}
```

![image-20221030141339653](D:\WorkSpace\Note\Picture\image-20221030141339653.png)

## 回文字符串

### 有效的回文串 [goto](https://leetcode.cn/problems/XltzEq/)

```java
class Solution {
    public boolean isPalindrome(String s) {
        int left = 0, right = s.length() - 1;
        while (left < right) {
            char charA = s.charAt(left);
            char charB = s.charAt(right);
            if (!Character.isLetterOrDigit(charA)) {
                left++;
            } else if (!Character.isLetterOrDigit(charB)) {
                right--;
            } else {
                charA = Character.toLowerCase(charA);
                charB = Character.toLowerCase(charB);
                if (charA != charB) {
                    return false;
                }
                left++;
                right--;
            }
        }
        return true;
    }
}
```

![image-20221031093159007](D:\WorkSpace\Note\Picture\image-20221031093159007.png)

### 最多删除一个字符串得到回文 [goto](https://leetcode.cn/problems/RQku0D/)

```java
class Solution {
    public boolean validPalindrome(String s) {
        int start = 0;
        int end = s.length() - 1;
        for (; start < s.length() / 2; ++start, --end) {
            if (s.charAt(start) != s.charAt(end)) {
                break;
            }
        }

        return start == s.length() / 2 || isPalindrome(s, start, end - 1) || isPalindrome(s, start + 1, end);
    }

    private boolean isPalindrome(String s, int start, int end) {
        while (start < end) {
            if (s.charAt(start) != s.charAt(end)) {
                break;
            }

            start++;
            end--;
        }

        return start >= end;
    }
}
```

![image-20221031095110457](D:\WorkSpace\Note\Picture\image-20221031095110457.png)

### 回文子字符的个数 [goto](https://leetcode.cn/problems/a7VOhD/)

```java
class Solution {
    public int countSubstrings(String s) {
        if (s == null || s.length() == 0) {
            return 0;
        }

        int count = 0;
        for (int i = 0; i < s.length(); ++i) {
            count += doFun(s, i, i);
            count += doFun(s, i, i + 1);
        }

        return count;
    }

    public int doFun(String str, int start, int end) {
        int count = 0;
        while (start >=0 && end < str.length() && str.charAt(start) == str.charAt(end)) {
            count++;
            start--;
            end++;
        }

        return count;
    }
}
```

![image-20221031101937059](D:\WorkSpace\Note\Picture\image-20221031101937059.png)

# 链表

## 双指针

### 删除倒是第k个节点 [goto](https://leetcode.cn/problems/SLwz0R/)

```java
/**
 * Definition for singly-linked list.
 * public class ListNode {
 *     int val;
 *     ListNode next;
 *     ListNode() {}
 *     ListNode(int val) { this.val = val; }
 *     ListNode(int val, ListNode next) { this.val = val; this.next = next; }
 * }
 */
class Solution {
    public ListNode removeNthFromEnd(ListNode head, int n) {
        ListNode ahead = new ListNode(0);
        ahead.next = head;

        ListNode cur = head;
        ListNode ch = ahead;
        for (int i = 0; i < n; ++i) {
            cur = cur.next;
        }

        while (cur != null) {
            cur = cur.next;
            ch = ch.next;
        }

        ch.next = ch.next.next;

        return ahead.next;
    }
}
```

![image-20221101212327380](D:\WorkSpace\Note\Picture\image-20221101212327380.png)

### 链表中环的入口节点 [goto](https://leetcode.cn/problems/c32eOV/)

```java
public class Solution {
    public ListNode detectCycle(ListNode head) {
        ListNode fast = head, slow = head;
        while (fast != null && fast.next != null) {
            slow = slow.next;
            fast = fast.next.next;

            if (slow == fast) {
                slow = head;
                while (slow != fast) {
                    slow = slow.next;
                    fast = fast.next;
                }
                return slow;
            }
        }
        return null;
    }
}
```

![image-20221101220145036](D:\WorkSpace\Note\Picture\image-20221101220145036.png)

### 两个链表的第一个重合节点 [goto](https://leetcode.cn/problems/3u1WK4/)

```java
/**
 * Definition for singly-linked list.
 * public class ListNode {
 *     int val;
 *     ListNode next;
 *     ListNode(int x) {
 *         val = x;
 *         next = null;
 *     }
 * }
 */
public class Solution {
    public ListNode getIntersectionNode(ListNode headA, ListNode headB) {
       ListNode a = headA, b = headB;
       while (a.next != null && b.next != null) {
           a = a.next;
           b = b.next;
       } 
         int count = 0;
       if (a.next == null) {
           while (b.next != null) {
               b = b.next;
               count++;
           }
           a = headB;
           b = headA;
       } else {
           while (a.next != null) {
               a = a.next;
               count++;
           }
           a = headA;
           b = headB;
       }

        for (int i = 0; i < count; ++i) {
            a = a.next;
        }

        while (a != b) {
            a = a.next;
            b = b.next;
        }

        return a;
    }
}
```

![image-20221101221101942](D:\WorkSpace\Note\Picture\image-20221101221101942.png)

## 反转链表

### 反转链表 [goto](https://leetcode.cn/problems/UHnkqh/)

```java
/**
 * Definition for singly-linked list.
 * public class ListNode {
 *     int val;
 *     ListNode next;
 *     ListNode() {}
 *     ListNode(int val) { this.val = val; }
 *     ListNode(int val, ListNode next) { this.val = val; this.next = next; }
 * }
 */
class Solution {
    public ListNode reverseList(ListNode head) {
        ListNode cur = head;
        ListNode pre = null;

            while (cur != null) {
            ListNode temp = pre;
            pre = cur;
            cur = pre.next;
            pre.next = temp;
        }

        return pre;
    }
}
```

![image-20221102085425439](D:\WorkSpace\Note\Picture\image-20221102085425439.png)

### 链表中两数相加 [goto](https://leetcode.cn/problems/lMSNwu/)

```java
class Solution {
    public ListNode addTwoNumbers(ListNode l1, ListNode l2) {
        Deque<Integer> stack1 = new ArrayDeque<Integer>();
        Deque<Integer> stack2 = new ArrayDeque<Integer>();
        while (l1 != null) {
            stack1.push(l1.val);
            l1 = l1.next;
        }
        while (l2 != null) {
            stack2.push(l2.val);
            l2 = l2.next;
        }
        int carry = 0;
        ListNode ans = null;
        while (!stack1.isEmpty() || !stack2.isEmpty() || carry != 0) {
            int a = stack1.isEmpty() ? 0 : stack1.pop();
            int b = stack2.isEmpty() ? 0 : stack2.pop();
            int cur = a + b + carry;
            carry = cur / 10;
            cur %= 10;
            ListNode curnode = new ListNode(cur);
            curnode.next = ans;
            ans = curnode;
        }
        return ans;
    }
}
```

![image-20221102092156430](D:\WorkSpace\Note\Picture\image-20221102092156430.png)

### 重排链表 [goto](https://leetcode.cn/problems/LGjMqU/)

```java
/**
 * Definition for singly-linked list.
 * public class ListNode {
 *     int val;
 *     ListNode next;
 *     ListNode() {}
 *     ListNode(int val) { this.val = val; }
 *     ListNode(int val, ListNode next) { this.val = val; this.next = next; }
 * }
 */
class Solution {
    public void reorderList(ListNode head) {
        if (head == null) {
            return;
        }

        List<ListNode> list = new ArrayList<>();
        while (head != null) {
            list.add(head);
            head = head.next;
        }

        int i = 0, j = list.size() - 1;
        while (i < j) {
            list.get(i).next = list.get(j);
            i++;
            if (i == j) {
                break;
            }
            list.get(j).next = list.get(i);
            j--;
        }
        list.get(j).next = null;
    }
}
```

![image-20221102212238961](D:\WorkSpace\Note\Picture\image-20221102212238961.png)

### 回文链表 [goto](https://leetcode.cn/problems/aMhZSa/)

```java
/**
 * Definition for singly-linked list.
 * public class ListNode {
 *     int val;
 *     ListNode next;
 *     ListNode() {}
 *     ListNode(int val) { this.val = val; }
 *     ListNode(int val, ListNode next) { this.val = val; this.next = next; }
 * }
 */
class Solution {
    ListNode ahead;

    public boolean isPalindrome(ListNode head) {
        ahead = head;
        return doFun(head);
    }

    public boolean doFun(ListNode cur) {
        if (cur != null) {
            if (!doFun(cur.next)) {
                return false;
            }

            if (cur.val != ahead.val) {
                return false;
            }

            ahead = ahead.next;
        }
        return true;
    }
}
```

![image-20221103210920977](D:\WorkSpace\Note\Picture\image-20221103210920977.png)

## 双向链表和循环链表

### 展平多级双向链表 [goto](https://leetcode.cn/problems/Qv1Da2/)

```java
/*
// Definition for a Node.
class Node {
    public int val;
    public Node prev;
    public Node next;
    public Node child;
};
*/

class Solution {
    public Node flatten(Node head) {
        if (head == null) {
            return head;
        }
        doFun(head);
        return head;
    }

    public Node doFun(Node cur) {
        while (cur.next != null || cur.child != null) {
            if (cur.child != null && cur.next != null) {
                Node temp = cur.next;
                cur.next = cur.child;
                cur.child.prev = cur;
                cur.child = null;
                Node last = doFun(cur.next);
                last.next = temp;
                temp.prev = last;
            } else if (cur.next == null) {
                cur.next = cur.child;
                cur.child.prev = cur;
                cur.child = null;
                Node last = doFun(cur.next);
            }
            cur = cur.next;
        }
        return cur;
    }
}
```

![image-20221103221905621](D:\WorkSpace\Note\Picture\image-20221103221905621.png)

### 排序的循环链表 [goto](https://leetcode.cn/problems/4ueAj6/)

```java
/*
// Definition for a Node.
class Node {
    public int val;
    public Node next;

    public Node() {}

    public Node(int _val) {
        val = _val;
    }

    public Node(int _val, Node _next) {
        val = _val;
        next = _next;
    }
};
*/

class Solution {
    public Node insert(Node head, int insertVal) {
       if (head == null) {
           Node node = new Node(insertVal);
           node.next = node;
           return node;
       } 

        Node temp = head;
        Node node = head;
       while (node.val <= node.next.val && node.next != node && node.next != head) {
           node = node.next;
       }
       System.out.println(node.val);

        Node cur = node.next;
       System.out.println(cur.val);

        if (insertVal < node.val) {
            while (insertVal > cur.val) {
                cur = cur.next;
                node = node.next;
            }
        }

        temp = new Node(insertVal);
        node.next = temp;
        temp.next = cur;

        return head;
    }
}
```

![image-20221104175358549](D:\WorkSpace\Note\Picture\image-20221104175358549.png)

# 哈希表

## 哈希表的设计

### 插入、删除和随机访问都是O(1)的容器 [goto](https://leetcode.cn/problems/FortPu/)

```java
class RandomizedSet {

    private List<Integer> list;
    private Map<Integer, Integer> map;

    /** Initialize your data structure here. */
    public RandomizedSet() {
        list = new ArrayList<>();
        map = new HashMap<>();
    }
    
    /** Inserts a value to the set. Returns true if the set did not already contain the specified element. */
    public boolean insert(int val) {
        if (map.containsKey(val)) {
            return false;
        }

        map.put(val, list.size());
        list.add(val);
        return true;
    }
    
    /** Removes a value from the set. Returns true if the set contained the specified element. */
    public boolean remove(int val) {
        if (!map.containsKey(val)) {
            return false;
        }

        int location = map.get(val);
        map.put(list.get(list.size() -1), location);
        map.remove(val);
        list.set(location, list.get(list.size() - 1));
        list.remove(list.size() - 1);
        return true;
    }
    
    /** Get a random element from the set. */
    public int getRandom() {
        Random random = new Random();
        int r = random.nextInt(list.size());
        return list.get(r);
    }
}

/**
 * Your RandomizedSet object will be instantiated and called as such:
 * RandomizedSet obj = new RandomizedSet();
 * boolean param_1 = obj.insert(val);
 * boolean param_2 = obj.remove(val);
 * int param_3 = obj.getRandom();
 */
```

![image-20221103224222772](D:\WorkSpace\Note\Picture\image-20221103224222772.png)

### 最近最少使用缓存 [goto](https://leetcode.cn/problems/OrIXps/)

```java
class LRUCache {
    int limit;
    Node head;
    Node tail;
    Map<Integer, Node> map;

    public LRUCache(int capacity) {
        limit = capacity;
        head = new Node(-1, null);
        tail = new Node(-1, null);
        head.next = tail;
        tail.prev = head;
        map = new HashMap<>();
    }

    public int get(int key) {
        Node node = map.get(key);
        if (node == null) {
            return -1;
        }
        unlink(node);
        toHead(node);
        return node.value;
    }

    public void put(int key, int value) {
        Node node = map.get(key);
        if (node == null) {
            node = new Node(key, value);
            map.put(key, node);
        } else {
            node.value = value;
            unlink(node);
        }
        toHead(node);
        if (map.size() > limit) {
            Node last = tail.prev;
            map.remove(last.key);
            unlink(last);
        }
    }

    public void unlink(Node node) {
        node.prev.next = node.next;
        node.next.prev = node.prev;
    }

    public void toHead(Node node) {
        node.prev = head;
        node.next = head.next;
        head.next.prev = node;
        head.next = node;
    }

    class Node {
        Node prev;
        Node next;
        int key;
        Integer value;

        public Node(int key, Integer value) {
            this.key = key;
            this.value = value;
        }
    }
}

/**
 * Your LRUCache object will be instantiated and called as such:
 * LRUCache obj = new LRUCache(capacity);
 * int param_1 = obj.get(key);
 * obj.put(key,value);
 */
```

![image-20221104205726378](D:\WorkSpace\Note\Picture\image-20221104205726378.png)

## 哈希表的应用

### 有效的变位词 [goto](https://leetcode.cn/problems/dKk3P7/)

```java
class Solution {
    int[] arr = new int[26];

    public boolean isAnagram(String s, String t) {
        if (s.equals(t)) {
            return false;
        }

        for (char ch : s.toCharArray()) {
            arr[ch - 'a']++;
        }

        for (char ch : t.toCharArray()) {
            arr[ch - 'a']--;
        }

        for (int i : arr) {
            if (i != 0) {
                return false;
            }
        }

        return true;
    }
}
```

![image-20221104210406556](D:\WorkSpace\Note\Picture\image-20221104210406556.png)

### 变位词组 [goto](https://leetcode.cn/problems/sfvd7V/)

```java
class Solution {
    private HashMap<String, List<String>> map = new HashMap<>();
    private List<List<String>> list = new ArrayList<>();

    public List<List<String>> groupAnagrams(String[] strs) {
        for (String str : strs) {
            char[] chs = str.toCharArray();
            Arrays.sort(chs);
            String s = new String(chs);
            if(!map.containsKey(s)){
                map.put(s,new ArrayList<>());
            }
            map.get(s).add(str);
        }
        return new ArrayList<>(map.values());
    }
}
```

![image-20221104212405029](D:\WorkSpace\Note\Picture\image-20221104212405029.png)

### 外星语言是否排序 [goto](https://leetcode.cn/problems/lwyVBB/)

```java
class Solution {
    public boolean isAlienSorted(String[] words, String order) {
        int[] index = new int[26];
        for (int i = 0; i < order.length(); ++i) {
            index[order.charAt(i) - 'a'] = i;
        }
        for (int i = 1; i < words.length; i++) {
            boolean valid = false;
            for (int j = 0; j < words[i - 1].length() && j < words[i].length(); j++) {
                int prev = index[words[i - 1].charAt(j) - 'a'];
                int curr = index[words[i].charAt(j) - 'a'];
                if (prev < curr) {
                    valid = true;
                    break;
                } else if (prev > curr) {
                    return false;
                }
            }
            if (!valid) {
                /* 比较两个字符串的长度 */
                if (words[i - 1].length() > words[i].length()) {
                    return false;
                }
            }
        }
        return true;
    }
}
```

![image-20221105210146037](D:\WorkSpace\Note\Picture\image-20221105210146037.png)

### 最小时间差 [goto](https://leetcode.cn/problems/569nqc/)

```java
class Solution {
    public int findMinDifference(List<String> timePoints) {
        boolean[] arr = new boolean[1441];
        for (String item : timePoints) {
            int time = toTime(item);
            if (arr[time]) {
                return 0;
            }
            arr[time] = true;
        }

        int min = Integer.MAX_VALUE;
        int pre = -1, first = -1;
        for (int i = 0; i < arr.length; ++i) {
            if (!arr[i]) {
                continue;
            }
            if (pre != -1) {
                min = Math.min(Math.abs(i - pre), min);
            } else {
                first = i;
            }
            pre = i;
        }
        min = Math.min(first + (1440 - pre), min);

        return min;
    }

    private int toTime(String time) {
        String[] temp = time.split(":");
        return Integer.parseInt(temp[0]) * 60 + Integer.parseInt(temp[1]);
    }
}
```

![image-20221106114514172](D:\WorkSpace\Note\Picture\image-20221106114514172.png)

# 栈

## 栈的应用

### 后缀表达式 [goto](https://leetcode.cn/problems/8Zf90G/)

```java
import java.util.regex.Pattern;

class Solution {
    public int evalRPN(String[] tokens) {
        Deque<Integer> stack = new LinkedList<>();

        for (String token : tokens) {
            char ch = token.charAt(0);
            if (isInteger(token)) {
                stack.push(Integer.parseInt(token));
            } else {
                int param1 = stack.pop();
                int param2 = stack.pop();
                if ("+".equals(token)) {
                    stack.push(param2 + param1);
                } else if ("-".equals(token)) {
                    stack.push(param2 - param1);
                } else if ("*".equals(token)) {
                    stack.push(param2 * param1);
                } else if ("/".equals(token)) {
                    stack.push(param2 / param1);
                } else {
                    stack.push(param2 % param1);
                }
            }
        }

        return stack.pop();
    }

    public static boolean isInteger(String str) { 
            Pattern pattern = Pattern.compile("-?[0-9]+\\.?[0-9]*");  
            return pattern.matcher(str).matches();  
    }
}
```

