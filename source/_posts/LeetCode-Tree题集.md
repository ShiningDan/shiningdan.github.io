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








