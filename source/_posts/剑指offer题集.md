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

**在解决链表，树等类型的题，一定要注意判断输入是否为 null**

```
function HasSubtree(pRoot1, pRoot2)
{
    if (pRoot1 === null || pRoot2 === null) {
        return false;
    }
    return isSubtree(pRoot1, pRoot2) || HasSubtree(pRoot1.left, pRoot2) || HasSubtree(pRoot1.right, pRoot2);
    
}
function isSubtree(pRoot1, pRoot2) {
    if (pRoot2 === null) {
        return true;
    }
    if (pRoot1 === null) {
        return false;
    }
    if (pRoot1.val === pRoot2.val) {
        return isSubtree(pRoot1.left, pRoot2.left) && isSubtree(pRoot1.right, pRoot2.right);
    }
    return false;
}
```

## 二叉树的镜像

### 题目描述

操作给定的二叉树，将其变换为源二叉树的镜像。 
输入描述:
二叉树的镜像定义：源二叉树 

```
    	    8
    	   /  \
    	  6   10
    	 / \  / \
    	5  7 9 11
    	镜像二叉树
    	    8
    	   /  \
    	  10   6
    	 / \  / \
    	11 9 7  5

```

```
function Mirror(root)
{
    if (root === null) {
        return null;
    }
    var tmp = root.left;
    root.left = root.right;
    root.right = tmp;
    root.left = Mirror(root.left);
    root.right = Mirror(root.right);
    return root;
}
```

## 顺时针打印矩阵

### 题目描述

输入一个矩阵，按照从外向里以顺时针的顺序依次打印出每一个数字，例如，如果输入如下矩阵： 1 2 3 4 5 6 7 8 9 10 11 12 13 14 15 16 则依次打印出数字1,2,3,4,8,12,16,15,14,13,9,5,6,7,11,10.


```
function printMatrix(matrix)
{
    if (matrix === null || matrix.length === 0) {
        return ;
    }
    var row = matrix.length, col = matrix[0].length;
		if (col === undefined) {
        return matrix;
    }
    var result = [], start = 0;
    while(row > 2 * start && col > 2 * start){
				var endx = col - start;
				var endy = row - start;
				for (var i = start; i < endx; i++) {
						result.push(matrix[start][i])
				}
				if(start<endy-1){
						for (var j= (start + 1); j < endy; j++) {
						result.push(matrix[j][endx-1]);
						}
				}
				if(start<(endx-1)&&start<(endy-1)){
						for (var  m = endx-2; m >= start; m--) {
								result.push(matrix[endy-1][m])
						}
				}
				if(start<(endx-1)&&start<endy){
						for (var n = endy - 2; n >= start+1; n--) {
								result.push(matrix[n][start])
						}
				}
				start++
		}
    return result;
}
```

## 包含 min 函数的栈

### 题目描述

定义栈的数据结构，请在该类型中实现一个能够得到栈最小元素的min函数。在该栈中，调用 min、push、pop 的时间复杂度为 O(1)

```
var d_data = [];
var m_data = [];
function push(node)
{
    d_data.push(node);
    var min = m_data[m_data.length - 1];
    if (min === undefined) {
        m_data.push(node);
    } else {
        m_data.push(Math.min(min, node))
    }
}
function pop()
{
    if (m_data.length === 0) {
        throw new Error('Stack is Empty')
    } else {
        m_data.pop();
    	return d_data.pop();
    }
}
function top()
{
    // write code here
}
function min()
{
    if (m_data.length === 0) {
        throw new Error('Stack is Empty')
    } else {
        return m_data[m_data.length - 1];
    }
}
```

## 栈的压入、弹出序列

### 题目描述

输入两个整数序列，第一个序列表示栈的压入顺序，请判断第二个序列是否为该栈的弹出顺序。假设压入栈的所有数字均不相等。例如序列1,2,3,4,5是某栈的压入顺序，序列4，5,3,2,1是该压栈序列对应的一个弹出序列，但4,3,5,1,2就不可能是该压栈序列的弹出序列。（注意：这两个序列的长度是相等的）

**这道题使用了一个新的辅助栈来解决问题**

