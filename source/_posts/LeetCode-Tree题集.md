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


#### JavaScript Solution

1.The first method checks whether the tree is balanced strictly according to the definition of balanced binary tree: the difference between the heights of the two sub trees are not bigger than 1, and both the left sub tree and right sub tree are also balanced. With the helper function depth(), we could easily write the code;


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
var isBalanced = function(root) {
	if (root === null) return true;
    
    let depth = function(node) {
    	if (node === null) return 0;
    	return 1 + Math.max(depth(node.left), depth(node.right));
    }

    let leftDep = depth(root.left);
    let rightDep = depth(root.right);
    return Math.abs(leftDep - rightDep) <= 1 && isBalanced(root.left) && isBalanced(root.right);
};
```

For the current node root, calling depth() for its left and right children actually has to access all of its children, thus the complexity is O(N). We do this for each node in the tree, so the overall complexity of isBalanced will be O(N^2). This is a top down approach.

#### Java Solution

2.The second method is based on DFS. Instead of calling depth() explicitly for each child node, we return the height of the current node in DFS recursion. When the sub tree of the current node (inclusive) is balanced, the function dfsHeight() returns a non-negative value as the height. Otherwise -1 is returned. According to the leftHeight and rightHeight of the two children, the parent node could check if the sub tree is balanced, and decides its return value.

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
    public boolean isBalanced(TreeNode root) {
        int height = dfsHeight(root);
        if (height == -1) return false;
        return true;
    }
    public int dfsHeight(TreeNode node) {
        if (node == null) return 0;

        int leftDep = dfsHeight(node.left);
        int rightDep = dfsHeight(node.right);

        if (leftDep == -1 || rightDep == -1) return -1;
        if (Math.abs(leftDep - rightDep) > 1) return -1;
        return 1 + Math.max(leftDep, rightDep);
    }
}
```

In this bottom up approach, each node in the tree only need to be accessed once. Thus the time complexity is O(N), better than the first solution.

### [111. Minimum Depth of Binary Tree](https://leetcode.com/problems/minimum-depth-of-binary-tree/)

Given a binary tree, find its minimum depth.

The minimum depth is the number of nodes along the shortest path from the root node down to the nearest leaf node.

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
 * @return {number}
 */
var minDepth = function(root) {
    if (root === null) return 0;

    let leftDep = minDepth(root.left);
    let rightDep = minDepth(root.right);
    return (leftDep === 0 || rightDep === 0) ?  leftDep + rightDep +1: 1 + Math.min(leftDep, rightDep);
};
```

### [112. Path Sum](https://leetcode.com/problems/path-sum/)

Given a binary tree and a sum, determine if the tree has a root-to-leaf path such that adding up all the values along the path equals the given sum.

For example:
Given the below binary tree and `sum = 22`,

```
              5
             / \
            4   8
           /   / \
          11  13  4
         /  \      \
        7    2      1
```

return true, as there exist a root-to-leaf path `5->4->11->2` which sum is 22.

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
 * @param {number} sum
 * @return {boolean}
 */
var hasPathSum = function(root, sum) {
    if (root === null) return false;
    
    let calPath = function(node, _s, sum) {
    	if (node === null) return false;
    	let s = _s + node.val;
    	if (s === sum && node.left === null && node.right === null) return true;
    	return calPath(node.left, s, sum) || calPath(node.right, s, sum);
    }

    return calPath(root, 0, sum);
};
```

#### Best Solution

最优解和我的想法大致相同，但是在递归函数的参数设计上比我的要好。

```
public class Solution {
    public boolean hasPathSum(TreeNode root, int sum) {
        if(root == null) return false;
    
        if(root.left == null && root.right == null && sum - root.val == 0) return true;
    
        return hasPathSum(root.left, sum - root.val) || hasPathSum(root.right, sum - root.val);
    }
}
```

### [226. Invert Binary Tree](https://leetcode.com/problems/invert-binary-tree/)

Invert a binary tree.

```
     4
   /   \
  2     7
 / \   / \
1   3 6   9
```

to

```
     4
   /   \
  7     2
 / \   / \
9   6 3   1
```

#### JavaScript Solution

DFS

The below solution is correct, but it is also bound to the application stack, which means that it's no so much scalable - (you can find the problem size that will overflow the stack and crash your application), so more robust solution would be to use stack data structure.

大意是，因为使用递归，并且没有使用尾递归，所以会导致栈溢出，最好不要这样写。可以使用队列，进行广度优先搜索。


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
 * @return {TreeNode}
 */
