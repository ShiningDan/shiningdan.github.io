---
title: 剑指offer题集
date: 2017-04-04 19:59:11
categories: coding
tags:
  - algorithm
---

本笔记是刷 《剑指 offer》 题集中题目的笔记记录。

<!--more-->

《剑指 offer》上的真题可以在 [牛客网](https://www.nowcoder.com/ta/coding-interviews) 上找到。

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

第一个方法是创建新的字符串，进行拼接，但是这样会使用多余的空间。**前两种方法都是要使用额外的空间的，是否使用额外的空间要提前询问面试官**

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

但是如果**不考虑使用额外的空间**，在原来的字符串上进行修改，在替换的时候，就需要将原来的字符串进行后移，一个空格需要向后移动三位，然后分别填充 `%`、`2`、`0`。

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

如果用栈来实现这个函数，而递归本身就是一个栈结构，可以使用递归，先输出自己的后一个节点，再输出自己：

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

## 旋转数组的最小数字

### 题目描述

把一个数组最开始的若干个元素搬到数组的末尾，我们称之为数组的旋转。
输入一个非递减排序的数组的一个旋转，输出旋转数组的最小元素。
例如数组{3,4,5,1,2}为{1,2,3,4,5}的一个旋转，该数组的最小值为1。
NOTE：给出的所有元素都大于0，若数组大小为0，请返回0。

使用折半查找法：

```
function minNumberInRotateArray(rotateArray)
{
    var index1 = 0;
    var index2 = rotateArray.length-1;
    var indexMid = index1;
    while (rotateArray[index1] >= rotateArray[index2]) {
        if (index2 - index1 == 1 || index2 - index1 ==0) {
          indexMid = index2;
          break;
        }
        indexMid = Math.floor((indeax1 + index2)/2);
        if (rotateArray[indexMid] >= rotateArray[index1]) {
            index1 = indexMid;
        } else {
            index2 = indexMid;
        }
    }
    return rotateArray[indexMid];
}
module.exports = {
    minNumberInRotateArray : minNumberInRotateArray
};
```

## 斐波那契数

### 题目描述

大家都知道斐波那契数列，现在要求输入一个整数n，请你输出斐波那契数列的第n项。
n<=39

使用递归的方法：

```
function Fibonacci(n)
{
    if (n === 0)
        return 0;
    if (n === 1)
        return 1;
    return Fibonacci(n-1) + Fibonacci(n-2);
}
module.exports = {
    Fibonacci : Fibonacci
};
```

但是使用递归的方法时，在求解的过程中，有很多结点重复计算，这使得计算量随着结点大大增加。

所以，我们可以将计算过程中涉及的中间值保存下来，可以通过 `f(0)` 和 `f(1)` 计算得到 `f(2)`，然后通过 `f(1)` 和`f(2)` 计算得到 `f(3)`，一直到`f(n)`

```
function Fibonacci(n)
{
    var fibOne = 0, fibTwo = 1, num = 1;
    if (n <= 1)
        return n;
    var temp;
    while(num < n) {
        temp = fibOne + fibTwo;
        fibOne = fibTwo;
        fibTwo = temp;
        ++num;
    }
    return temp;
}
module.exports = {
    Fibonacci : Fibonacci
};
```

## 跳台阶

### 题目描述

一只青蛙一次可以跳上1级台阶，也可以跳上2级。求该青蛙跳上一个n级的台阶总共有多少种跳法。

其实这道题的思路和斐波那契数的思路很像。如果只有 1 个台阶，就只有 1 种跳法。如果只有 2 个台阶。就有两种跳法，一个是分两次，另一个是一次跳两个。

如果有 `n` 个台阶，就有 `jumpFloor(n)` 种跳法，其中，`jumpFloor(n)` 的数等于 `jumpFloor(n-1) + jumpFloor(n-2)`

所以这个问题和斐波那契数是一样的。

```
function jumpFloor(number)
{	
    if (number === 1) {
        return 1;
    } else if (number === 2) {
        return 2;
    } else {
        var jNum1 = 1, jNum2 = 2, tmp;
        while(number > 2) {
            tmp = jNum2;
            jNum2 = jNum2 + jNum1;
            jNum1 = tmp;
            --number;
        }
        return jNum2;
    }
}
```

## 变态跳台阶

### 题目描述

一只青蛙一次可以跳上1级台阶，也可以跳上2级……它也可以跳上n级。求该青蛙跳上一个n级的台阶总共有多少种跳法。

这里具体的分析思路可以参考 [第一个回答](https://www.nowcoder.com/questionTerminal/22243d016f6b47f2a6928b4313c85387)，大致含义就是用数学归纳法证明 `f(n) = Math.pow(2, n-1)`

```
function jumpFloorII(number)
{
    return Math.pow(2, number-1);
}
```

## 矩形覆盖

### 题目描述

我们可以用 `2*1`的小矩形横着或者竖着去覆盖更大的矩形。请问用n个 `2*1`的小矩形无重叠地覆盖一个 `2*n` 的大矩形，总共有多少种方法？

其实这也是一个斐波那契数的问题，`f(n) = f(n-1) + f(n-2)`。

**像这种动态规划的问题，其实求解的方法都类似于数学归纳法，找到 `f(n)` 和之前状态的直接关系，然后构造公式，并且提供初始值。**

```
function rectCover(number)
{
    if (number === 0) {
        return 0;
    } else if (number === 1) {
        return 1;
    } else if (number === 2) {
        return 2;
    } else {
        var rNum1 = 1, rNum2 = 2, tmp;
        while(number > 2) {
            tmp = rNum2;
            rNum2 = rNum1 + rNum2;
            rNum1 = tmp;
            --number;
        }
        return rNum2;
    }
}
```

## 二进制中 1 的个数

### 题目描述

输入一个整数，输出该数二进制表示中1的个数。其中负数用补码表示。

```
function NumberOf1(n)
{
    var count = 0;
    while(n) {
        ++ count;
        n = n & (n - 1);
    }
    return count;
}
```

解答详解可以参见 [详解]（https://www.nowcoder.com/profile/1498510/codeBookDetail?submissionId=8851153）这里我要谈的是 JS 中的数值的二进制表达。

默认在 JS 中，数值是用 64 位的浮点型二进制进行存储的，但是只要对 JS 中的任何数字做位运算操作系统内部都会将其转换成 32 位的整型。

在计算机中，正整数存储的是正常的二进制真值，负数存储的是二进制补码（真值码取反加一后加上符号位）。所以计算负数二进制中 1 的个数的时候，得到的是正整数补码中 1 的位数。并且在进行位运算的时候，负数使用的是补码。


## 数值的整数次方

### 题目描述

给定一个double类型的浮点数base和int类型的整数exponent。求base的exponent次方。

**在这道题中，我们可能还需要考虑指数是 0 或者负数的情况！**

所以，第一次的解法如此：

```
function Power(base, exponent)
{
    if (exponent < 0) {
        if (base === 0) {
            throw new Error('the deno should not be 0');
        } else {
            var absexponent = -exponent, result = 1;
            for (var i = 0; i < absexponent; i++) {
                result *= base;
            }
			result = 1/result;
            return result;
        }
    } else if (exponent === 0) {
        return 1;
    } else {
        var result = 1;
        for (var i = 0; i < exponent; i++) {
            result *= base;
        }
        return result;
    }
}
```

但是，使用这种解法的问题就在于，如果 `exponent` 的值为 `32`，我们就需要计算 32 次，其实 32 次方就等于 16 的平方的平方，这个问题就变成了之前的斐波拉契数列的问题，我们可以由此来减少计算的步骤：

```
function PowerPosExp(base, exponent) {
    var result = base;
    while(exponent > 1) {
        if (exponent & 0x01 === 1) {
            result = result * result * base;
            exponent = (exponent - 1) / 2;
        } else {
            result = result * result;
            exponent = exponent / 2;
        }
    }
    return result;
}
function Power(base, exponent)
{
    if (exponent < 0) {
        if (base === 0) {
            throw new Error('the deno should not be 0');
        } else {
            var result = PowerPosExp(base, -exponent);
						result = 1/result;
            return result;
        }
    } else if (exponent === 0) {
        return 1;
    } else {
        var result = PowerPosExp(base, exponent);
        return result;
    }
}
```

## 调整数组顺序使奇数位于偶数前面

### 题目描述

输入一个整数数组，实现一个函数来调整该数组中数字的顺序，使得所有的奇数位于数组的前半部分，所有的偶数位于位于数组的后半部分，并保证奇数和奇数，偶数和偶数之间的相对位置不变。

```
/**
 * 1.要想保证原有次序，则只能顺次移动或相邻交换。
 * 2.i从左向右遍历，找到第一个偶数。
 * 3.j从i+1开始向后找，直到找到第一个奇数。
 * 4.将[i,...,j-1]的元素整体后移一位，最后将找到的奇数放入i位置，然后i++。
 * 5.终止条件：j向后遍历查找失敗。
 */
function reOrderArray(array)
{
    for (var i = 0; i < array.length; i++) {
        if (array[i]%2 == 1) {
            continue;
        }
        var j = i + 1;
        while(j < array.length) {
            if (array[j]%2 == 1) {
            	while(j > i) {
                    var tmp = array[j];
                    array[j] = array[j - 1];
                    array[j - 1] = tmp;
                    --j;
                }  
                break;
            }
            ++j;
        }
    }
		return array;
}

let a = [1, 2, 3, 4, 5, 6, 6, 7];
console.log(reOrderArray(a));
```

## 链表中倒数第k个结点

### 题目描述

输入一个链表，输出该链表中倒数第k个结点。

为了实现一次遍历就找到倒数第 k 个节点，我们可以定义两个指针，他们之间的节点数相差 `k - 1`。

然后，这道题考虑的是鲁棒性，所以，我们要考虑

1. 如果头指针为空
2. 如果链表数少于 k
3. 如果输入的 `k` 为 `0`

```
function FindKthToTail(head, k)
{
    if (head === null || k <= 0) {
        return null;
    }
    var pA = head, pB = null;
    for (var i = 0; i < k - 1; i++) {
        if (pA.next === null) {
            return null;
        }
        pA = pA.next;
    }
    pB = head;
    while(pA.next !== null) {
        pA = pA.next;
        pB = pB.next;
    }
    return pB;
}
```

## 翻转链表

### 题目描述

输入一个链表，反转链表后，输出链表的所有元素。

这道题需要考虑，链表头为 `null`，链表只有一个节点和多个节点的情况。

```
function ReverseList(pHead)
{
    if (pHead) {
        if (pHead.next === null) {
            return pHead;
        }
        var pNext = pHead.next,
            p = pHead,
            pPrev = null;
        p.next = pPrev;
        while(pNext.next !== null) {
            pPrev = p;
            p = pNext;
            pNext = pNext.next;
            p.next = pPrev;
        }
        pNext.next = p;
        return pNext;
    }
    return null;
}
```

## 合并两个排序的链表

### 题目描述

输入两个单调递增的链表，输出两个链表合成后的链表，当然我们需要合成后的链表满足单调不减规则。

我们需要考虑的是如果给的链表是 `null`，该如何处理

```
function Merge(pHead1, pHead2)
{
    if (pHead1 === null) {
        return pHead2;
    } else if (pHead2 === null) {
        return pHead1;
    } else {
        if (pHead1.val <= pHead2.val) {
            pHead1.next = Merge(pHead1.next, pHead2);
            return pHead1;
        } else {
            pHead2.next = Merge(pHead1, pHead2.next);
            return pHead2;
        }
        
    }
}
```

## 树的子结构

### 题目描述

输入两棵二叉树A，B，判断B是不是A的子结构。（ps：我们约定空树不是任意一个树的子结构）




















