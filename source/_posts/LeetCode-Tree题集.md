---
title: LeetCode Tree题集
date: 2017-02-12 21:43:54
categories: coding
tags:
  - LeetCode
  - Tree
  - Java
  - JavaScript
---


下面所记录的是我在刷 [LeetCode](https://leetcode.com) Array 相关的题目自己的解法。

#### [100. Same Tree](https://leetcode.com/problems/same-tree/)

Given two binary trees, write a function to check if they are equal or not.

Two binary trees are considered equal if they are structurally identical and the nodes have the same value

#### JavaScript Solution

```
/**
 * Definition for a binary tree node.
 * function TreeNode(val) {
 *     this.val = val;
 *     this.left = this.right = null;
 * }
 */
/**
 * @param {TreeNode} p
 * @param {TreeNode} q
 * @return {boolean}
 */
var isSameTree = function(p, q) {
    if (p === null && q === null) return true;
    if (p === null || q === null) return false;
    return (p.val === q.val && isSameTree(p.left, q.left) && isSameTree(p.right, q.right));
};
```

#### Java Solution

```
/**
 * Definition for a binary tree node.
 * public class TreeNode {
 *     int val;
 *     TreeNode left;
 *     TreeNode right;
 *     TreeNode(int x) { val = x; }
 * }
 */
public class Solution {
    public boolean isSameTree(TreeNode p, TreeNode q) {
        if (p == null && q == null) return true;
        if (p == null || q == null) return false;
        return (p.val == q.val && isSameTree(p.left, q.left) && isSameTree(p.right, q.right));
    }
}
```

<!--more-->

### [101. Symmetric Tree](https://leetcode.com/problems/symmetric-tree/)

Given a binary tree, check whether it is a mirror of itself (ie, symmetric around its center).

For example, this binary tree `[1,2,2,3,4,4,3]` is symmetric:

```
    1
   / \
  2   2
 / \ / \
3  4 4  3
```

But the following `[1,2,2,null,3,null,3]` is not:

```
    1
   / \
  2   2
   \   \
   3    3
```

**Note:**

Bonus points if you could solve it both recursively and iteratively.

#### JavaScript Solution

使用的是递归。

```
/**
 * Definition for a binary tree node.
 * function TreeNode(val) {
 *     this.val = val;
 *     this.left = this.right = null;
 * }
 */
/**
 * @param {TreeNode} root
 * @return {boolean}
 */
var isSymmetric = function(root) {
	let isSymmetricTree = function(p, q) {
    	if (p === null && q === null) return true;
    	if (p === null || q === null) return false;
    	return (p.val === q.val && isSymmetricTree(p.left, q.right) && isSymmetricTree(p.right, q.left));
    }
    if (root === null) return true;
    return isSymmetricTree(root.left, root.right);
};
```

#### Java Solution

使用的是循环，利用 Stack 来存储要比较的节点

```
public boolean isSymmetric(TreeNode root) {
        Stack<TreeNode> stack = new Stack<>();
        if (root == null) return true;
        stack.push(root.left);
        stack.push(root.right);
        while (!stack.empty()) {
            TreeNode r = stack.pop();
            TreeNode l = stack.pop();
            if (r == null && l == null) continue;
            if (r == null || l == null) return false;

            if (r.val != l.val) return false;
            stack.push(l.left);
            stack.push(r.right);
            stack.push(l.right);
            stack.push(r.left);
        }
        return true;
    }
```

### [104. Maximum Depth of Binary Tree](https://leetcode.com/problems/maximum-depth-of-binary-tree/)

Given a binary tree, find its maximum depth.

The maximum depth is the number of nodes along the longest path from the root node down to the farthest leaf node.

#### JavaScript Solution

深度优先搜索算法：

```
/**
 * Definition for a binary tree node.
 * function TreeNode(val) {
 *     this.val = val;
 *     this.left = this.right = null;
 * }
 */
/**
 * @param {TreeNode} root
 * @return {number}
 */
var maxDepth = function(root) {
    if (root === null) return 0;
    return (1 + Math.max(maxDepth(root.left), maxDepth(root.right)));
};
```

#### Java Solution

广度优先搜索算法。

```
public int maxDepth(TreeNode root) {
        if (root ==  null) return 0;
        Queue<TreeNode> q = new LinkedList<>();
        int deep = 0;
        q.add(root);
        while (!q.isEmpty()) {
            int len = q.size();
            deep ++;
            for (int i = 0; i < len; i++) {
                TreeNode t = q.poll();
                if (t.left != null) q.add(t.left);
                if (t.right != null) q.add(t.right);
            }
        }
        return deep;
    }
```

### [107. Binary Tree Level Order Traversal II](https://leetcode.com/problems/binary-tree-level-order-traversal-ii/)

Given a binary tree, return the bottom-up level order traversal of its nodes' values. (ie, from left to right, level by level from leaf to root).

For example:
Given binary tree `[3,9,20,null,null,15,7]`,

```
    3
   / \
  9  20
    /  \
   15   7
```

return its bottom-up level order traversal as:

```
[
  [15,7],
  [9,20],
  [3]
]
```

#### JavaScript Solution

使用广度优先遍历该树

```
/**
 * Definition for a binary tree node.
 * function TreeNode(val) {
 *     this.val = val;
 *     this.left = this.right = null;
 * }
 */
/**
 * @param {TreeNode} root
 * @return {number[][]}
 */
var levelOrderBottom = function(root) {
    if (root === null) return [];
    let result = [];
    let a = [root];
    while (a.length > 0) {
    	let len = a.length;
    	let rtmp = []
    	for (let i = 0; i < len; i ++) {
    		let node = a.shift();
    		rtmp.push(node.val);
    		if (node.left !== null) a.push(node.left);
    		if (node.right !== null) a.push(node.right);
    	}
    	result.unshift(rtmp);
    }
    return result;
};
```

### [108. Convert Sorted Array to Binary Search Tree](https://leetcode.com/problems/convert-sorted-array-to-binary-search-tree/)

Given an array where elements are sorted in ascending order, convert it to a height balanced BST.

#### JavaScript Solution

通过一个排好序的数组，生成平衡二叉查找树。

```
/**
 * Definition for a binary tree node.
 * function TreeNode(val) {
 *     this.val = val;
 *     this.left = this.right = null;
 * }
 */
/**
 * @param {number[]} nums
 * @return {TreeNode}
 */
var sortedArrayToBST = function(nums) {
    if (nums.length === 0) return null;

	let getMid = function(nums, low, high) {
		if (low > high) return null;
		let mid = Math.floor((low + high)/2);
		let t = new TreeNode(nums[mid]);
		t.left = getMid(nums, low, mid-1);
		t.right = getMid(nums, mid+1, high);
		return t;
	};

	return getMid(nums, 0, nums.length-1);

};
```

### [110. Balanced Binary Tree](https://leetcode.com/problems/balanced-binary-tree/)

Given a binary tree, determine if it is height-balanced.

For this problem, a height-balanced binary tree is defined as a binary tree in which the depth of the two subtrees of every node never differ by more than 1.




