var invertTree = function(root) {
    if (root === null) return null;
    [root.left, root.right] = [root.right, root.left];
    invertTree(root.left);
    invertTree(root.right);
    return root;
};
```

#### Java Solution

BFS

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
    public TreeNode invertTree(TreeNode root) {
        Queue<TreeNode> q = new LinkedList<>();
        if (root == null) return root;
        q.add(root);
        while (!q.isEmpty()) {
            TreeNode n = q.poll();
            TreeNode tmp = n.left;
            n.left = n.right;
            n.right = tmp;
            if (n.left != null) q.add(n.left);
            if (n.right != null) q.add(n.right);
        }
        return root;
    }
}
```

#### [235. Lowest Common Ancestor of a Binary Search Tree](https://leetcode.com/problems/lowest-common-ancestor-of-a-binary-search-tree/)

Given a binary search tree (BST), find the [lowest common ancestor (LCA)](https://en.wikipedia.org/wiki/Lowest_common_ancestor) of two given nodes in the BST.

According to the definition of LCA on Wikipedia: “The lowest common ancestor is defined between two nodes v and w as the lowest node in T that has both v and w as descendants (where we allow a node to be a descendant of itself).”

```
        _______6______
       /              \
    ___2__          ___8__
   /      \        /      \
   0      _4       7       9
         /  \
         3   5
```

For example, the lowest common ancestor (LCA) of nodes 2 and 8 is 6. Another example is LCA of nodes 2 and 4 is 2, since a node can be a descendant of itself according to the LCA definition.

#### JavaScript Solution

递归：

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
 * @param {TreeNode} p
 * @param {TreeNode} q
 * @return {TreeNode}
 */
var lowestCommonAncestor = function(root, p, q) {
    if (p.val < root.val && q.val < root.val) return lowestCommonAncestor(root.left, p, q);
    if (p.val > root.val && q.val > root.val) return lowestCommonAncestor(root.right, p, q);
    return root;
};  
```

#### Best Solution

非递归

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
    public TreeNode lowestCommonAncestor(TreeNode root, TreeNode p, TreeNode q) {
        while ((p.val - root.val) * (q.val - root.val) > 0) {
            root = (p.val - root.val) < 0 ? root.left: root.right;
        }
        return root;
    }
}
```

### [257. Binary Tree Paths](https://leetcode.com/problems/binary-tree-paths/)


Given a binary tree, return all root-to-leaf paths.

For example, given the following binary tree:

```
   1
 /   \
2     3
 \
  5
```

All root-to-leaf paths are:

```
["1->2->5", "1->3"]
```

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
 * @return {string[]}
 */
var binaryTreePaths = function(root) {
    let result = [];
    let getTreePath = function(node, path) {
    	if (node === null) return ;
    	if (path !== "") {path = path + "->" + node.val;}
    	else path = path + node.val;
    	if (node.left === null && node.right === null) 
    		result.push(path);
    	if (node.left !== null) 
    		getTreePath(node.left, path);
    	if (node.right !== null) 
    		getTreePath(node.right, path);
    }
    getTreePath(root, "");
    return result;
};
```

#### Best Solution

最优解在递归的参数上进行 path 的修改，比我的解法上减少了更多的初始条件判断。

```
public List<String> binaryTreePaths(TreeNode root) {
    List<String> answer = new ArrayList<String>();
    if (root != null) searchBT(root, "", answer);
    return answer;
}
private void searchBT(TreeNode root, String path, List<String> answer) {
    if (root.left == null && root.right == null) answer.add(path + root.val);
    if (root.left != null) searchBT(root.left, path + root.val + "->", answer);
    if (root.right != null) searchBT(root.right, path + root.val + "->", answer);
}
```

### [404. Sum of Left Leaves](https://leetcode.com/problems/sum-of-left-leaves/?tab=Description)

Find the sum of all left leaves in a given binary tree.

**Example:**

```
    3
   / \
  9  20
    /  \
   15   7

There are two left leaves in the binary tree, with values 9 and 15 respectively. Return 24.
```

#### JavaScript Solution

Iterative method.

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
var sumOfLeftLeaves = function(root) {
    let nodes = [];
    let sum = 0;
    if (root === null) return sum;
    nodes.push(root);
    while (nodes.length !== 0) {
    	let node = nodes.pop();
    	if (node.left !== null) {
    		nodes.push(node.left);
    		if (node.left.left === null && node.left.right === null) 
    			sum += node.left.val;
    	}
    	if (node.right !== null) {
    		nodes.push(node.right);
    	}
    }
    return sum;
};
```

#### Java Solution

Recursive method

```
public int sumOfLeftLeaves(TreeNode root) {
    if(root == null) return 0;
    int ans = 0;
    if(root.left != null) {
        if(root.left.left == null && root.left.right == null) ans += root.left.val;
        else ans += sumOfLeftLeaves(root.left);
    }
    ans += sumOfLeftLeaves(root.right);
    
    return ans;
}
```

### [437. Path Sum III](https://leetcode.com/problems/path-sum-iii/)

You are given a binary tree in which each node contains an integer value.

Find the number of paths that sum to a given value.

The path does not need to start or end at the root or a leaf, but it must go downwards (traveling only from parent nodes to child nodes).

The tree has no more than 1,000 nodes and the values are in the range -1,000,000 to 1,000,000.

**Example:**

```
root = [10,5,-3,3,2,null,11,3,-2,null,1], sum = 8

      10
     /  \
    5   -3
   / \    \
  3   2   11
 / \   \
3  -2   1

Return 3. The paths that sum to 8 are:

1.  5 -> 3
2.  5 -> 2 -> 1
3. -3 -> 11
```

#### JavaScript Solution

做法就是，遍历树，并且计算所有中间和的结果，放在 map 里面。

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
 * @param {number} sum
 * @return {number}
 */
var pathSum = function(root, sum) {
    let map = new Map();
    let calPathSum = function(node, sums) {
    	if (node === null) {return ;}
    	for (let i = 0; i < sums.length; i ++) {
    		sums[i] += node.val;
    	}
    	sums.push(node.val);
    	for (let i = 0; i < sums.length; i++) {
    		if (map.has(sums[i])) {
    			map.set(sums[i], map.get(sums[i]) + 1);
    		} else {
    			map.set(sums[i], 1);
    		}
    	}
    	calPathSum(node.left, sums.concat());
    	calPathSum(node.right, sums.concat());
    }
    calPathSum(root, []);
    if (map.has(sum)) {return map.get(sum)}
    return 0;
};
```

#### Best Solution

```
 public int pathSum(TreeNode root, int sum) {
        HashMap<Integer, Integer> preSum = new HashMap();
        preSum.put(0,1);
        return helper(root, 0, sum, preSum);
    }
    
    public int helper(TreeNode root, int currSum, int target, HashMap<Integer, Integer> preSum) {
        if (root == null) {
            return 0;
        }
        
        currSum += root.val;
        int res = preSum.getOrDefault(currSum - target, 0);
        preSum.put(currSum, preSum.getOrDefault(currSum, 0) + 1);
        
        res += helper(root.left, currSum, target, preSum) + helper(root.right, currSum, target, preSum);
        preSum.put(currSum, preSum.get(currSum) - 1);
        return res;
    }
```

### [501. Find Mode in Binary Search Tree](https://leetcode.com/problems/find-mode-in-binary-search-tree/?tab=Description)


Given a binary search tree (BST) with duplicates, find all the mode(s) (the most frequently occurred element) in the given BST.

Assume a BST is defined as follows:

* The left subtree of a node contains only nodes with keys **less than or equal to** the node's key.
* The right subtree of a node contains only nodes with keys **greater than or equal to** the node's key.
* Both the left and right subtrees must also be binary search trees.

For example:

Given BST `[1,null,2,2]`,

```
   1
    \
     2
    /
   2
```

return `[2]`.

**Note:** If a tree has more than one mode, you can return them in any order.

**Follow up: **Could you do that without using any extra space? (Assume that the implicit stack space incurred due to recursion does not count).

#### Best Solution

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
    public int[] findMode(TreeNode root) {
        inorder(root);
        modes = new int[modeCount];
        modeCount = 0;
        currCount = 0;
        inorder(root);
        return modes;
    }

    private int currVal;
    private int currCount = 0;
    private int maxCount = 0;
    private int modeCount = 0;
    
    private int[] modes;

    private void handleValue(int val) {
        if (val != currVal) {
            currVal = val;
            currCount = 0;
        }
        currCount++;
        if (currCount > maxCount) {
            maxCount = currCount;
            modeCount = 1;
        } else if (currCount == maxCount) {
            if (modes != null)
                modes[modeCount] = currVal;
            modeCount++;
        }
    }
    
    private void inorder(TreeNode root) {
        if (root == null) return;
        inorder(root.left);
        handleValue(root.val);
        inorder(root.right);
    }
}
```