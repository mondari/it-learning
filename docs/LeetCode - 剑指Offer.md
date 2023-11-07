# 字符串

## [剑指 Offer 05. 替换空格](https://leetcode.cn/problems/ti-huan-kong-ge-lcof/)

```java
class Solution {
    public String replaceSpace(String s) {
        StringBuilder sb = new StringBuilder();
        for (int i = 0; i < s.length(); i++) {
            char c = s.charAt(i);
            if (c == ' ') {
                sb.append("%20");
            } else {
                sb.append(c);
            }
        }
        return sb.toString();
    }
}
```

## [剑指 Offer 58 - II. 左旋转字符串](https://leetcode.cn/problems/zuo-xuan-zhuan-zi-fu-chuan-lcof/)

**1.StringBuilder 解法**

```java
class Solution {
    public String reverseLeftWords(String s, int n) {
        StringBuilder sb = new StringBuilder();
        for (int i = n; i < s.length(); i++) {
            sb.append(s.charAt(i));
        }
        for (int i = 0; i < n; i++) {
            sb.append(s.charAt(i));
        }
        return sb.toString();
    }
}
```

**2.切片法(击败100%)**

```java
class Solution {
    public String reverseLeftWords(String s, int n) {
        return s.substring(n) + s.substring(0, n);
    }
}
```

## [*剑指 Offer 20. 表示数值的字符串](https://leetcode.cn/problems/biao-shi-shu-zhi-de-zi-fu-chuan-lcof/)

知识点：状态机、正则匹配

**我的解法**

[sign] < digits [.] digits > [e [sign] digits]

逐位匹配，有以下规则：

- 首位只能是符号字符、数字、“.”
- 中间有效字符只是数字、“e”、“E”、“.”
- 数字至少存在1个
- 符号字符只能存在1个
- “e”、“E”、“.” 只能存在一个
- “.” 左或右要有数字
- “e” 或 “E” 后面要有整数
- “e” 或 “E” 后面符号位要在第一位

```java
class Solution {
    public boolean isNumber(String s) {
        boolean hasDot = false;// 是否有小数点，重复出现则非法
        boolean hasNumber = false;// 是否有数字，没有则非法
        int idx_e = -1;// e出现的位置
        boolean hasENumber = false; //e后面是否有数字，没有则非法
        boolean hasESign = false;// e后面是否有+-符号，重复出现则非法
        s = s.trim();
        for (int i = 0; i < s.length(); i++) {
            char c = s.charAt(i);
            // 首位只能是符号字符、数字、“.”
            if (i == 0) {
                if (c == '.') {
                    hasDot = true;
                } else if (Character.isDigit(c)) {
                    hasNumber = true;
                } else if (c != '-' && c != '+') {
                    return false;
                }
                continue;
            }
            // e后面是整数
            if (idx_e != -1) {
                if (c == '-' || c == '+') {
                    if (hasESign == true || idx_e != i - 1) {
                        return false;
                    }
                    hasESign = true;
                } else if (Character.isDigit(c)) {
                    hasENumber = true;
                } else {
                    return false;
                }
            } else if (c == '.') {
                if (hasDot) {
                    return false;
                }
                hasDot = true;
            } else if (Character.isDigit(c)) {
                hasNumber = true;
            } else if (c == 'e' || c == 'E') {
                if (!hasNumber || idx_e != -1) {
                    return false;
                }
                idx_e = i;
            } else {
                return false;
            }
        }
        if (!hasNumber) {// 数字至少存在1个，否则非法
            return false;
        } else if (idx_e != -1 && !hasENumber) {// e后面要有数字，否则非法
            return false;
        }
        return true;
    }
}
```

## [剑指 Offer 67. 把字符串转换成整数](https://leetcode.cn/problems/ba-zi-fu-chuan-zhuan-huan-cheng-zheng-shu-lcof/)