```
var stack = [];
function IsPopOrder(pushV, popV){
    for (var i = 0; i < pushV.length; i++) {
        if (pushV[i] === popV[0]) {
            popV.shift();
			while(stack.length > 0) {
				if (stack[stack.length - 1] === popV[0]) {
					stack.pop();
					popV.shift();
				} else {
					break;
				}
			}
        } else {
			stack.push(pushV[i]);
		}
    }
    if (stack.length === 0) {
        return true;
    }
    return false;
}
```

## 从上往下打印二叉树

### 题目描述

从上往下打印出二叉树的每个节点，同层节点从左至右打印。

**这道题就是广度优先遍历，使用一个辅助的队列进行遍历**

```
function PrintFromTopToBottom(root)
{
    if (root === null) {
        return [];
    }
    var a = [], result = [];
    a.push(root);
    while(a.length > 0) {
        var elem = a.shift();
        if (elem) {
            result.push(elem.val);
            a.push(elem.left);
            a.push(elem.right);
        } 
    }
    return result;
}
```

## 二叉搜索树的后序遍历序列

### 题目描述

输入一个整数数组，判断该数组是不是某二叉搜索树的后序遍历的结果。如果是则输出Yes,否则输出No。假设输入的数组的任意两个数字都互不相同。

```
function VerifySquenceOfBST(sequence)
{
    if (sequence === null || sequence.length === 0) {
        return false;
    } 
    if (sequence.length < 3) {
        return true;
    }
    var i = 0, root = sequence[sequence.length - 1];
    for (; i < sequence.length - 1; i++) {
        if (sequence[i] > root) {
            break;
        }
    }
	var j = i;
	for(;j < sequence.length - 1; j++) {
		if (sequence[j] < root) {
			return false;
		}
	}
	if (i > 0 && i < sequence.length - 1) {
		return (VerifySquenceOfBST(sequence.slice(0, i)) && VerifySquenceOfBST(sequence.slice(i, sequence.length - 1)))
	} else if (i > 0) {
		return VerifySquenceOfBST(sequence.slice(0, i));
	} else if (i < sequence.length - 1) {
		return VerifySquenceOfBST(sequence.slice(i, sequence.length - 1));
	}
}
```

## 二叉树中和为某一值的路径

### 题目描述

输入一颗二叉树和一个整数，打印出二叉树中结点值的和为输入整数的所有路径。路径定义为从树的根结点开始往下一直到叶结点所经过的结点形成一条路径。

**这道题其实是深度优先遍历，并且使用一个栈结构来存储路径**

在计算每个节点的时候，要先把节点的值入栈，执行完该节点以及该节点的子节点的时候，要把该节点出栈。

**在 `result` 中存储的值是 `path` 的一个深拷贝，所以使用 `path.slice()` 创建一个新的数组**

```
var result = [], path = [];
function FindPath(root, expectNumber)
{
    if (root === null) {
        return []
    }
    cal(root, expectNumber);
    return result;
}
function cal(root, expectNumber) {
	path.push(root.val);
 	if (expectNumber === root.val && root.left === null && root.right === null) {
        result.push(path.slice());
    } else {
        if (root.left !== null) {
            cal(root.left, expectNumber - root.val);
        }
        if (root.right !== null) {
            cal(root.right, expectNumber - root.val);
        }
    } 
	path.pop();
}
```

## 复杂链表的复制

### 题目描述

输入一个复杂链表（每个节点中有节点值，以及两个指针，一个指向下一个节点，另一个特殊指针指向任意一个节点），返回结果为复制后复杂链表的head。（注意，输出结果中请不要返回参数中的节点引用，否则判题程序会直接返回空）

**这个算法中，指向任意一个节点的寻找，是 O(n ^ 2) 时间复杂度。**为了简化寻找任意一个节点，我们可以在创建下一个节点的时候，使用 HashMap 存储一个`<N, N'>` 的哈希表，这就是一个用空间换时间的方法。

第二种方法是：创建的 `N'` 链接到 `N` 的后面，这样就可以根据 `N` 找到 `N'` 指向的的任意节点

