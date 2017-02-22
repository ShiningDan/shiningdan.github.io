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
















