---
title: 剑指offer题集
date: 2017-04-04 19:59:11
categories: coding
tags:
  - algorithm
---

本笔记是刷 《剑指 offer》 题集中题目的笔记记录。

## 二维数组中的查找

### 题目描述

在一个二维数组中，每一行都按照从左到右递增的顺序排序，每一列都按照从上到下递增的顺序排序。请完成一个函数，输入这样的一个二维数组和一个整数，判断数组中是否含有该整数。

**该解法将数组考虑成一个矩阵，不过每次比较的时候，是从矩阵的右上角或者左下角开始比较。假设从左下角开始比较，则通过每一次比较的结果，如果 `target > a[i][j]`，则比较上一节点；如果`target < a[i][j]`，则比较右边的节点。**

还需要注意边界判断，如果给的数组是 `[[]]`，则很容易返回边界错误，此时如何解决？

解法：

```
function Find(target, array)
{
    var len = array.length;
  	var colLen = array[0].length;
    if (array != null && len > 0 && colLen > 0) {
        var i = 0, j = colLen-1;
        while(i < len && j >= 0) {
            if (target === array[i][j]) {
                return true;
            } else if (target > array[i][j]) {
                ++i;
            } else {
                --j;
            }
        }
    }
    return false;
}

module.exports = {
    Find : Find
};
```

## 替换空格

### 题目描述

请实现一个函数，将一个字符串中的空格替换成“%20”。例如，当字符串为We Are Happy.则经过替换之后的字符串为We%20Are%20Happy。

第一个方法是创建新的字符串，进行拼接，但是这样会使用多余的空间。

```
function replaceSpace(str)
{
    for (var i = 0; i < str.length; i++) {
        if (str[i] === ' ') {
            var left = str.slice(0, i), right = str.slice(i+1, str.length);
            str = left + '%20' + right;
        }
    }
    return str;
}
module.exports = {
    replaceSpace : replaceSpace
};
```

或者直接使用 replace 方法：

```
function replaceSpace(str)
{
    return str.replace(/\s/g, '%20');
}
module.exports = {
    replaceSpace : replaceSpace
};
```

但是如果不考虑使用额外的空间，在原来的字符串上进行修改，在替换的时候，就需要将原来的字符串进行后移，一个空格需要向后移动三位，然后分别填充 `%`、`2`、`0`。

假设字符串的长度是 n，则对每个空格字符，都需要移动后面 O(n) 个字符，时间效率为 O(n^2)

下面介绍使用 O(n) 时间复杂度的移动字符的方法：

首先遍历一遍字符串，统计有多少个空格，就知道要后移 2 倍空格的长度。当入到字符串的时候，就用 `%20` 进行添加，否则，只是复制该字符：

```
function replaceSpace(str)
{
    var spaceNum = 0;
    for (var i = 0; i < str.length; i++) {
        if (str[i] === ' ')
            ++spaceNum;
    }
    var p = str.length - 1 + 2*spaceNum;
    for (var i = str.length-1; i >=0; i--) {
        if (str[i] === ' ') {
            str[p--] = '0';
            str[p--] = '2';
            str[p--] = '%';
        } else {
            str[p--] = str[i];
        }
    }
    return str;
}
module.exports = {
    replaceSpace : replaceSpace
};
```

## 从头到尾打印链表

### 题目描述

输入一个链表，从尾到头打印链表每个节点的值。

如果可以修改链表的指向，可以让后一项指向前一项。但是如果不能修改链表的指向，我们可以如下分析：

**遍历的第一个节点最后一个输出，遍历的最后一个节点第一个输出，也就是先入后出，可以使用栈来实现。不过使用栈需要额外的 O(n) 空间：**

```
/*function ListNode(x){
    this.val = x;
    this.next = null;
}*/
function printListFromTailToHead(head)
{
    var stack = [];
    while (head) {
        stack.unshift(head.val);
        head = head.next;
    }
    return stack;
}
module.exports = {
    printListFromTailToHead : printListFromTailToHead
};
```

如果用栈来实现这个函数，而递归本身就是一个栈结构，可以使用递归，先输出自己的后一个节点，再数组自己：

```
/*function ListNode(x){
    this.val = x;
    this.next = null;
}*/
var result = [];
function printListFromTailToHead(head)
{
    if (head) {
        if (head.next) {
            printListFromTailToHead(head.next);
        }
        result.add(head.val);
    }
}
return result;
module.exports = {
    printListFromTailToHead : printListFromTailToHead
};
```

## 重建二叉树

### 题目描述

输入某二叉树的前序遍历和中序遍历的结果，请重建出该二叉树。假设输入的前序遍历和中序遍历的结果中都不含重复的数字。例如输入前序遍历序列{1,2,4,7,3,5,6,8}和中序遍历序列{4,7,2,1,5,3,8,6}，则重建二叉树并返回。

**通过前序遍历，第一个节点就是根节点，然后在中序遍历中，根节点前面的就是根节点的左子树，根节点右边的就是右子树。**

```
function TreeNode(x) {
    this.val = x;
    this.left = null;
    this.right = null;
}
function reConstructBinaryTree(pre, vin)
{
    if (pre == null || pre.length === 0) {
        return null;
    }
    var node = new TreeNode(pre[0]);
    for (var index = 0; index < pre.length; index++) {
        if (vin[index] === pre[0]) {
            node.left = reConstructBinaryTree(pre.slice(1, index+1), vin.slice(0, index));
    		node.right = reConstructBinaryTree(pre.slice(index+1, pre.length), vin.slice(index+1, vin.length));
        }
    }
   
    return node;
    
}
module.exports = {
    reConstructBinaryTree : reConstructBinaryTree
};
```

## 使用两个栈实现队列

### 题目描述

用两个栈来实现一个队列，完成队列的Push和Pop操作。 队列中的元素为int类型。具体实现思路如下：当 stack2 为空的时候，再把 stack1 的内容全部放到 stack2 里面，从 stack1 入队，stack2 出队。

![](http://ojt6zsxg2.bkt.clouddn.com/db1a4a13ee3697ba144dae3c999c114c.png)

```
var q1 = [];
var q2 = [];
function push(node)
{
    q1.push(node);
}
function pop()
{
    if(q2.length === 0) {
        while(q1.length !== 0) {
            q2.push(q1.pop());
        }
    }
    return q2.pop();
}
module.exports = {
    push : push,
    pop : pop
};
```