```
function RandomListNode(x){
    this.label = x;
    this.next = null;
    this.random = null;
}
function Clone(pHead)
{
    if (pHead !== null) {
        cloneNodes(pHead);
        ConnectRandomNodes(pHead);
        return ReconnectNodes(pHead);
    } else {
        return null
    }
}
function cloneNodes(pHead) {
    var cNode = new RandomListNode(pHead.label);
    var nextNode = pHead.next;
    pHead.next = cNode;
    cNode.next = nextNode;
    if(nextNode !==  null) {
        cloneNodes(nextNode);
    }
}
function ConnectRandomNodes(pHead) {
    if (pHead.random) {
        var rNode = pHead.random,
            cNode = pHead.next;
        cNode.random = rNode.next;
    }
    if (pHead.next.next) {
        ConnectRandomNodes(pHead.next.next);
    }
}
function ReconnectNodes(pHead) {
    var cPHead = pHead.next, nNode = pHead.next;
    while(nNode.next) {
        nNode.next = nNode.next.next;
        nNode = nNode.next;
    }
    return cPHead;
}
```

## 二叉搜索树与双向链表

### 题目描述

输入一棵二叉搜索树，将该二叉搜索树转换成一个排序的双向链表。要求不能创建任何新的结点，只能调整树中结点指针的指向。

二叉搜索树中，左节点的值总是小于右节点的值，生成的双向链表中，前一项的值总是小于后一项的值。

**相当于对二叉搜索树的中序遍历。**

在这里使用的是非递归方法：

```
function Convert(pRootOfTree)
{
    if (pRootOfTree === null) {
        return null;
    }
    var stack = [], p = pRootOfTree, head, pre;
    var isFirst = true;
    while(p !== null || stack.length !== 0) {
        while(p) {
            stack.push(p);
            p = p.left;
        }
        var node = stack.pop();
        if (isFirst) {
            head = node;
            pre = node;
            isFirst = false;
        } else {
            pre.right = node;
            node.left = pre;
            pre = node;
        }
        p = node.right;
    }
    return head;
}
```

## 字符串的排列

### 题目描述

输入一个字符串,按字典序打印出该字符串中字符的所有排列。例如输入字符串abc,则打印出由字符a,b,c所能排列出来的所有字符串abc,acb,bac,bca,cab和cba。 

```
function Permutation(str)
{
    var result = [];
    if (str === null || str.length === 0) {
        return result;
    }
    var arr = Array.from(str);
    reformStr(arr, 0, result);
	result = Array.from(new Set(result));
    return result;
}
function reformStr(arr, start, result) {
	result.push(arr.join(""));
	for (var i = start; i < arr.length; i++) {
		if (arr[start] !== arr[i]) {
			[arr[start], arr[i]] = [arr[i], arr[start]];
			reformStr(arr, start + 1, result);
			[arr[start], arr[i]] = [arr[i], arr[start]]; 
		} else {
			reformStr(arr, start + 1, result);
		}
	}
}
```

## 数组中出现次数超过一半的数字

### 题目描述

数组中有一个数字出现的次数超过数组长度的一半，请找出这个数字。例如输入一个长度为9的数组{1,2,3,2,2,2,5,4,2}。由于数字2在数组中出现了5次，超过数组长度的一半，因此输出2。如果不存在则输出0。

**方法一，可以对数组进行排序，因为该数出现的次数超过数组长度的一半，所以排序后该数字必定出现在排序数组中的第 `n/2` 个位置上。**

**方法二，我们在遍历数组的时候保存两个值，第一个是数组的数字，第二个是出现的次数。如果遍历下一个数字和保存的数字相同，则该次数加一；如果不同，则该次数减一。如果次数减到 0，则保存下一个数字。**

这里用的是第二种方法，第二种方法的时间复杂度是 O(n)，比排序的时间复杂度要小。

```
function MoreThanHalfNum_Solution(numbers)
{
    if (numbers === null || numbers.length === 0) {
		return 0;
	}
    var val = numbers[0], time = 0;
    for (var i = 0; i < numbers.length; i++) {
        if (val === numbers[i]) {
            ++time;
        } else if (time > 0) {
            --time;
        } else {
            val = numbers[i];
            ++time;
        }
    }
    var k = 0;
	for (var i = 0; i < numbers.length; i++) {
		if (numbers[i] == val) {
			k++;
		}
	}
	return (k > Math.floor(numbers.length/2) ? val : 0);
}
```

