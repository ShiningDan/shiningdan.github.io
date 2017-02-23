---
title: LeetCode Linked List 题集
date: 2017-02-22 21:13:01
categories: coding
tags:
   - LeetCode
   - LinkedList
   - JavaScript
   - Java
---

下面所记录的是我在刷 [LeetCode](https://leetcode.com) Array 相关的题目自己的解法。

### [21. Merge Two Sorted Lists](https://leetcode.com/problems/merge-two-sorted-lists/)

Merge two sorted linked lists and return it as a new list. The new list should be made by splicing together the nodes of the first two lists.

#### JavaScript Solution

递归方法

```
/**
 * Definition for singly-linked list.
 * function ListNode(val) {
 *     this.val = val;
 *     this.next = null;
 * }
 */
/**
 * @param {ListNode} l1
 * @param {ListNode} l2
 * @return {ListNode}
 */
var mergeTwoLists = function(l1, l2) {
    if (l1 === null) return l2;
    if (l2 === null) return l1;

    if (l1.val < l2.val) {
    	l1.next = mergeTwoLists(l1.next, l2);
    	return l1;
    } else {
    	l2.next = mergeTwoLists(l2.next, l1);
    	return l2;
    }
};
```

#### Java Solution

迭代方法

```
/**
 * Definition for singly-linked list.
 * public class ListNode {
 *     int val;
 *     ListNode next;
 *     ListNode(int x) { val = x; }
 * }
 */
public class Solution {
    public ListNode mergeTwoLists(ListNode l1, ListNode l2) {
        if (l1 == null) return l2;
        if (l2 == null) return l1;

        ListNode lEnd = new ListNode(0);
        ListNode lStart = lEnd;
        while (l1 != null && l2 != null) {
            if (l1.val < l2.val) {
                lEnd.next = l1;
                l1 = l1.next;
            } else {
                lEnd.next = l2;
                l2 = l2.next;
            }
            lEnd = lEnd.next;
        }
        lEnd.next = (l1 == null) ? l2 : l1;
        return lStart.next;
    }
}
```

<!--more-->

### [83. Remove Duplicates from Sorted List](https://leetcode.com/problems/remove-duplicates-from-sorted-list/?tab=Description)


For example,
Given `1->1->2`, return `1->2`.
Given `1->1->2->3->3`, return `1->2->3`.

#### JavaScript Solution

```
/**
 * Definition for singly-linked list.
 * function ListNode(val) {
 *     this.val = val;
 *     this.next = null;
 * }
 */
/**
 * @param {ListNode} head
 * @return {ListNode}
 */
var deleteDuplicates = function(head) {
    if (head === null) return head;
    let result = head;
    while (head.next !== null) {
    	if (head.next.val === head.val) {
    		head.next = head.next.next;
    	} else {
    		head = head.next;
    	}
    }
    return result;
};
```

#### Java Solution

递归方法

```
public ListNode deleteDuplicates(ListNode head) {
        if(head == null || head.next == null)return head;
        head.next = deleteDuplicates(head.next);
        return head.val == head.next.val ? head.next : head;
}
```

### [141. Linked List Cycle](https://leetcode.com/problems/linked-list-cycle/?tab=Description)

Given a linked list, determine if it has a cycle in it.

Follow up:
Can you solve it without using extra space?

#### JavaScript Solution

```
/**
 * Definition for singly-linked list.
 * function ListNode(val) {
 *     this.val = val;
 *     this.next = null;
 * }
 */

/**
 * @param {ListNode} head
 * @return {boolean}
 */
var hasCycle = function(head) {
    if (head === null) {return false;}
    while (head.next !== null) {
    	if (head.next === head) {
    		return true;
    	}
    	head.next = head.next.next;
    }
    return false;
};
```

#### Best Solution

Best Solution 的做法是一个 fast runner 和一个 slow runner，如果有环路，fast 最终一定可以和 slow 遇上。

```
/**
 * Definition for singly-linked list.
 * function ListNode(val) {
 *     this.val = val;
 *     this.next = null;
 * }
 */

/**
 * @param {ListNode} head
 * @return {boolean}
 */
var hasCycle = function(head) {
    if (head === null) {return false;}
    let fast = slow = head;
    while (fast !== null && fast.next !== null) {
    	fast = fast.next.next;
    	slow = slow.next;
    	if (fast === slow) {return true;}
    }
    return false;
};
```

### [160. Intersection of Two Linked Lists](https://leetcode.com/problems/intersection-of-two-linked-lists/?tab=Description)

Write a program to find the node at which the intersection of two singly linked lists begins.

For example, the following two linked lists:

```
A:          a1 → a2
                   ↘
                     c1 → c2 → c3
                   ↗            
B:     b1 → b2 → b3
```

begin to intersect at node c1.


**Notes:**

If the two linked lists have no intersection at all, return `null`.
The linked lists must retain their original structure after the function returns.
You may assume there are no cycles anywhere in the entire linked structure.
Your code should preferably run in O(n) time and use only O(1) memory.

#### Best Solution

这个解法妙在将两个链表合为一个链表，比如：

```
0 -> 1 -> 2 //第一个链表
3 -> 4 -> 5 -> 1 -> 2 //第二个链表
```

合成后的链表为：

```
0 1 2 3 4 5 1 2 //第一个链表
3 4 5 1 2 0 1 2 // 第二个链表
```

经过比较就可以找到相同的点

```
/**
 * Definition for singly-linked list.
 * function ListNode(val) {
 *     this.val = val;
 *     this.next = null;
 * }a
 */

/**
 * @param {ListNode} headA
 * @param {ListNode} headB
 * @return {ListNode}
 */
var getIntersectionNode = function(headA, headB) {
    if (headA === null || headB === null) {return null}
    let a = headA, b =headB;
	while (a !== b) {
		a = (a === null) ? headB : a.next;
		b = (b === null) ? headA : b.next;
	}
	return a;
};
```

### [203. Remove Linked List Elements](https://leetcode.com/problems/remove-linked-list-elements/?tab=Description)

Remove all elements from a linked list of integers that have value **val**.

**Example**

**Given:** 1 --> 2 --> 6 --> 3 --> 4 --> 5 --> 6, **val** = 6

**Return:** 1 --> 2 --> 3 --> 4 --> 5

#### JavaScript Solution

如果链表的第一个值不方便处理的时候，可以再创建一个节点指向第一个值。

```
/**
 * Definition for singly-linked list.
 * function ListNode(val) {
 *     this.val = val;
 *     this.next = null;
 * }
 */
/**
 * @param {ListNode} head
 * @param {number} val
 * @return {ListNode}
 */
var removeElements = function(head, val) {
    let cur = new ListNode(0), start = null;
    cur.next = head;
    while (cur.next !== null) {
    	if (cur.next.val === val) {
    		cur.next = cur.next.next;
    	} else if (start === null) {
    		start = cur.next;
    	} else {
    		cur = cur.next;
    	}
    }
    return start;
};
```

#### Best Solution

递归方法，这个方法的思想就是，如果本节点不满足要求，就返回下一个节点。

```
/**
 * Definition for singly-linked list.
 * public class ListNode {
 *     int val;
 *     ListNode next;
 *     ListNode(int x) { val = x; }
 * }
 */
public class Solution {
    public ListNode removeElements(ListNode head, int val) {
        if (head == null) return null;
        head.next = removeElements(head.next, val);
        return head.val == val ? head.next : head;
    }
}
```

### [206. Reverse Linked List](https://leetcode.com/problems/reverse-linked-list/?tab=Description)

Reverse a singly linked list.

#### JavaScript Solution







