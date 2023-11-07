# [876. 链表的中间结点](https://leetcode.cn/problems/middle-of-the-linked-list/)

快慢指针，快指针走两步，慢指针走一步，快指针不能继续走时，则慢指针在中间节点

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
    public ListNode middleNode(ListNode head) {
        if(head == null || head.next == null){
            return head;
        }

        ListNode fast = head;
        ListNode slow = fast;

        while(fast.next != null && fast.next.next != null){
            fast = fast.next.next;
            slow = slow.next;
        }

        return fast.next == null ? slow : slow.next;
    }
}
```

# [148. 排序链表](https://leetcode.cn/problems/sort-list/)

归并排序，知识点：[876. 链表的中间结点](#876. 链表的中间结点)、[21. 合并两个有序链表](https://leetcode.cn/problems/merge-two-sorted-lists/) 

```java
/**
 * Definition for singly-linked list.
 * public class ListNode {
 * int val;
 * ListNode next;
 * ListNode() {}
 * ListNode(int val) { this.val = val; }
 * ListNode(int val, ListNode next) { this.val = val; this.next = next; }
 * }
 */
class Solution {
    public ListNode sortList(ListNode head) {
        if (head == null || head.next == null) {
            return head;
        }

        ListNode fast = head, slow = fast;
        while (fast.next != null && fast.next.next != null) {
            fast = fast.next.next;
            slow = slow.next;
        }
        // 这里取下一个节点作为中间节点，方便做截断
        ListNode mid = slow.next;
        slow.next = null;

        ListNode left = sortList(head);
        ListNode right = sortList(mid);

        // 因为已经通过递归的方式分解成单个节点，
        // 所以可以用“合并两个有序链表”的方式来合并
        return mergeSort(left, right);
    }

    /**
     * 合并两个有序链表（迭代解法）
     */
    private ListNode mergeSort(ListNode l1, ListNode l2) {
        ListNode head = new ListNode(-1);
        ListNode prev = head;

        while (l1 != null && l2 != null) {
            if (l1.val < l2.val) {
                prev = prev.next = l1;
                l1 = l1.next;
            } else {
                prev = prev.next = l2;
                l2 = l2.next;
            }
        }

        prev.next = l1 != null ? l1 : l2;
        return head.next;
    }
}
```

# [1. 两数之和](https://leetcode.cn/problems/two-sum/)

注意：本题与 [剑指 Offer 57. 和为s的两个数字](https://leetcode.cn/problems/he-wei-sde-liang-ge-shu-zi-lcof/) 相似但不相同

**1.暴力求解法**

```bash
class Solution {
    public int[] twoSum(int[] nums, int target) {
        int[] result = new int[2];
        for (int i = 0; i < nums.length - 1; i++) {
            for (int j = i + 1; j < nums.length; j++) {
                if (nums[j] + nums[i] == target) {
                    result[0] = i;
                    result[1] = j;
                    break;
                }
            }
        }
        return result;
    }
}
```

**2.哈希求解法(击败98.81%)**

```java
class Solution {
    public int[] twoSum(int[] nums, int target) {
        Map<Integer, Integer> map = new HashMap<>();
        for (int i = 0; i < nums.length; i++) {
            if (map.containsKey(target - nums[i])) {
                return new int[]{i, map.get(target - nums[i])};
            }
            map.put(nums[i], i);
        }
        return new int[0];
    }
}
```