## 最小的K个数

### 题目描述

[解题思路](https://www.nowcoder.com/profile/7551561/codeBookDetail?submissionId=11048551)

输入n个整数，找出其中最小的K个数。例如输入4,5,1,6,2,7,3,8这8个数字，则最小的4个数字是1,2,3,4,。

**如果是先对所有的数据进行排序后再获得最小的 K 个数，这种思路的时间复杂度是 O(n)**

**还可以参考快排的方法，首先找到第 K 小的数字，然后用快排，将数组左右分割即可，该算法的时间复杂度是 O(n)**

**还可以构造最小堆，时间复杂度是 O(nlogk)，或者维护一个只有 K 大小的排序数组。**

我这里使用的是类似快排的方法。

```
function GetLeastNumbers_Solution(input, k)
{
    if (input === null || input.length < k) {
        return []
    }
    if (input.length === k) {
        return input;
    }
    var start = 0, end = input.length - 1, len = 0;
    var index = Partition(input, start, end);
    while(index != k - 1) {
        if (index < k - 1) {
            index = Partition(input, index + 1, end);
        } else {
            index = Partition(input, start, index - 1);
        }
    }
    return input.slice(0, k);
}
function Partition(input, start, end) {
	var key = input[start], low = start, high = end;
    while(low < high) {
        while(low < high && key < input[high]) {
            --high;
        }
        input[low] = input[high];
        while(low < high && key > input[low]) {
            ++low;
        }
        input[high] = input[low];	
    }
    input[low] = key;
    return low;
}
```

## 连续子数组的最大和

### 题目描述

HZ偶尔会拿些专业问题来忽悠那些非计算机专业的同学。今天测试组开完会后,他又发话了:在古老的一维模式识别中,常常需要计算连续子向量的最大和,当向量全为正数的时候,问题很好解决。但是,如果向量中包含负数,是否应该包含某个负数,并期望旁边的正数会弥补它呢？例如:{6,-3,-2,7,-15,1,2,2},连续子向量的最大和为8(从第0个开始,到第3个为止)。你会不会被他忽悠住？(子向量的长度至少是1)

[解答](https://www.nowcoder.com/profile/284008/codeBookDetail?submissionId=9704055)


本方法是使用动态规划的方法：

```
function FindGreatestSumOfSubArray(array)
{
    if (array.length === 0) {
        return 0;
    }
    var max = array[0], temp = array[0];
    for (var i = 1; i < array.length; i++) {
        temp = temp < 0 ? array[i] : array[i] + temp;
        max = max > temp ? max : temp;
    }
    return max;
}
```

## 整数中1出现的次数（从1到n整数中1出现的次数）

### 题目描述

求出1~13的整数中1出现的次数,并算出100~1300的整数中1出现的次数？为此他特别数了一下1~13中包含1的数字有1、10、11、12、13因此共出现6次,但是对于后面问题他就没辙了。ACMer希望你们帮帮他,并把问题更加普遍化,可以很快的求出任意非负整数区间中1出现的次数。

[解答](https://www.nowcoder.com/profile/9571580/codeBookDetail?submissionId=12601864)

## 把数组排成最小的数

### 题目描述

输入一个正整数数组，把数组里所有数字拼接起来排成一个数，打印能拼接出的所有数字中最小的一个。例如输入数组{3，32，321}，则打印出这三个数字能排成的最小数字为321323。

解决的方法是，使用 `Array.protorype.sort` 方法中接收的比较函数来比较两个数字。由于数字组成的新的数值可能溢出，所以比较的方法就是把数字组成字符串，然后比较字符串的大小。

```
function PrintMinNumber(numbers)
{
    if (numbers.length === 0) {
        return "";
    }
    var a = numbers.sort(compare);
    return a.join("");
}
function compare(a, b) {
    return a + "" + b > b + "" + a ? 1 : -1 ;
}
```

## 丑数

### 题目描述

把只包含因子2、3和5的数称作丑数（Ugly Number）。例如6、8都是丑数，但14不是，因为它包含因子7。 习惯上我们把1当做是第一个丑数。求按从小到大的顺序的第N个丑数。

在这道题上，每个丑数都是前一个丑数乘以 2、3、5 得到的，关键在于如何确保数组中的丑数是排好序的。

**我们没有必要把每个丑数都乘以 2、3、5，而是记录上一个被乘以 2、3、5的丑数即可。**


**所以，我们在迭代数组中的数据计算下一个数据的时候，可以保存上一次迭代的节点。这样既方便计算，又减少了时间复杂度。**

```
function min(n1, n2, n3) {
    return n1 < n2 ? (n1 < n3 ? n1 : n3) : (n2 < n3 ? n2 : n3);
}
function GetUglyNumber_Solution(index)
{
    if (index < 1) {
        return 0;
    }
    var gulyNumber = [1];
    var nextIndex = 0;
    var T2index = 0, T3index = 0, T5index = 0;
    
    while (nextIndex + 1 < index) {
        var minNum = min(gulyNumber[T2index] * 2, gulyNumber[T3index] * 3, gulyNumber[T5index] * 5);
        gulyNumber.push(minNum);
        ++nextIndex;
        while(gulyNumber[T2index] * 2 <= gulyNumber[nextIndex]) {
            ++ T2index;
        }
        while(gulyNumber[T3index] * 3 <= gulyNumber[nextIndex]) {
            ++ T3index;
        }
        while(gulyNumber[T5index] * 5 <= gulyNumber[nextIndex]) {
            ++ T5index;
        }
    }
    return gulyNumber[nextIndex];
} 
```

## 第一个只出现一次的字符位置

### 题目描述

在一个字符串(1<=字符串长度<=10000，全部由字母组成)中找到第一个只出现一次的字符,并返回它的位置

这个题的解法，是用 map 或者对象存储字符串出现的位置。

```
function FirstNotRepeatingChar(str)
{
    if (str.length === 0) {
        return -1;
    }
    var map = {};
    for (var i = 0; i < str.length; i++) {
        var x = str[i];
        if (map[x]) {
           map[x] = -1; 
        } else {
            map[x] = i + 1;
        }
    }
    for (var i = 0; i < str.length; i++) {
    	if (map[str[i]] >= 0) {
            return i;
        }        
    }
}
```

## 数组中的逆序对

### 题目描述

在数组中的两个数字，如果前面一个数字大于后面的数字，则这两个数字组成一个逆序对。输入一个数组,求出这个数组中的逆序对的总数P。并将P对1000000007取模的结果输出。 即输出P%1000000007 
输入描述:

```
题目保证输入的数组中没有的相同的数字
数据范围：
	对于%50的数据,size<=10^4
	对于%75的数据,size<=10^5
	对于%100的数据,size<=2*10^5
```

输入例子:

```
1,2,3,4,5,6,7,0
```

输出例子:

```
7
```

[解答](https://www.nowcoder.com/profile/6050759/codeBookDetail?submissionId=12615137)

```
function InversePairs(data)
{
    if (data === null || data.length < 2) {
        return 0;
    } 
    var copy = data.slice();
    return mergeSort(data, copy, 0, data.length - 1);
}
function mergeSort(data, copy, start, end) {
    if (start === end) return 0;
    var mid = Math.floor((start + end) / 2);
    var left = mergeSort(data, copy, start, mid);
    var right = mergeSort(data, copy, mid+1, end);    
    var count = 0, p1 = mid, p2 = end , p3 = end;
    while(p1 >= start && p2 >= mid + 1) {
        if (copy[p1] > copy[p2]) {
            count += p2 - mid;
            data[p3--] = copy[p1--];
        } else {
            data[p3--] = copy[p2--];
        }
    }
    while (p1 >= start) {
        data[p3--] = copy[p1--];
    }
    while (p2 >= mid + 1) {
        data[p3--] = copy[p2--];
    }
    for (var i = start; i <= end; i++) {
        copy[i] = data[i];
    }
    return left + right + count
}
```

## 两个链表的第一个公共结点

### 题目描述


输入两个链表，找出它们的第一个公共结点。

```
function FindFirstCommonNode(pHead1, pHead2)
{
    if (pHead1 === null || pHead2 === null) {
        return null;
    }
    var pNext1 = pHead1, pNext2 = pHead2;
    var len = getlen(pHead1, pHead2);
    for (var i = 0; i < len; i++) {
        if (pNext1 === null) {
            pNext1 = pHead2;
        }
        if (pNext2 === null) {
            pNext2 = pHead1;
        }
        if (pNext1 === pNext2) {
            return pNext1;
        } else {
            pNext1 = pNext1.next;
            pNext2 = pNext2.next;
        }
    }
    return null;
}

function getlen(pHead1, pHead2) {
    var len = 0;
    while (pHead1 !== null) {
        ++len;
        pHead1 = pHead1.next;
    }
    while (pHead2 !== null) {
        ++len;
        pHead2 = pHead2.next;
    }
    return len;
}
```

## 数字在排序数组中出现的次数

### 题目描述

统计一个数字在排序数组中出现的次数。

因为是排序的数组，所以考虑使用二分查找法找到数组中第一个数字和最后一个数字。

```
function GetNumberOfK(data, k)
{
    if (data === null) {
        return 0;
    } else if (data.length === 0) {
        return 0;
    }
    var start = getFirstK(data, 0, data.length - 1, k);
    var end = getLastK(data, 0, data.length - 1, k);
		if (start === -1) {
			return 0;
		}
		return end - start + 1;
}
function getFirstK(data, start, end, k) {
    if (data.length === 0) {
        return -1;
    } else if (start === end) {
			if (data[start] === k) {
				return start;
			} else {
				return -1;
			}
		}
    var mid = Math.floor((start + end)/2);
    if (data[mid] === k) {
        if (mid === 0) {
            return mid;
        } else if (data[mid - 1] !== k) {
            return mid;
        } else {
					return getFirstK(data, start, mid - 1, k);
				}
    } else if (data[mid] < k) {
        return getFirstK(data, mid + 1, end, k)
    } else {
        return getFirstK(data, start, mid - 1, k);
    }
}
function getLastK(data, start, end, k) {
    if (data.length === 0) {
        return -1;
    } else if (start === end) {
			if (data[start] === k) {
				return start;
			} else {
				return -1;
			}
		}
    var mid = Math.floor((start + end)/2);
    if (data[mid] === k) {
        if (mid === data.length - 1) {
            return mid;
        } else if (data[mid + 1] !== k) {
            return mid;
        } else {
					return getLastK(data, mid + 1, end, k);
				}
    } else if (data[mid] < k) {
        return getLastK(data, mid + 1, end, k);
    } else {
        return getLastK(data, start, end - 1, k);
    }
}
```

## 二叉树的深度

### 题目描述

输入一棵二叉树，求该树的深度。从根结点到叶结点依次经过的结点（含根、叶结点）形成树的一条路径，最长路径的长度为树的深度。


```
function TreeDepth(pRoot)
{
    if (pRoot === null) {
        return 0;
    } else if (pRoot.left === null && pRoot.right === null) {
        return 1;
    }
    var len = 0, stack = [[pRoot]];
    while(stack.length !== 0) {
        var tmp = stack.pop(), nextStack = [];
        tmp.forEach(function(node) {
            if (node.left !== null) {
                nextStack.push(node.left)
            }
            if (node.right !== null) {
                nextStack.push(node.right);
            }
        });
        if (nextStack.length > 0) {
            stack.push(nextStack);
        }
        ++len;
    } 
    return len;
}
```

这道题还可以使用递归的方法来求解：

```
function TreeDepth(pRoot)
{
 
    if(pRoot == null){
        return 0;
    }
    var left = TreeDepth(pRoot.left)+1;
    var right = TreeDepth(pRoot.right)+1;
    return Math.max(left, right);
}
```

## 平衡二叉树

### 题目描述

输入一棵二叉树，判断该二叉树是否是平衡二叉树。

使用后序遍历来解决这个问题

```
function IsBalanced_Solution(pRoot)
{
    var result = isBalanced(pRoot);
    return result[0];
}
function isBalanced(root) {
    if (root === null) {
        return [true, 0];
    }
    var left = isBalanced(root.left);
    var right = isBalanced(root.right);
    if (left[0] && right[0]) {
        if (Math.abs(left[1] - right[1]) > 1) {
            return [false, 0];
        } else {
            return [true, left[1] > right[1] ? left[1] + 1: right[1] + 1];
        }
    } else {
        return [left[0] && right[0], 0]
    }
}
```