注意：本题与 [8. 字符串转换整数 (atoi) - 力扣（LeetCode）](https://leetcode.cn/problems/string-to-integer-atoi/) 相同

该题覆盖知识点比较多，列举如下：

- 字符串去除首尾空格：String#trim()
- 字符串取字符：String#charAt()
- 字符串转字符数组：String#toCharArray()
- 字符串长度：String#length()
- 判断字符是否是数字：char >= '0' && char <= '9'，或 Character.isDigit()
- 字符转数字+数字拼接：num = num * 10 + (char - '0')
- 符号位：signed=0,1,-1 分别表示未判定，正号，负号
- 整数最大数和最小数：Integer.MAX_VALUE=2147483647，Integer.MIN_VALUE=-2147483648
- 越界判定：因为bounds=Integer.MAX_VALUE/10=214748364，正数余7，负数余8，所以以下两种情况视为越界
  - num > bounds
  - num == bounds, (char - '0') > 7（因为>7就返回最值，所以无需判断负数>8的情况）
- 字符串首位要判断是否是“+”、“-”、“0-9”，否则不是有效数字
- 字符串非首位不是数字则终止

代码示例：

```java
class Solution {
    public int strToInt(String str) {
        if (str == null) {
            return 0;
        }

        str = str.trim();

        int signed = 0;
        int num = 0;
        int bounds = Integer.MAX_VALUE / 10;
        int digit = 0;

        for (int i = 0; i < str.length(); i++) {
            char c = str.charAt(i);

            if (i == 0) {
                if (c == '-') {
                    signed = -1;
                    continue;
                } else if (c == '+') {
                    signed = 1;
                    continue;
                } else if (c < '0' || c > '9') {
                    return 0;
                }
            }

            if (c >= '0' && c <= '9') {
                digit = c - '0';

                if (num > bounds || (bounds == num && digit > 7)) {
                    return signed == -1 ? Integer.MIN_VALUE : Integer.MAX_VALUE;
                }
                num = num * 10 + digit;
            } else {
                break;
            }

        }

        if (signed == -1) {
            num = -num;
        }

        return num;
    }
}
```

**注意以下问题**

**整数最小值溢出**

> 输入："-2147483649"
>输出：-2147483648

**整数最大值溢出**

> 输入："2147483648"
>输入：2147483647

**整数最大值**

> 输入："2147483646"
>输出：2147483646

**中间有个负号**

> 输入："0-1"
>输出：0

**测试用例**

>"42"
>"   -42"
>"4193 with words"
>"words and 987"
>"-91283472332"
>"0-1"
>"-2147483649"
>"2147483648"
>"2147483646"

# 链表

## [剑指 Offer 06. 从尾到头打印链表](https://leetcode.cn/problems/cong-wei-dao-tou-da-yin-lian-biao-lcof/)

**1.栈**

Java Stack 继承自 Vector，速度慢，建议用 LinkedList 代替。

```java
import java.util.Stack;

/**
 * Definition for singly-linked list.
 * public class ListNode {
 * int val;
 * ListNode next;
 * ListNode(int x) { val = x; }
 * }
 */
class Solution {
    public int[] reversePrint(ListNode head) {
        Stack<Integer> stack = new Stack<>();
        while (head != null) {
            stack.push(head.val);
            head = head.next;
        }
        int[] nums = new int[stack.size()];
        int i = 0;
        while (!stack.empty()) {
            nums[i++] = stack.pop();
        }
        return nums;
    }
}
```

**2.递归**

```java
import java.util.Arrays;

/**
 * Definition for singly-linked list.
 * public class ListNode {
 * int val;
 * ListNode next;
 * ListNode(int x) { val = x; }
 * }
 */
class Solution {
    public int[] reversePrint(ListNode head) {
        if (head == null) {
            return new int[0];
        }
        if (head.next == null) {
            return new int[]{head.val};
        }
        int[] arr = reversePrint(head.next);
        int[] nums = Arrays.copyOf(arr, arr.length + 1);
        nums[arr.length] = head.val;
        return nums;
    }
}
```

**3.不用栈+递归(击败100%)**

先求链表长度，创建数组，遍历链表，将其节点倒序存放到数组中

```java
/**
 * Definition for singly-linked list.
 * public class ListNode {
 * int val;
 * ListNode next;
 * ListNode(int x) { val = x; }
 * }
 */
class Solution {
    public int[] reversePrint(ListNode head) {
        int length = 0;
        ListNode node = head;
        while (node != null) {
            node = node.next;
            length++;
        }

        int[] nums = new int[length];
        for (int i = length - 1; i >= 0; i--) {
            nums[i] = head.val;
            head = head.next;
        }

        return nums;
    }
}
```

## [剑指 Offer 24. 反转链表](https://leetcode.cn/problems/fan-zhuan-lian-biao-lcof/)

注意：本题与 [206. 反转链表 - 力扣（LeetCode）](https://leetcode.cn/problems/reverse-linked-list/) 相同

**1.迭代解法**

```java
/**
 * Definition for singly-linked list.
 * public class ListNode {
 *     int val;
 *     ListNode next;
 *     ListNode(int x) { val = x; }
 * }
 */
class Solution {
    public ListNode reverseList(ListNode head) {
        ListNode prev = null;
        while (head != null) {
            ListNode next = head.next;
            head.next = prev;

            prev = head;
            head = next;
        }
        return prev;
    }
}
```

步骤：

1. 先保存 next 节点
2. 将当前节点指向上一个节点
3. 将当前节点保存为上一个节点，将 next 节点作为当前节点
4. 继续步骤1，直到当前节点为空
5. 返回上一个节点

**2.递归解法**

```java
/**
 * Definition for singly-linked list.
 * public class ListNode {
 *     int val;
 *     ListNode next;
 *     ListNode(int x) { val = x; }
 * }
 */
class Solution {
    public ListNode reverseList(ListNode head) {
        // 链表为空，或只有一个节点，直接返回
        if (head == null || head.next == null) {
            return head;
        }
        ListNode node = reverseList(head.next);
        head.next.next = head;
        head.next = null;
        return node;
    }
}
```

递归的 base case：找到链表最后一个节点。

递归的操作：将 next 节点指向当前节点，将当前节点指向 null。最后一步是为了原链表的头节点指向 null，不会影响下一步递归的操作。

## [剑指 Offer 35. 复杂链表的复制](https://leetcode.cn/problems/fu-za-lian-biao-de-fu-zhi-lcof/)

注意：本题与 [138. 复制带随机指针的链表](https://leetcode.cn/problems/copy-list-with-random-pointer/) 相同

```java
/*
// Definition for a Node.
class Node {
    int val;
    Node next;
    Node random;

    public Node(int val) {
        this.val = val;
        this.next = null;
        this.random = null;
    }
}
*/
class Solution {
    public Node copyRandomList(Node head) {
        if (head == null) {
            return head;
        }

        // 将复制的节点追加到原节点后面
        Node node = head;
        while (node != null) {
            Node next = node.next;
            Node copy = new Node(node.val);
            copy.next = next;
            node.next = copy;
            node = next;
        }

        // 复制random
        node = head;
        while (node != null) {
            Node random = node.random;
            if (random != null) {
                node.next.random = random.next;
            }
            node = node.next.next;
        }

        // 拆分成新链表，恢复旧链表
        Node newHead = node = head.next;
        Node prev = head;
        while (node != null && node.next != null && node.next.next != null) {
            prev.next = node.next;
            prev = node.next;

            node.next = node.next.next;
            node = node.next;
        }
        prev.next = null;

        return newHead;
    }
}
```

# 双指针

## [剑指 Offer 18. 删除链表的节点](https://leetcode.cn/problems/shan-chu-lian-biao-de-jie-dian-lcof/)

**1.双指针法**

```java
/**
 * Definition for singly-linked list.
 * public class ListNode {
 * int val;
 * ListNode next;
 * ListNode(int x) { val = x; }
 * }
 */
class Solution {
    public ListNode deleteNode(ListNode head, int val) {
        if (head == null) {
            return head;
        } else if (head.val == val) {
            return head.next;
        }

        ListNode prev = head, node = head.next;
        while (node != null) {
            if (node.val == val) {
                break;
            }
            prev = node;
            node = node.next;
        }

        prev.next = node == null ? node : node.next;
        return head;
    }
}
```

**2.单指针法**

```java
/**
 * Definition for singly-linked list.
 * public class ListNode {
 * int val;
 * ListNode next;
 * ListNode(int x) { val = x; }
 * }
 */
class Solution {
    public ListNode deleteNode(ListNode head, int val) {
        if (head == null) {
            return head;
        } else if (head.val == val) {
            return head.next;
        }

        ListNode node = head;
        while (node.next != null && node.next.val != val) {
            node = node.next;
        }
        if (node.next != null) {
            node.next = node.next.next;
        }

        return head;
    }
}
```

注意删除的节点在链表头、尾、中间，以及删除节点不存在、链表为空的情况。

**测试用例**

> [4,5,1,9]
> 5
> [4,5,1,9]
> 4
> [4,5,1,9]
> 9
> [4,5,1,9]
> 3
> []
> 3

## [剑指 Offer 22. 链表中倒数第k个节点](https://leetcode.cn/problems/lian-biao-zhong-dao-shu-di-kge-jie-dian-lcof/)

```java
/**
 * Definition for singly-linked list.
 * public class ListNode {
 *     int val;
 *     ListNode next;
 *     ListNode(int x) { val = x; }
 * }
 */
class Solution {
    public ListNode getKthFromEnd(ListNode head, int k) {
        ListNode fast = head, slow = head;

        while (fast != null) {
            fast = fast.next;

            if (k > 0) {
                k--;
            } else {
                slow = slow.next;
            }
        }

        return slow;
    }
}
```

思路：快慢指针解法，有以下两种走法

- 快指针先走k步，然后快慢指针一起走，直到快指针走到 null 位置，慢指针就在倒数第 k 个节点。
- 快指针先走k-1步，然后快慢指针一起走，直到快指针走到最后一个节点位置，慢指针就在倒数第 k 个节点。

## [剑指 Offer 25. 合并两个排序的链表](https://leetcode.cn/problems/he-bing-liang-ge-pai-xu-de-lian-biao-lcof/)

注意：本题与 [21. 合并两个有序链表 - 力扣（LeetCode）](https://leetcode.cn/problems/merge-two-sorted-lists/) 相同

**递归解法**

```java
/**
 * Definition for singly-linked list.
 * public class ListNode {
 * int val;
 * ListNode next;
 * ListNode(int x) { val = x; }
 * }
 */
class Solution {
    public ListNode mergeTwoLists(ListNode l1, ListNode l2) {
        if (l1 == null) {
            return l2;
        } else if (l2 == null) {
            return l1;
        } else if (l1.val < l2.val) {
            l1.next = mergeTwoLists(l1.next, l2);
            return l1;
        } else {
            l2.next = mergeTwoLists(l1, l2.next);
            return l2;
        }
    }
}
```

**双指针迭代法（击败100%）**

```java
/**
 * Definition for singly-linked list.
 * public class ListNode {
 * int val;
 * ListNode next;
 * ListNode(int x) { val = x; }
 * }
 */
class Solution {
    public ListNode mergeTwoLists(ListNode l1, ListNode l2) {
        // 创建一个非空的头节点，方便追加节点
        ListNode head = new ListNode(-1);
        ListNode prev = head;

        while (l1 != null && l2 != null) {
            if (l1.val <= l2.val) {
                prev = prev.next = l1;
                l1 = l1.next;
            } else {
                prev = prev.next = l2;
                l2 = l2.next;
            }
        }

        prev.next = l1 != null ? l1 : l2;
        // 头节点的下一个节点才是真的头节点
        return head.next;
    }
}
```

## [剑指 Offer 52. 两个链表的第一个公共节点](https://leetcode.cn/problems/liang-ge-lian-biao-de-di-yi-ge-gong-gong-jie-dian-lcof/)

**1.双循环法**

```java
/**
 * Definition for singly-linked list.
 * public class ListNode {
 * int val;
 * ListNode next;
 * ListNode(int x) {
 * val = x;
 * next = null;
 * }
 * }
 */
class Solution {
    ListNode getIntersectionNode(ListNode headA, ListNode headB) {
        if (headA == null || headB == null) {
            return null;
        }

        ListNode n1 = headA;
        ListNode n2;
        while (n1 != null) {
            n2 = headB;
            while (n2 != null) {
                // 注意这里比较引用而不是 ListNode 的值
                if (n1 == n2) {
                    return n1;
                }
                n2 = n2.next;
            }
            n1 = n1.next;
        }

        return null;
    }
}
```

**2.哈希大法**

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
        Set<ListNode> visited = new HashSet<>();
        ListNode tmp = headA;
        while (tmp != null) {
            visited.add(tmp);
            tmp = tmp.next;
        }

        tmp = headB;
        while (tmp != null) {
            if (visited.contains(tmp)) {
                return tmp;
            }
            visited.add(tmp);
            tmp = tmp.next;
        }

        return null;
    }
}
```

**3.先计算长度，截掉头部多余的节点，再对比(击败99.61%)**

```java
/**
 * Definition for singly-linked list.
 * public class ListNode {
 * int val;
 * ListNode next;
 * ListNode(int x) {
 * val = x;
 * next = null;
 * }
 * }
 */
class Solution {
    ListNode getIntersectionNode(ListNode headA, ListNode headB) {
        if (headA == null || headB == null) {
            return null;
        }

        int aLength = getLength(headA);
        int bLength = getLength(headB);
        int diff = aLength - bLength;
        ListNode a = headA;
        ListNode b = headB;
        while (diff > 0) {
            a = a.next;
            diff--;
        }
        while (diff < 0) {
            b = b.next;
            diff++;
        }

        while (a != b) {
            a = a.next;
            b = b.next;
        }

        return a;
    }

    int getLength(ListNode node) {
        int len = 0;
        while (node != null) {
            node = node.next;
            len++;
        }
        return len;
    }
}
```

**4.双指针浪漫相遇法(最优，击败99.61%)**

> 两个结点不断的去对方的轨迹中寻找对方的身影，只要二人有交集，就终会相遇❤

a指针遍历A+B，b指针遍历B+A，两指针遍历长度一样，终将相遇。

```java
/**
 * Definition for singly-linked list.
 * public class ListNode {
 * int val;
 * ListNode next;
 * ListNode(int x) {
 * val = x;
 * next = null;
 * }
 * }
 */
class Solution {
    ListNode getIntersectionNode(ListNode headA, ListNode headB) {
        if (headA == null || headB == null) {
            return null;
        }

        ListNode a = headA, b = headB;
        while (a != b) {
            a = a == null ? headB : a.next;
            b = b == null ? headA : b.next;
        }
        return a;
    }
}
```

**测试用例**

>8
>[4,1,8,4,5]
>[5,0,1,8,4,5]
>2
>3
>2
>[0,9,1,2,4]
>[3,2,4]
>3
>1
>0
>[2,6,4]
>[1,5]
>3
>2

## [剑指 Offer 21. 调整数组顺序使奇数位于偶数前面](https://leetcode.cn/problems/diao-zheng-shu-zu-shun-xu-shi-qi-shu-wei-yu-ou-shu-qian-mian-lcof/)

类似于快速排序的左右双指针数据交换

```java
class Solution {
    public int[] exchange(int[] nums) {
        int left = 0, right = nums.length - 1;
        while (left < right) {
            while (left < right && ((nums[left] & 0x01) == 1)) {
                left++;
            }
            while (left < right && ((nums[right] & 0x01) == 0)) {
                right--;
            }
            if (left < right) {
                int tmp = nums[left];
                nums[left] = nums[right];
                nums[right] = tmp;
            }
        }
        return nums;
    }
}
```

## [剑指 Offer 57. 和为s的两个数字](https://leetcode.cn/problems/he-wei-sde-liang-ge-shu-zi-lcof/)

注意：本题与 [1. 两数之和 - 力扣（LeetCode）](https://leetcode.cn/problems/two-sum/) 相似但不相同，本题的数组是有序的。

**1.哈希求解法**

```java
class Solution {
    public int[] twoSum(int[] nums, int target) {
        Set<Integer> visited = new HashSet<>();
        for (int num : nums) {
            if (visited.contains(target - num)) {
                return new int[]{num, target - num};
            }
            visited.add(num);
        }
        return new int[0];
    }
}
```

**2.双指针法(击败99.74%)**

因为是有序数组，所以双指针法更优

```java
class Solution {
    public int[] twoSum(int[] nums, int target) {
        int left = 0, right = nums.length - 1;
        while (left < right) {
            int sum = nums[left] + nums[right];
            if (sum == target) {
                return new int[]{nums[left], nums[right]};
            }
            if (sum > target) {
                right--;
            } else {
                left++;
            }
        }
        return new int[0];
    }
}
```

## [剑指 Offer 58 - I. 翻转单词顺序](https://leetcode.cn/problems/fan-zhuan-dan-ci-shun-xu-lcof/)

**1.分割+倒序**

```java
class Solution {
    public String reverseWords(String s) {
        String[] splits = s.trim().split("\\s+");
        StringBuilder sb = new StringBuilder();
        for (int i = splits.length - 1; i >= 0; i--) {
            if (i != splits.length - 1) {
                sb.append(" ");
            }
            sb.append(splits[i]);
        }
        return sb.toString();
    }
}
```

**2.双指针倒序遍历(击败79.46%)**

```java
class Solution {
    public String reverseWords(String s) {
        s = s.trim();
        int left = s.length() - 1, right = left;
        StringBuilder sb = new StringBuilder();
        while (left >= 0) {
            while (left >= 0 && s.charAt(left) != ' ') left--;// 搜索首个空格
            sb.append(s.substring(left + 1, right + 1) + " ");// 添加单词
            while (left >= 0 && s.charAt(left) == ' ') left--;// 跳过单词间空格
            right = left;// 指向下一个单词尾字符
        }
        return sb.toString().trim();
    }
}
```

# 栈和队列

## [剑指 Offer 09. 用两个栈实现队列](https://leetcode.cn/problems/yong-liang-ge-zhan-shi-xian-dui-lie-lcof/)

Java 中的栈请用 LinkedList 来代替 Stack。

```java
class CQueue {

    private LinkedList<Integer> appendStack;
    private LinkedList<Integer> deleteStack;

    public CQueue() {
        appendStack = new LinkedList<>();
        deleteStack = new LinkedList<>();
    }
    
    public void appendTail(int value) {
        appendStack.push(value);
    }
    
    public int deleteHead() {
        if (deleteStack.isEmpty()) {
            while (!appendStack.isEmpty()) {
                deleteStack.push(appendStack.pop());
            }
        }
        if (deleteStack.isEmpty()) {
            return -1;
        }
        return deleteStack.pop();
    }
}

/**
 * Your CQueue object will be instantiated and called as such:
 * CQueue obj = new CQueue();
 * obj.appendTail(value);
 * int param_2 = obj.deleteHead();
 */
```

## [剑指 Offer 30. 包含min函数的栈](https://leetcode.cn/problems/bao-han-minhan-shu-de-zhan-lcof/)

加个辅助栈，每次 push 时也 push 当前 min 到辅助栈栈顶。

```java
import java.util.LinkedList;

class MinStack {

    private LinkedList<Integer> stack;
    private LinkedList<Integer> minStack;

    /**
     * initialize your data structure here.
     */
    public MinStack() {
        stack = new LinkedList<>();
        minStack = new LinkedList<>();
    }

    public void push(int x) {
        stack.push(x);
        if (minStack.isEmpty()) {
            minStack.push(x);
        } else {
            int min = minStack.peek();
            minStack.push(min > x ? x : min);
        }
    }

    public void pop() {
        stack.pop();
        minStack.pop();
    }

    public int top() {
        return stack.peek();
    }

    public int min() {
        return minStack.peek();
    }
}

/**
 * Your MinStack object will be instantiated and called as such:
 * MinStack obj = new MinStack();
 * obj.push(x);
 * obj.pop();
 * int param_3 = obj.top();
 * int param_4 = obj.min();
 */
```

优化一下，不用每次都维护辅助栈，只有 push 的元素小于等于当前 min 时，才维护

```java
class MinStack {

    private LinkedList<Integer> stack;
    private LinkedList<Integer> minStack;

    /** initialize your data structure here. */
    public MinStack() {
        stack = new LinkedList<>();
        minStack = new LinkedList<>();
    }
    
    public void push(int x) {
        stack.push(x);
        // 注意这里的条件是“>= x”，不是“>x”，否则遇到多个 min 值时会出问题
        if (minStack.isEmpty() || minStack.peek() >= x) {
            minStack.push(x);
        }
    }
    
    public void pop() {
        if (stack.pop().equals(minStack.peek())) {
            minStack.pop();
        }
    }
    
    public int top() {
        return stack.peek();
    }
    
    public int min() {
        return minStack.peek();
    }
}

/**
 * Your MinStack object will be instantiated and called as such:
 * MinStack obj = new MinStack();
 * obj.push(x);
 * obj.pop();
 * int param_3 = obj.top();
 * int param_4 = obj.min();
 */
```

## [剑指 Offer 59 - I. 滑动窗口的最大值](https://leetcode.cn/problems/hua-dong-chuang-kou-de-zui-da-zhi-lcof/)

知识点：单调队列

建议先完成 [剑指 Offer 59 - II. 队列的最大值](https://leetcode.cn/problems/dui-lie-de-zui-da-zhi-lcof/) 以明白单调队列如何维护。



设数组 nums 长度为 n，则共有 n - k + 1 个窗口。

需要维护一个单调递减队列，来存储最大值。当添加的值比队列中的值要大时，要先清空队列，然后再添加，以保证队列头是最大值且队列是单调递减。

```java
class Solution {
    public int[] maxSlidingWindow(int[] nums, int k) {
        if (nums == null || nums.length == 0) {
            return nums;
        }
        Deque<Integer> queue = new LinkedList<>();
        // 初始化窗口
        for (int i = 0; i < k; i++) {
            // 维护单调队列
            while (!queue.isEmpty() && queue.peekLast() < nums[i]) {
                queue.removeLast();
            }
            queue.addLast(nums[i]);
        }
        int[] res = new int[nums.length - k + 1];
        res[0] = queue.getFirst();

        // 开始滑动窗口
        for (int i = k; i < nums.length; i++) {
            // 窗口移动时，要删除最左值
            if (queue.getFirst() == nums[i - k]) {
                queue.removeFirst();
            }
            // 维护单调队列
            while (!queue.isEmpty() && queue.peekLast() < nums[i]) {
                queue.removeLast();
            }
            queue.addLast(nums[i]);
            res[i - k + 1] = queue.getFirst();
        }

        return res;
    }
}
```

## [剑指 Offer 59 - II. 队列的最大值](https://leetcode.cn/problems/dui-lie-de-zui-da-zhi-lcof/)

知识点：单调队列

**FIFO + 双向队列（单调队列）**

```java
class MaxQueue {

    private Queue<Integer> queue;
    private Deque<Integer> max;

    public MaxQueue() {
        queue = new LinkedList<Integer>();
        max = new LinkedList<Integer>();
    }

    public int max_value() {
        if (max.isEmpty()) {
            return -1;
        }
        return max.getFirst();
    }

    public void push_back(int value) {
        queue.add(value);
        while (!max.isEmpty() && max.getLast() < value) {
            max.removeLast();
        }
        max.add(value);
    }

    public int pop_front() {
        if (queue.isEmpty()) {
            return -1;
        }
        int first = queue.remove();
        if (max.getFirst() == first) {
            max.removeFirst();
        }
        return first;
    }
}

/**
 * Your MaxQueue object will be instantiated and called as such:
 * MaxQueue obj = new MaxQueue();
 * int param_1 = obj.max_value();
 * obj.push_back(value);
 * int param_3 = obj.pop_front();
 */
```



# *模拟

## [剑指 Offer 29. 顺时针打印矩阵](https://leetcode.cn/problems/shun-shi-zhen-da-yin-ju-zhen-lcof/)

正方形矩阵、长方形矩阵

## [剑指 Offer 31. 栈的压入、弹出序列](https://leetcode.cn/problems/zhan-de-ya-ru-dan-chu-xu-lie-lcof/)

# 查找算法

## [剑指 Offer 03. 数组中重复的数字](https://leetcode.cn/problems/shu-zu-zhong-zhong-fu-de-shu-zi-lcof/)

> 找出数组中重复的数字。
> 在一个长度为 n 的数组 nums 里的所有数字都在 0～n-1 的范围内。数组中某些数字是重复的，但不知道有几个数字重复了，也不知道每个数字重复了几次。请找出数组中任意一个重复的数字。

知识点：哈希表、原地交换

原地交换：因为数字都在 0～n-1 的范围内，所以直接将数字交换到对应下标下，相同说明有重复

```java
class Solution {
    public int findRepeatNumber(int[] nums) {
        for (int i = 0; i < nums.length; i++) {
            int num = nums[i];
            while (i != num) {
                if (nums[num] == num) {
                    return num;
                }
                int tmp = nums[num];
                nums[num] = num;
                nums[i] = num = tmp;
            }
        }
        return -1;
    }
}
```

一个循环就搞定的写法：

```java
class Solution {
    public int findRepeatNumber(int[] nums) {
        int i = 0;
        while (i < nums.length) {
            if (nums[i] == i) {
                i++;
                continue;
            }
            if (nums[nums[i]] == nums[i]) {
                return nums[i];
            }
            int tmp = nums[i];
            nums[i] = nums[tmp];// 易写错成“nums[nums[i]]”
            nums[tmp] = tmp;
        }
        return -1;
    }
}
```

## [*剑指 Offer 53 - I. 在排序数组中查找数字 I](https://leetcode.cn/problems/zai-pai-xu-shu-zu-zhong-cha-zhao-shu-zi-lcof/)

> 中等难度题目
>
> 统计一个数字在排序数组中出现的次数。

**注意：**本题与 [34. 在排序数组中查找元素的第一个和最后一个位置 - 力扣（LeetCode）](https://leetcode.cn/problems/find-first-and-last-position-of-element-in-sorted-array/) 相同（仅返回值不同）

建议先完成[剑指 Offer 53 - II. 0～n-1中缺失的数字](https://leetcode.cn/problems/que-shi-de-shu-zi-lcof/)，再做这道题。

知识点：二分查找法找边界

**我的解法**

```java
class Solution {
    public int search(int[] nums, int target) {
        if (nums.length == 0) {
            return 0;
        }
        if (nums.length == 1) {
            return nums[0] == target ? 1 : 0;
        }

        if (nums[0] <= target && nums[nums.length - 1] >= target) {
            int min = nums.length >> 1;
            int left = search(Arrays.copyOfRange(nums, 0, min), target);
            int right = search(Arrays.copyOfRange(nums, min, nums.length), target);
            return left + right;
        } else {
            return 0;
        }
    }
}
```

**二分查找法找边界**

先找到 target 的左右边界 lo 和 hi， 然后 hi - lo - 1 即为 target 出现的次数

```java
class Solution {
    public int search(int[] nums, int target) {
        int i = 0, j = nums.length - 1;
        // 查找右边界
        while (i <= j) {
            int mid = (i + j) >> 1;
            if (nums[mid] <= target) {
                i = mid + 1;// i要定位到target的右边
            } else {
                j = mid - 1;
            }
        }
        int hi = i;

        // 查找左边界
        i = 0;
        j = nums.length - 1;
        while (i <= j) {
            int mid = (i + j) >> 1;
            if (nums[mid] >= target) {
                j = mid - 1;// j要定位到target的左边
            } else {
                i = mid + 1;
            }
        }
        int lo = j;

        return hi - lo - 1;
    }
}
```

## [剑指 Offer 53 - II. 0～n-1中缺失的数字](https://leetcode.cn/problems/que-shi-de-shu-zi-lcof/)

知识点：二分查找法找边界、哈希表、数学求和、位运算异或

将数组划分成 `array[i] == i` 和 `array[i] != i` 左右两个数组，缺失的数字位于右数组首位。

**我的解法**

```java
class Solution {
    public int missingNumber(int[] nums) {
        // 补充下面二分查找法覆盖不了的情况
        if (nums[0] == 1) {
            return 0;
        } else if (nums[nums.length - 1] == nums.length - 1) {
            return nums.length;
        }

        int i = 0, j = nums.length - 1;
        // 查找右数组首位
        while (i < j) {
            int mid = (i + j) >> 1;
            if (nums[mid] == mid) {
                i = mid + 1;
            } else {
                j = mid;
            }
        }
        return j;
    }
}
```

**更优的解法**

```java
class Solution {
    public int missingNumber(int[] nums) {
        int i = 0, j = nums.length - 1;
        while (i <= j) {// i > j 时退出
            int mid = (i + j) >> 1;
            if (nums[mid] == mid) {
                i = mid + 1;// 左指针查找右数组首位
            } else {
                j = mid - 1;// 右指针查找左数组末位
            }
        }
        return i;// 返回右数组首位
    }
}
```

**测试用例**

> [0,1,3]
> [0,1,2,3,4,5,6,7,9]
> [0]
> [0,1]

## [-剑指 Offer 04. 二维数组中的查找](https://leetcode.cn/problems/er-wei-shu-zu-zhong-de-cha-zhao-lcof/)

## [剑指 Offer 11. 旋转数组的最小数字](https://leetcode.cn/problems/xuan-zhuan-shu-zu-de-zui-xiao-shu-zi-lcof/)

> 困难级别题目

注意：本题与 [154. 寻找旋转排序数组中的最小值 II](https://leetcode.cn/problems/find-minimum-in-rotated-sorted-array-ii/) 相同

比原书解法最简单的写法，中间数和**右数**比，来判断最小值位于哪个区间。

```java
class Solution {
    public int minArray(int[] numbers) {
        int i = 0, j = numbers.length - 1;
        while (i < j) {
            int mid = (i + j) >> 1;
            if (numbers[mid] < numbers[j]) {
                j = mid;
            } else if (numbers[mid] > numbers[j]) {
                i = mid + 1;
            } else {
                j--;
            }
        }

        return numbers[j];
    }
}
```

## [剑指 Offer 50. 第一个只出现一次的字符](https://leetcode.cn/problems/di-yi-ge-zhi-chu-xian-yi-ci-de-zi-fu-lcof/)

知识点：有序哈希表

因为只出现一次的字符可能会有很多个，要找到第一个，就需要记录统计的顺序。

```java
class Solution {
    public char firstUniqChar(String s) {
      Map<Character, Boolean> map = new LinkedHashMap<>();
      for (int i = 0; i < s.length(); i++) {
        char c = s.charAt(i);
        map.put(c, map.containsKey(c));
      }

      for (Map.Entry<Character, Boolean> entry : map.entrySet()) {
        if (!entry.getValue()) {
          return entry.getKey();
        }
      }

      return ' ';
    }
}
```

# 搜索与回溯算法

## [剑指 Offer 32 - I. 从上到下打印二叉树](https://leetcode.cn/problems/cong-shang-dao-xia-da-yin-er-cha-shu-lcof/)

知识点：层序遍历 BFS

## [剑指 Offer 32 - II. 从上到下打印二叉树 II](https://leetcode.cn/problems/cong-shang-dao-xia-da-yin-er-cha-shu-ii-lcof/)

## [剑指 Offer 32 - III. 从上到下打印二叉树 III](https://leetcode.cn/problems/cong-shang-dao-xia-da-yin-er-cha-shu-iii-lcof/)

## [剑指 Offer 26. 树的子结构](https://leetcode.cn/problems/shu-de-zi-jie-gou-lcof/)

## [剑指 Offer 27. 二叉树的镜像](https://leetcode.cn/problems/er-cha-shu-de-jing-xiang-lcof/)

## [剑指 Offer 28. 对称的二叉树](https://leetcode.cn/problems/dui-cheng-de-er-cha-shu-lcof/)

测试用例

> [1,2,2,2,null,2]
> []

## [剑指 Offer 12. 矩阵中的路径](https://leetcode.cn/problems/ju-zhen-zhong-de-lu-jing-lcof)

知识点：深度遍历 DFS

遍历矩阵中的每个元素，找到与目标第一个元素相同的元素后，开始进行上下左右四个路径的搜索。遇到越界和不相同的元素时，停止搜索。

注意标记访问过的元素，防止重复访问；递归访问结束后还要恢复，防止影响其它路径访问和下一轮访问

```java
class Solution {
    private char[] chars;

    public boolean wordPuzzle(char[][] grid, String target) {
        if (grid.length == 0 || grid[0].length == 0) {
            return false;
        }
        chars = target.toCharArray();

        for (int j = 0; j < grid.length; j++) {
            for (int i = 0; i < grid[0].length; i++) {
                if (dfs(grid, i, j, 0)) {
                    return true;
                }
            }
        }
        return false;
    }

    private boolean dfs(char[][] grid, int i, int j, int t) {
        if (j < 0 || j >= grid.length || i < 0 || i >= grid[0].length || grid[j][i] != chars[t]) {
            return false;
        }
        if (++t == chars.length) {
            return true;
        }

        grid[j][i] = ' ';
        boolean res = dfs(grid, i - 1, j, t) || dfs(grid, i + 1, j, t)
                || dfs(grid, i, j - 1, t) || dfs(grid, i, j + 1, t);
        grid[j][i] = chars[t - 1];
        return res;
    }
}
```

注意，如果后面那段代码写成这样会导致超时

```java
grid[j][i] = ' ';

boolean left = dfs(grid, i - 1, j, t);
boolean right = dfs(grid, i + 1, j, t);
boolean up = dfs(grid, i, j - 1, t);
boolean down = dfs(grid, i, j + 1, t);

grid[j][i] = chars[t - 1];
return left || right || up || down;
```

## [剑指 Offer 13. 机器人的运动范围](https://leetcode.cn/problems/ji-qi-ren-de-yun-dong-fan-wei-lcof)

知识点：深度遍历 DFS

从(0,0)开始进行上下左右四个方向的搜索，每到达一个能访问的地方，+1，并且标记该地方，避免重复访问。

```java
class Solution {
    public int movingCount(int threshold, int rows, int cols) {
        boolean[][] visited = new boolean[rows][cols];
        return dfs(threshold, rows, cols, 0, 0, visited);
    }

    private int dfs(int threshold, int rows, int cols, int i, int j,
                    boolean[][] visited) {
        if (i < 0 || i >= rows || j < 0 || j >= cols || visited[i][j] ||
                !canVisit(i, j, threshold)) {
            return 0;
        }
        visited[i][j] = true;
        return dfs(threshold, rows, cols, i + 1, j, visited)
               + dfs(threshold, rows, cols, i - 1, j, visited)
               + dfs(threshold, rows, cols, i, j + 1, visited)
               + dfs(threshold, rows, cols, i, j - 1, visited)
               + 1;
    }

    private boolean canVisit(int i, int j, int threshold) {
        int sum = 0;
        while (i != 0) {
            sum += i % 10;
            i /= 10;
        }
        while (j != 0) {
            sum += j % 10;
            j /= 10;
        }
        return sum <= threshold;
    }
}
```

## [-剑指 Offer 34. 二叉树中和为某一值的路径](https://leetcode.cn/problems/er-cha-shu-zhong-he-wei-mou-yi-zhi-de-lu-jing-lcof/)

## [-剑指 Offer 36. 二叉搜索树与双向链表](https://leetcode.cn/problems/er-cha-sou-suo-shu-yu-shuang-xiang-lian-biao-lcof/)

## [-剑指 Offer 54. 二叉搜索树的第 k 大节点](https://leetcode.cn/problems/er-cha-sou-suo-shu-de-di-kda-jie-dian-lcof/)

有更优的解法，避免O(n)

## [剑指 Offer 55 - I. 二叉树的深度](https://leetcode.cn/problems/er-cha-shu-de-shen-du-lcof/)

```java
/**
 * Definition for a binary tree node.
 * public class TreeNode {
 *     int val;
 *     TreeNode left;
 *     TreeNode right;
 *     TreeNode() {}
 *     TreeNode(int val) { this.val = val; }
 *     TreeNode(int val, TreeNode left, TreeNode right) {
 *         this.val = val;
 *         this.left = left;
 *         this.right = right;
 *     }
 * }
 */
class Solution {
    public int calculateDepth(TreeNode root) {
        if (root == null) {
            return 0;
        }
        return Math.max(calculateDepth(root.left), calculateDepth(root.right)) + 1;
    }
}
```

## [*剑指 Offer 55 - II. 平衡二叉树](https://leetcode.cn/problems/ping-heng-er-cha-shu-lcof/)

还有优化空间

需要满足以下条件：

- 左右子树高度相差不超过1
- 左右子树是平衡二叉树，不能是链表

注意空树和只有一个节点的树也是平衡二叉树。

```java
/**
 * Definition for a binary tree node.
 * public class TreeNode {
 *     int val;
 *     TreeNode left;
 *     TreeNode right;
 *     TreeNode() {}
 *     TreeNode(int val) { this.val = val; }
 *     TreeNode(int val, TreeNode left, TreeNode right) {
 *         this.val = val;
 *         this.left = left;
 *         this.right = right;
 *     }
 * }
 */
class Solution {
    public boolean isBalanced(TreeNode root) {
        if (root == null) {
            return true;
        }
        int minus = maxDepth(root.left) - maxDepth(root.right);
        return minus >= -1 && minus <= 1 && isBalanced(root.left) && isBalanced(root.right);
    }

    public int maxDepth(TreeNode root) {
        if (root == null) {
            return 0;
        }
        return Math.max(maxDepth(root.left), maxDepth(root.right)) + 1;
    }
}
```

测试用例

> []
> [1]
> [2,1,3]
> [1,2,2,3,null,null,3,4,null,null,4]

## [-剑指 Offer 64. 求 1 + 2 + … + n](https://leetcode.cn/problems/qiu-12n-lcof/)

## [剑指 Offer 68 - I. 二叉搜索树的最近公共祖先](https://leetcode.cn/problems/er-cha-sou-suo-shu-de-zui-jin-gong-gong-zu-xian-lcof/)

还有优化空间

利用二叉搜索树有序性，即小于根节点的数位于左边、大于的位于右边。

- 如果两个数一个位于左边、一个位于右边，则根节点为祖先
- 如果两个数均位于左边，则递归左子树
- 如果两个数均位于右边，则递归右子树

```java
/**
 * Definition for a binary tree node.
 * public class TreeNode {
 *     int val;
 *     TreeNode left;
 *     TreeNode right;
 *     TreeNode(int x) { val = x; }
 * }
 */
class Solution {
    public TreeNode lowestCommonAncestor(TreeNode root, TreeNode p, TreeNode q) {
        if (root.val < p.val && root.val < q.val) {
            return lowestCommonAncestor(root.right, p, q);
        } else if (root.val > p.val && root.val > q.val) {
            return lowestCommonAncestor(root.left, p, q);
        }
        return root;
    }
}
```

## [*剑指 Offer 68 - II. 二叉树的最近公共祖先](https://leetcode.cn/problems/er-cha-shu-de-zui-jin-gong-gong-zu-xian-lcof/)

还有优化空间

有以下情况

- 两个节点位于不同子树，则根节点是最近公共祖先
- 两个节点都位于同一子树，则有以下情况
  - 两个节点位于子树的不同子树，则子树的根节点作为最近公共祖先
  - 两个节点位于子树的同一个子树，则其中一个作为最近公共祖先

所以只需要递归遍历左右子树，直到找到其中一个节点则返回该节点作为祖先，或遇到空节点则直接返回。

```java
/**
 * Definition for a binary tree node.
 * public class TreeNode {
 *     int val;
 *     TreeNode left;
 *     TreeNode right;
 *     TreeNode(int x) { val = x; }
 * }
 */
class Solution {
    public TreeNode lowestCommonAncestor(TreeNode root, TreeNode p, TreeNode q) {
        if (root == null || root.val == p.val || root.val == q.val) {
            return root;
        }

        TreeNode left = lowestCommonAncestor(root.left, p, q);
        TreeNode right = lowestCommonAncestor(root.right, p, q);

        if (left == null) {
            return right;
        } else if (right == null) {
            return left;
        } else {
            return root;
        }
    }
}
```

## [-剑指 Offer 37. 序列化二叉树](https://leetcode.cn/problems/xu-lie-hua-er-cha-shu-lcof/)

知识点：层序遍历 BFS

## [剑指 Offer 38. 字符串的排列](https://leetcode.cn/problems/zi-fu-chuan-de-pai-lie-lcof/)

知识点：全排列 树的遍历

全排列就是用树来遍历各种排列情况，对同一层的重复节点进行剪枝，同时下层不能选择上层已选择的字符

**1.遍历法**

```java
class Solution {
    private char[] chars;
    private List<String> res = new ArrayList<>();
    private char[] path;// 遍历的路径
    private boolean[] selected;// 控制下层不选择上层已选的字符

    public String[] goodsOrder(String goods) {
        chars = goods.toCharArray();
        path = new char[chars.length];
        selected = new boolean[chars.length];
        dfs(0);
        return res.toArray(new String[res.size()]);
    }

    private void dfs(int x) {
        if (x == chars.length) {// 路径长度等于字符串长度时，结束
            res.add(new String(path));
            return;
        }
        Set<Character> set = new HashSet<>();// 控制同一层字符不重复
        for (int i = 0; i < chars.length; i++) {
            // 不能选择上层已选的字符，也不能选择本层重复的字符
            if (set.contains(chars[i]) || selected[i]) {
                continue;
            }
            set.add(chars[i]);
            selected[i] = true;
            path[x] = chars[i];
            dfs(x + 1);// 递归选择路径下一个字符
            selected[i] = false;
        }
    }
}
```

**2.置换法(97.57%)**

```java
class Solution {
    private char[] chars;
    private List<String> res = new ArrayList<>();

    public String[] goodsOrder(String goods) {
        chars = goods.toCharArray();
        dfs(0);
        return res.toArray(new String[res.size()]);
    }

    private void dfs(int x) {
        if (x == chars.length - 1) {// 剩余1个字符时不用交换
            res.add(new String(chars));
            return;
        }
        Set<Character> set = new HashSet<>();// 控制本层不选择重复字符
        for (int i = x; i < chars.length; i++) {
            if (set.contains(chars[i])) {
                continue;
            }
            set.add(chars[i]);
            swap(i, x);// 将i位置字符交换到x位置
            dfs(x + 1);// 继续下一个位置
            swap(i, x);// 恢复交换
        }
    }

    private void swap(int i, int x) {
        char tmp = chars[i];
        chars[i] = chars[x];
        chars[x] = tmp;
    }
}
```

单元测试

> ""
> "aab"
> "aba"

# 分治算法

## [-剑指 Offer 07. 重建二叉树](https://leetcode.cn/problems/zhong-jian-er-cha-shu-lcof/)

知识点：前序遍历、中序遍历

有优解

## [剑指 Offer 16. 数值的整数次方](https://leetcode.cn/problems/shu-zhi-de-zheng-shu-ci-fang-lcof/)

知识点：快速幂

- n为0时，result = 1
- n为1时，result = x
- n为-1时，result = 1/x
- n为其它值时，包括正负，因为计算机奇数和偶数整除有差异，分以下情况：
  - n为偶数：result = x^n/2 * x^n/2
  - n为奇数：result = x^n/2 * x^n/2 * x

```java
class Solution {
    public double myPow(double x, int n) {
        if (n == 0) {
            return 1;
        } else if (n == 1) {
            return x;
        } else if (n == -1) {
            return 1 / x;
        }

        double half = myPow(x, n >> 1);
        double remain = myPow(x, n & 1);
        return half * half * remain;
    }
}
```

## [剑指 Offer 33. 二叉搜索树的后序遍历序列](https://leetcode.cn/problems/er-cha-sou-suo-shu-de-hou-xu-bian-li-xu-lie-lcof/)

知识点：递归、后序遍历、单调队列

方法：

- 先找根节点
- 再找左右子树序列的中间节点，判断右子树序列是否全部大于根节点
- 递归左右子树序列，判断是否也满足上面两个条件

**我的代码**

```java
class Solution {
    public boolean verifyTreeOrder(int[] postorder) {
        return verify(postorder, 0, postorder.length - 1);
    }

    public boolean verify(int[] postorder, int left, int right) {
        if (left >= right) {
            return true;
        }
        int mid = right;
        for (int i = left; i < right; i++) {
            if (postorder[i] > postorder[right]) {
                mid = i;
                break;
            }
        }
        for (int i = mid; i < right; i++) {
            if (postorder[i] < postorder[right]) {
                return false;
            }
        }
        return verify(postorder, left, mid - 1) && verify(postorder, mid, right - 1);
    }
}
```

**简化版**

```java
class Solution {
    public boolean verifyTreeOrder(int[] postorder) {
        return verify(postorder, 0, postorder.length - 1);
    }

    public boolean verify(int[] postorder, int left, int right) {
        if (left >= right) {
            return true;
        }
        int i = left;
        while (postorder[i] < postorder[right]) {
            i++;
        }
        int mid = i;
        while (postorder[i] > postorder[right]) {
            i++;
        }
        return i == right && verify(postorder, left, mid - 1) && verify(postorder, mid, right - 1);
    }
}
```

## [*剑指 Offer 17. 打印从1到最大的n位数](https://leetcode.cn/problems/da-yin-cong-1dao-zui-da-de-nwei-shu-lcof/)

知识点：大数问题，字符串模拟加法

因为这里不用考虑大数问题，所以此题没啥意义。

```java
class Solution {
    public int[] printNumbers(int n) {
        int max = 1;
        for (int i = 0; i < n; i++) {
            max *= 10;
        }
        int[] res = new int[max - 1];
        for (int i = 1; i < max; i++) {
            res[i - 1] = i;
        }
        return res;
    }
}
```

## [剑指 Offer 51. 数组中的逆序对](https://leetcode.cn/problems/shu-zu-zhong-de-ni-xu-dui-lcof/)

知识点：归并排序

利用归并排序最后的归并和排序步骤来统计逆序对

```java
class Solution {
    public int reversePairs(int[] record) {
        return mergeSort(record, 0, record.length - 1);
    }

    private int mergeSort(int[] record, int left, int right) {
        if (left >= right) {
            return 0;
        }
        int mid = (left + right) >> 1;
        return mergeSort(record, left, mid) 
            + mergeSort(record, mid + 1, right) 
            + merge(record, left, mid, right);
    }

    private int merge(int[] record, int left, int mid, int right) {
        int[] tmp = new int[right - left + 1];
        int i = 0, j = left, k = mid + 1;
        int reversePairs = 0;
        while (j <= mid && k <= right) {
            if (record[j] > record[k]) {
                tmp[i++] = record[k++];
                reversePairs += mid - j + 1;
            } else {
                tmp[i++] = record[j++];
            }
        }
        while (j <= mid) {
            tmp[i++] = record[j++];
        }
        while (k <= right) {
            tmp[i++] = record[k++];
        }
        for (i = 0; i < tmp.length; i++) {
            record[left + i] = tmp[i];
        }
        return reversePairs;
    }
}
```

# 排序

## [剑指 Offer 45. 把数组排成最小的数](https://leetcode.cn/problems/ba-shu-zu-pai-cheng-zui-xiao-de-shu-lcof/)

知识点：快速排序、自定义排序

基于快速排序实现自定义排序

还有优化空间

## [剑指 Offer 61. 扑克牌中的顺子](https://leetcode.cn/problems/bu-ke-pai-zhong-de-shun-zi-lcof/)

**我的解法**

```java
class Solution {
    public boolean isStraight(int[] nums) {
        Arrays.sort(nums);

        int count = 0;
        int prev = -1;
        for (int num : nums) {
            if (num == 0) {
                count++;
                continue;
            }
            if (prev == num) {
                return false;
            }
            if (prev != -1) {
                count -= num - prev - 1;
            }
            if (count < 0) {
                return false;
            }
            prev = num;
        }
        return true;
    }
}
```

**简化版**

- 先 Arrays.sort 排序
- 满足以下两个条件即为顺子
  - 除了大小王，其它牌不能重复
  - **最大牌 - 最小牌 < 5** （帅炸了这个判断条件！）


```java
class Solution {
    public boolean isStraight(int[] nums) {
        Arrays.sort(nums);

        int jokers = 0;
        for (int i = 0; i < nums.length - 1; i++) {
            if (nums[i] == 0) {
                jokers++;
                continue;
            }
            if (nums[i] == nums[i + 1]) {
                return false;
            }
        }
        return nums[4] - nums[jokers] < 5;
    }
}
```

## [*剑指 Offer 40. 最小的k个数](https://leetcode.cn/problems/zui-xiao-de-kge-shu-lcof/)

考点：快速排序

思路1：先排序，后取最小的k个数

```java
class Solution {
    public int[] getLeastNumbers(int[] arr, int k) {
        quickSort(arr, 0, arr.length - 1);
        return Arrays.copyOf(arr, k);
    }

    private void quickSort(int[] arr, int left, int right) {
        // 容易漏掉该判断
        if (left >= right) {
            return;
        }

        // 一般以左边为基准
        int i = left, j = right, pivot = arr[left];
        // i==j 时，跳出循环
        while (i < j) {
            // 以左边为基准，则先从右边遍历，反则反之
            while (i < j && arr[j] >= pivot) {
                j--;
            }

            while (i < j && arr[i] <= pivot) {
                i++;
            }

            swap(arr, i, j);
        }
        // i==j 时，与基准位置元素互换
        swap(arr, i, left);

        quickSort(arr, left, i - 1);
        quickSort(arr, i + 1, right);
    }

    private void swap(int[] arr, int a, int b) {
        int tmp = arr[a];
        arr[a] = arr[b];
        arr[b] = tmp;
    }
}
```

快速排序还有个简化的写法，可以少写个交换方法：

```java
// 初始值 left=0, right=array.length-1
private void quickSort(int[] array, int left, int right) {
    // 这个判断容易漏
    if (left >= right) return;

    int i = left, j = right, pivot = array[left];
    while (i < j) {
        while (i < j && array[j] >= pivot) {
            j--;
        }
        // 将j交换到i位置，初始i位置为基准数位置
        if (i < j) array[i++] = array[j];

        while (i < j && array[i] <= pivot) {
            i++;
        }
        // 将i交换到j位置
        if (i < j) array[j] = array[i];
    }
    // 将基准数交换到中间位置
    array[i] = pivot;

    quickSort(array, left, i - 1);
    quickSort(array, i + 1, right);
}
```

快速排序的时间复杂度是 *O(nlogn)*，最坏情况下是 *O(n^2)*，空间复杂度是 *O(logn)*

思路2：因为快速排序时，所有小于等于基准数的数放在左边，大于基准数的数放在右边，基于这个特性，无需全部排序，只需基准数刚好在 k+1 的位置，则其左边的数就是最小的k个数。

## [剑指 Offer 41. 数据流中的中位数](https://leetcode.cn/problems/shu-ju-liu-zhong-de-zhong-wei-shu-lcof/)

知识点：优先级队列

# 动态规划

## [剑指 Offer 10- I. 斐波那契数列](https://leetcode.cn/problems/fei-bo-na-qi-shu-lie-lcof/)

```java
class Solution {
    public int fib(int n) {
        if (n < 2) {
            return n;
        }

        int[] dp = new int[n + 1];
        dp[0] = 0;
        dp[1] = 1;

        for (int i = 2; i <= n; i++) {
            dp[i] = (dp[i - 1] + dp[i - 2]) % 1000000007;
        }

        return dp[n];
    }
}
```

## [剑指 Offer 10- II. 青蛙跳台阶问题](https://leetcode.cn/problems/qing-wa-tiao-tai-jie-wen-ti-lcof/)

注意：本题与 [70. 爬楼梯 - 力扣（LeetCode）](https://leetcode.cn/problems/climbing-stairs/) 相同

```java
class Solution {
    public int numWays(int n) {
        if (n < 2) {
            return 1;
        }

        int[] dp = new int[n + 1];
        dp[0] = 1;
        dp[1] = 1;

        for (int i = 2; i <= n; i++) {
            dp[i] = (dp[i - 1] + dp[i - 2]) % 1000000007;
        }

        return dp[n];
    }
}
```

递推公式：*f(n)=f(n-1)+f(n-2)*，跟斐波那契数列一样，只是起始数字不同

- 青蛙跳台阶问题： *f(0)=1, f(1)=1, f(2)=2*

- 斐波那契数列问题：  *f(0)=0, f(1)=1, f(2)=1*

## [剑指 Offer 63. 股票的最大利润](https://leetcode.cn/problems/gu-piao-de-zui-da-li-run-lcof/)

**注意：**本题与 [121. 买卖股票的最佳时机 - 力扣（LeetCode）](https://leetcode.cn/problems/best-time-to-buy-and-sell-stock/) 相同

```java
class Solution {
    public int bestTiming(int[] prices) {
        // dp[i] = max(dp[i - 1], prices[i] - minPrice);
        // dp[0] = 0;
        int minPrice = Integer.MAX_VALUE;
        int dp = 0;
        for (int price : prices) {
            minPrice = Math.min(minPrice, price);
            dp = Math.max(dp, price - minPrice);
        }
        return dp;
    }
}
```

## [剑指 Offer 42. 连续子数组的最大和](https://leetcode.cn/problems/lian-xu-zi-shu-zu-de-zui-da-he-lcof/)

注意：本题与 [53. 最大子数组和](https://leetcode.cn/problems/maximum-subarray/) 相同

>  **如果你和我加在一起能让我变得更好，那我们就在一起，否则我就丢下你，自己往前走**
>
> **如果前途和爱情二选一，毫不犹豫选前途~**

状态转移方程：*dp[i]=max(dp[i-1]+nums[i],nums[i])*，表示连续子数组的和

```java
class Solution {
    public int maxSubArray(int[] nums) {
        int max = nums[0];
        for (int i  = 1; i < nums.length; i++) {
            nums[i] = Math.max(nums[i - 1] + nums[i], nums[i]);
            max = Math.max(max, nums[i]);
        }
        return max;
    }
}
```

## [剑指 Offer 47. 礼物的最大价值](https://leetcode.cn/problems/li-wu-de-zui-da-jie-zhi-lcof/)

递推公式： `dp[m-1][n-1] = max(dp[m-2][n-1], dp[m-1][n-2]) + grid[m-1][n-1]`

解题思路：礼物的最大价值，为来自左边累计的最大礼物价值与来自上边累计的最大礼物价值的较大值，加上当前的价值。

需要考虑只有一行或一列的特殊情况。另外因为使用递归会超时，所以这里使用迭代

**我的解法**

```java
public class Solution {
    /**
     * 代码中的类名、方法名、参数名已经指定，请勿修改，直接返回方法规定的值即可
     *
     * @param grid int整型二维数组
     * @return int整型
     */
    public int maxValue(int[][] grid) {
        if (grid.length == 0 || grid[0].length == 0) {
            return 0;
        }
        // write code here
        // dp[m-1][n-1] = max(dp[m-2][n-1], dp[m-1][n-2]) + grid[m-1][n-1]
        int[][] dp = new int[grid.length][grid[0].length];
        dp[0][0] = grid[0][0];
        for (int i = 1; i < dp.length; i++) {
            dp[i][0] = grid[i][0] + dp[i - 1][0];
        }
        for (int j = 1; j < dp[0].length; j++) {
            dp[0][j] = grid[0][j] + dp[0][j - 1];
        }
        for (int i = 1; i < dp.length; i++) {
            for (int j = 1; j < dp[0].length; j++) {
                dp[i][j] = Math.max(dp[i - 1][j], dp[i][j - 1]) + grid[i][j];
            }
        }

        return dp[grid.length - 1][grid[0].length - 1];
    }
}
```

**简化版本**(直接将原数组作为dp数组)

```java
class Solution {
    public int jewelleryValue(int[][] frame) {
        if (frame.length == 0 || frame[0].length == 0) {
            return 0;
        }
        for (int i = 1; i < frame.length; i++) {
            frame[i][0] += frame[i - 1][0];
        }
        for (int j = 1; j < frame[0].length; j++) {
            frame[0][j] += frame[0][j - 1];
        }
        for (int i = 1; i < frame.length; i++) {
            for (int j = 1; j < frame[0].length; j++) {
                frame[i][j] += Math.max(frame[i - 1][j], frame[i][j - 1]);
            }
        }
        return frame[frame.length - 1][frame[0].length - 1];
    }
}
```

测试用例

> []
> [[[]]
> [[1,2,3]]
> [[1],[2],[3]]

## [剑指 Offer 46. 把数字翻译成字符串](https://leetcode.cn/problems/ba-shu-zi-fan-yi-cheng-zi-fu-chuan-lcof/)

知识点：跳台阶

这是跳台阶的特殊情况，因为英文字母只有26个(0-25)，所以遇到“26”以上两位数只有一种翻译结果，并且“0-09”也只有一种翻译结果

- 后两位数在 10-25：`dp[i] = dp[i-1] + dp[i-2]` 
- 后两位数其它情况：`dp[i] = dp[i-1]` 
- 初始值：dp[0]=1, dp[1]=1

**我的代码**

```java
class Solution {
    public int crackNumber(int ciphertext) {
        int len = String.valueOf(ciphertext).length();
        int[] dp = new int[len + 1];
        dp[0] = 1;
        dp[1] = 1;
        for (int i = 2; i <= len; i++) {
            int remain = ciphertext % 100;
            if (remain >= 10 && remain <= 25) {
                dp[i] = dp[i - 1] + dp[i - 2];
            } else {
                dp[i] = dp[i - 1];
            }
            ciphertext = ciphertext / 10;
        }
        return dp[len];
    }
}
```

**简化版本**

```java
class Solution {
    public int crackNumber(int ciphertext) {
        int dp0 = 1, dp1 = 1;
        while (ciphertext != 0) {
            int remain = ciphertext % 100;
            if (remain < 10 || remain > 25) {
                dp0 = dp1;
            } else {
                int tmp = dp1;
                dp1 = dp0 + dp1;
                dp0 = tmp;
            }
            ciphertext /= 10;
        }
        return dp1;
    }
}
```

测试用例

> "27"
> "0"
> "01"
> "010"
> "001"
> "10"
> "20"
> "30"

## [剑指 Offer 48. 最长不含重复字符的子字符串](https://leetcode.cn/problems/zui-chang-bu-han-zhong-fu-zi-fu-de-zi-zi-fu-chuan-lcof/)

注意：本题与 [3. 无重复字符的最长子串 - 力扣（LeetCode）](https://leetcode.cn/problems/longest-substring-without-repeating-characters/) 相同

知识点：哈希表、滑动窗口

解题思路：

- 设滑动窗口左右两个指针分别为 start 和 i，并增加一个哈希表记录重复字符
- start 和 i 初始值为 0，即指向字符串首位
- i 向右遍历字符串，每次遇到重复字符时，如果上一个重复字符在 start 右边或相同位置，记该位置为 x，则 start = x + 1；否则 start 不变
- 字符串长度 = i - start + 1，并增加一个变量记录最大字符串长度

```java
class Solution {
    public int dismantlingAction(String list) {
        int max = 0;
        Map<Character, Integer> map = new HashMap<>();
        for (int i = 0, start = 0; i < list.length(); i++) {
            Integer old = map.put(list.charAt(i), i);
            if (old != null) {
                start = Math.max(old + 1, start);
            }
            max = Math.max(max, i - start + 1);
        }
        return max;
    }
}
```

测试用例

> ""
> "a"
> "dvdf"
> "tmmzuxt"
> "wobgrovw"

## [剑指 Offer 19. 正则表达式匹配](https://leetcode.cn/problems/zheng-ze-biao-da-shi-pi-pei-lcof/)

注意：本题与 [10. 正则表达式匹配 - 力扣（LeetCode）](https://leetcode.cn/problems/regular-expression-matching/) 相同

解题思路：设 `dp[i][j]` 表示长度为 i 字符串和 长度为 j 的正则是否匹配的问题

- `dp[i][0] = i == 0 ? true : false` ：空正则能和空串匹配，但不能和非空串匹配
- `dp[0][j] = p[j] == '*' ? (dp[0][j-2]) : false` ：空串和正则能否匹配看情况，如果正则后面有 `*`，则有可能匹配，还要看 `n*` 前部分正则能否匹配
- `dp[i][j]` ：非空串和非空正分以下情况
  - `p[j] != '*'` 时
    - `dp[i][j] = s[i] == p[j] || p[j] == '.' ? dp[i-1][j-1] : false` 
  - `p[j] == '*'` 时
    - 当 `s[i] == p[j-1] || p[j-1] == '.'` 时 `dp[i][j] = dp[i-1][j]` ，即 `ba` 和 `ba*` 、`ba` 和 `b.*` 能匹配
    - 否则 `dp[i][j] = dp[i][j-2]` ，即 `b` 和 `ba` 能匹配

```java
class Solution {
    public boolean articleMatch(String s, String p) {
        int m = s.length(), n = p.length();
        boolean[][] dp = new boolean[m + 1][n + 1];

        // 初始化dp数组
        dp[0][0] = true;
        for (int j = 2; j <= n; j++) {
            if (p.charAt(j - 1) == '*') {
                dp[0][j] = dp[0][j - 2];
            }
        }

        for (int i = 1; i <= m; i++) {
            for (int j = 1; j <= n; j++) {
                if (p.charAt(j - 1) != '*') {
                    if (s.charAt(i - 1) == p.charAt(j - 1) || p.charAt(j - 1) == '.') {
                        dp[i][j] = dp[i - 1][j - 1];
                    }
                } else {
                    if (s.charAt(i - 1) == p.charAt(j - 2) || p.charAt(j - 2) == '.') {
                        dp[i][j] = dp[i - 1][j] // *前面字符出现1次及以上的情况
                            || dp[i][j - 2];// *前面字符出现0次的情况
                    } else {
                        dp[i][j] = dp[i][j - 2];// *前面字符出现0次的情况
                    }
                }
            }
        }
        return dp[m][n];
    }
}
```

## [剑指 Offer 49. 丑数](https://leetcode.cn/problems/chou-shu-lcof/)

注意：本题与 [264. 丑数 II - 力扣（LeetCode）](https://leetcode.cn/problems/ugly-number-ii/) 相同

三指针解法

```java
class Solution {
    public int nthUglyNumber(int n) {
        int i = 0, j = 0, k = 0, idx = 0;
        int[] res = new int[n];
        res[0] = 1;
        while (idx++ < n - 1) {
            res[idx] = Math.min(Math.min(res[i] * 2, res[j] * 3), res[k] * 5);
            if (res[idx] == res[i] * 2) {
                i++;
            }
            if (res[idx] == res[j] * 3) {
                j++;
            }
            if (res[idx] == res[k] * 5) {
                k++;
            }
        }

        return res[idx - 1];
    }
}
```

## [剑指 Offer 60. n 个骰子的点数](https://leetcode.cn/problems/nge-tou-zi-de-dian-shu-lcof/)

递推公式：`dp(n,s)=dp(n-1,s-1)+dp(n-2,s-2)+dp(n-3,s-3)+dp(n-4,s-4)+dp(n-5,s-5)+dp(n-6,s-6)`

解题思路：

- 确定表达式：设 `dp(n，s)` 为 n 个骰子 s 个点数的排列。

- 确定递推公式：如果第 n 个骰子的点数为 1，需要前 n -1 个骰子的点数为 s - 1；如果第 n 个骰子的点数为 2，需要前 n -1 个骰子的点数为 s - 2；以此类推，得 `dp(n,s)=dp(n-1,s-1)+dp(n-1,s-2)...+dp(n-1,s-6)` 。
- 确定初始值：`dp(1,1)=dp(1,2)=...=dp(1,6) = 1`
- 1个骰子能投出点数范围为1~6，n个骰子能投出点数范围为n~6n，共6n-n+1=5n+1个点数。
- 1个骰子投1点的概率为1/6，n个骰子的概率为1/(6^n)

```java
class Solution {
    public double[] statisticsProbability(int num) {
        int[][] dp = new int[num + 1][num * 6 + 1];
        double[] res = new double[num * 5 + 1];
        double totals = Math.pow(6, num);

        for (int i = 1; i <= 6; i++) {
            dp[1][i] = 1;
        }

        for (int i = 1; i <= num; i++) {
            for (int j = i; j <= num * 6; j++) {
                // 循环累加dp数组
                for (int k = 1; k <= 6 && j >= k; k++) {
                    dp[i][j] += dp[i - 1][j - k];
                }
                if (i == num) {
                    res[j - i] = dp[i][j] / totals;
                }
            }
        }

        return res;
    }
}
```

# 位运算

## [剑指 Offer 15. 二进制中1的个数](https://leetcode.cn/problems/er-jin-zhi-zhong-1de-ge-shu-lcof/)

注意：本题与 [191. 位1的个数](https://leetcode.cn/problems/number-of-1-bits/) 相同

```java
public class Solution {
    // you need to treat n as an unsigned value
    public int hammingWeight(int n) {
        int count = 0;
        while (n != 0) {
            count++;
            n = n & (n - 1);
        }
        return count;
    }
}
```

## [剑指 Offer 65. 不用加减乘除做加法](https://leetcode.cn/problems/bu-yong-jia-jian-cheng-chu-zuo-jia-fa-lcof/)

**1.递归解法(内存击败83.04%)**

```java
class Solution {
    public int add(int a, int b) {
        int sum = a ^ b;// 无进位的和
        int carry = (a & b) << 1;// 进位信息
        return carry == 0 ? sum : add(sum, carry);
    }
}
```

**2.迭代解法(内存击败98.62%)**

```java
class Solution {
    public int add(int a, int b) {
        int tmp = 0;
        while (b != 0) {
            tmp = a;
            a ^= b;// 将a作为无进位的和
            b = (tmp & b) << 1;// 将b作为进位信息
        }
        return a;
    }
}
```

## [剑指 Offer 56 - I. 数组中数字出现的次数](https://leetcode.cn/problems/shu-zu-zhong-shu-zi-chu-xian-de-ci-shu-lcof/)

知识点：异或运算

- 如果数组只有一个数出现奇数次，对数组所有数进行异或可以求出这个数；

- 如果数组有两个数出现奇数次，则需要将这两个数拆分到不同的数组，再进行异或。

拆分依据推导如下：

- 对数组所有数异或得到的结果，跟对这两个数异或结果相同。
- 因为这两个数不等，所以异或结果肯定不为0。
- 由于异或结果中为“1”的位，表示这两个数在对应位上不同。所以可取异或结果最右边的1来划分这两个数到不同数组

```java
class Solution {
    public int[] singleNumbers(int[] nums) {
        int xor = 0, m = 0x01, x_xor = 0, y_xor = 0;
        // 对数组异或
        for (int num : nums) {
            xor ^= num;
        }
        // 取异或结果中最右边的“1”所在的位来划分数组
        while ((xor & m) == 0) {
            m <<= 1;
        }
        for (int num : nums) {
            if ((num & m) == 0) {
                x_xor ^= num;
            } else {
                y_xor ^= num;
            }
        }
        return new int[]{x_xor, y_xor};
    }
}
```

## [剑指 Offer 56 - II. 数组中数字出现的次数 II](https://leetcode.cn/problems/shu-zu-zhong-shu-zi-chu-xian-de-ci-shu-ii-lcof/)

解题思路：出现三次的数字，其二进制表示中各位也会出现三次，所以可以通过对所有数字二进制表示中各位求和，然后将各位对3取模，可得只出现1次的数字。

```java
class Solution {
    public int trainingPlan(int[] actions) {
        return singleNumber(actions);
    }
    public int singleNumber(int[] nums) {
        int[] bits = new int[32];
        for (int num : nums) {
            for (int i = 0; i < 32; i++) {
                bits[i] += num & 0x01;
                num >>= 1;
            }
        }
        int res = 0;
        for (int i = 0; i< bits.length; i++) {
            res |= (bits[i] % 3) << i;
        }
        return res;
    }
}
```

# 数学

## [剑指 Offer 39. 数组中出现次数超过一半的数字](https://leetcode.cn/problems/shu-zu-zhong-chu-xian-ci-shu-chao-guo-yi-ban-de-shu-zi-lcof/)

注意：本题与 [169. 多数元素 - 力扣（LeetCode）](https://leetcode.cn/problems/majority-element/) 相同

知识点：众数问题、摩尔投票法

```java
class Solution {
    public int majorityElement(int[] nums) {
        int votes = 0, x = 0;
        for (int i = 0; i < nums.length; i++) {
            if (votes == 0) {
                x = nums[i];
            }
            votes += x == nums[i] ? 1 : -1;
        }
        return x;
    }
}
```

## [剑指 Offer 66. 构建乘积数组](https://leetcode.cn/problems/gou-jian-cheng-ji-shu-zu-lcof/)

解题思路：先求出所有元素乘积，再除以当前元素。

由于涉及到除法，需要考虑被除数为0的情况：

- 数组中不存在0，则所有元素的乘积除以当前元素；
- 数组中存在一个0，则乘积数组除0外的其它元素均为0；
- 数组中存在多个0，则乘积数组各元素均为0；

```java
class Solution {
    public int[] statisticalResult(int[] arrayA) {
        if (arrayA.length < 2) {
            return new int[0];
        }

        int multiply = 1;
        int zeros = 0;
        for (int i = 0; i < arrayA.length; i++) {
            if (arrayA[i] == 0) {
                zeros++;
                continue;
            }
            multiply *= arrayA[i];
        }

        for (int i = 0; i < arrayA.length; i++) {
            if (zeros == 0) {
                arrayA[i] = (int) (multiply * 1.0 / arrayA[i]);
            } else if (zeros == 1 && arrayA[i] == 0) {
                arrayA[i] = multiply;
            } else {
                arrayA[i] = 0;
            }
        }
        return arrayA;
    }
}
```

测试用例

> [1, 2, 0, 4, 5]
> [1, 2, 0, 4, 0]

## [剑指 Offer 14- I. 剪绳子](https://leetcode.cn/problems/jian-sheng-zi-lcof/)

注意：本题与 [343. 整数拆分 - 力扣（LeetCode）](https://leetcode.cn/problems/integer-break/) 相同

```java
class Solution {
    public int cuttingBamboo(int bamboo_len) {
        if (bamboo_len <= 3) {
            return bamboo_len - 1;
        }
        int pow = bamboo_len / 3;
        int remain = bamboo_len % 3;
        if (remain == 0) {
            return (int) Math.pow(3, pow);
        } else if (remain == 1) {
            return (int) Math.pow(3, pow - 1) * 4;
        } else {
            return (int) Math.pow(3, pow) * remain;
        }
    }
}
```

## [剑指 Offer 14- II. 剪绳子](https://leetcode.cn/problems/jian-sheng-zi-ii-lcof/)

注意：本题与 [343. 整数拆分 - 力扣（LeetCode）](https://leetcode.cn/problems/integer-break/) 相同

与上一题相比，长度从58变为1000，需要考虑大数问题(取模、long类型表示乘积)

```java
class Solution {
    public int cuttingBamboo(int bamboo_len) {
        if (bamboo_len <= 3) {
            return bamboo_len - 1;
        }
        int pow = bamboo_len / 3;
        int remain = bamboo_len % 3;
        if (remain == 0) {
            return (int) pow(3, pow);
        } else if (remain == 1) {
            return (int) (pow(3, pow - 1) * 4 % 1000000007);
        } else {
            return (int) (pow(3, pow) * remain % 1000000007);
        }
    }

    // 因为是大数，所以返回结果要为 long 类型
    private long pow(int x, int pow) {
        long power = 1;
        for (int i = 0; i < pow; i++) {
            power = (power * x) % 1000000007;
        }
        return power;
    }
}
```

## [剑指 Offer 57 - II. 和为s的连续正数序列](https://leetcode.cn/problems/he-wei-sde-lian-xu-zheng-shu-xu-lie-lcof/)

知识点：滑动窗口双指针

- 初始化左指针为1，右指针为2，和为3；
- 和小于目标时，右指针右移，和累加右元素；
- 和大于目标时，左指针右移，和减去左元素；
- 和等于目标时，记录并左指针右移，和减去左元素；
- 左右指针相等时退出循环

```java
class Solution {
    public int[][] findContinuousSequence(int target) {
        List<int[]> res = new ArrayList<>();
        int lo = 1, hi = 2, sum = 3;
        while (lo < hi) {
            if (sum < target) {
                hi++;
                sum += hi;
            } else {
                if (sum == target) {
                    int[] e = new int[hi - lo + 1];
                    for (int i = 0; i < e.length; i++) {
                        e[i] = lo + i;
                    }
                    res.add(e);
                }
                sum -= lo;
                lo++;
            }
        }
        return res.toArray(new int[0][]);
    }
}
```

## [*剑指 Offer 62. 圆圈中最后剩下的数字](https://leetcode.cn/problems/yuan-quan-zhong-zui-hou-sheng-xia-de-shu-zi-lcof/)

知识点：约瑟尔环问题

**1.模拟链表法**

```java
class Solution {
    public int lastRemaining(int n, int m) {
        List<Integer> list = new ArrayList<>();// 用LinkedList会超时
        for (int i = 0; i < n; i++) {
            list.add(i);
        }
        int idx = 0;
        while (n > 1) {
            idx = (idx + m - 1) % n--;// 由“n=5,m=3;n=5,m=6;n=5,m=2”推导出来
            list.remove(idx);
        }
        return list.get(0);
    }
}
```

**2.数学解法(击败99.90%)**

```java
class Solution {
    public int lastRemaining(int n, int m) {
        int ans = 0;
        // 最后一轮剩下2个人，所以从2开始反推
        for (int i = 2; i <= n; i++) {
            ans = (ans + m) % i;
        }
        return ans;
    }
}
```

## [剑指 Offer 43. 1～n 整数中 1 出现的次数](https://leetcode.cn/problems/1nzheng-shu-zhong-1chu-xian-de-ci-shu-lcof/)

注意：本题与 [233. 数字 1 的个数 - 力扣（LeetCode）](https://leetcode.cn/problems/number-of-digit-one/) 相同

**1.评论区解法(击败100%)**

```java
class Solution {
    public int digitOneInNumber(int num) {
        if (num == 0) {
            return 0;
        } else if (num < 10) {
            return 1;
        }

        // 这里以“1234”为例
        String number = String.valueOf(num);
        // “1234”的长度为4
        int length = number.length();
        // “1234”的高位为1
        int high = number.charAt(0) - '0';
        // 0-9有1个1，0-99有20个1，0-999有300个1，以此类推
        // “1234”的base为999
        int base = (int)((length - 1) * Math.pow(10, length - 2));
        // "1234"的level为1000
        int level = (int)(Math.pow(10, length - 1));

        // 高位是1的情况
        if (high == 1) {
            // “1234”中1的个数有999+1+234+f(234)
            // 即“0-999”中有999个1，“1001-1234”中左边有1234-1000+1个1，右边有f(234)个1
            return base + 1 + num - level + digitOneInNumber(num - level);
        } else {
            // 高位不是1的情况
            // “2345”中1的个数有2*999+1000+f(345)
            // 即“0-999,1000-1999”中有2*999+1000个1，“2000-2345”中有f(345)个1
            return high * base + level + digitOneInNumber(num - high * level);
        }
    }
}
```

**2.K神解法(击败100%)**

遍历 num 每一位，统计每一位出现 1 的次数。

```java
class Solution {
    public int digitOneInNumber(int num) {
        int high = num / 10;
        int low = 0;
        int digit = 1;
        int current = num % 10;
        int count = 0;
        while (current != 0 || high != 0) {
            if (current == 0) {
                count += high * digit;
            } else if (current == 1) {
                count += high * digit + low + 1;// 易错点
            } else {
                count += (high + 1) * digit;
            }
            current = high % 10;
            high /= 10;
            digit *= 10;
            low = num % digit;
        }
        return count;
    }
}
```

测试用例

> 11
> 110

## [剑指 Offer 44. 数字序列中某一位的数字](https://leetcode.cn/problems/shu-zi-xu-lie-zhong-mou-yi-wei-de-shu-zi-lcof/)

注意：本题与 [400. 第 N 位数字 - 力扣（LeetCode）](https://leetcode.cn/problems/nth-digit/) 相同

注意大数问题，需要用 long 类型。

解题思路：

先定位第k位是属于哪个数，然后再定位是属于该数的第几位，即可得结果。

以求第12位数字为例，因为

- 0-9有9-0+1=10个数字

- 10-99有(99-10+1) * 2=90*2个数字

- 100-999有(999-100+1) * 3=900*3个数字
- 以此类推

12-10=2 <= 90*2，所以是属于10-99这个范围。

2/2=1，所以是属于10-99中的第1位数字（从0开始），即10+1=11。

2%2=0，所以是属于数字“11”中的第0位，即1

我的代码：

```java
class Solution {
    public int findKthNumber(int k) {
        if (k < 10) {
            return k;
        }
        int i = 2;
        long j = 10;// 需要是 long 类型
        k -= 10;
        while (k > i * j * 9) {
            k -= i * j * 9;
            i++;
            j *= 10;
        }
        long n = k / i + j;
        return String.valueOf(n).charAt(k % i) - '0';
    }
}
```

