---
title: LeetCode-Array题集
date: 2017-02-03 17:39:06
categories: coding
tags:
  - LeetCode
  - Array
  - Java
  - JavaScript
---

下面所记录的是我在刷 [LeetCode](https://leetcode.com) Array 相关的题目自己的解法。

## [1. Two Sum Easy](https://leetcode.com/problems/two-sum/)

Given an array of integers, return **indices** of the two numbers such that they add up to a specific target.

You may assume that each input would have **exactly** one solution, and you may not use the same element twice.

**Example:**

```
Given nums = [2, 7, 11, 15], target = 9,

Because nums[0] + nums[1] = 2 + 7 = 9,
return [0, 1].
```

根据题目中的要求，找到组成和为 target 的数组中两个数的下标，并且只能遍历一遍，所以在遍历数组的时候，需要开辟额外的空间来缓存已经遍历的数据。在我的解法中，使用了 Map 来缓存已经遍历的数据，key 为数组元素的大小，value 代表着数据对应的数组下标。

### JavaScript Solution

```
var twoSum = function(nums, target) {
	var tmp =  new Map();
	for (var i=0; i< nums.length; i++) {
		var another = target - nums[i];
		if(tmp.has(another)) {
			return [tmp.get(another), i];
		} else {
			tmp.set(nums[i], i)
		}
	}
	return null;
};

twoSum([2, 7, 11, 15], 9);
```


### Java Solution

```
public int[] twoSum(int[] nums, int target) {
        Map<Integer, Integer> tmp = new HashMap<Integer, Integer>();
        for(int i =0; i<nums.length; i++) {
            int another = target - nums[i];
            if(tmp.containsKey(another)) {
                int[] result = new int[2];
                result[0] = tmp.get(another);
                result[1] = i;
                return result;
            } else {
                tmp.put(nums[i], i);
            }
        }
        return null;
    }
```

<!--more-->

## [26. Remove Duplicates from Sorted Array](https://leetcode.com/problems/remove-duplicates-from-sorted-array/)

en a sorted array, remove the duplicates in place such that each element appear only once and return the new length.

Do not allocate extra space for another array, you must do this in place with constant memory.

For example,
Given input array nums = `[1,1,2]`,

Your function should return length = `2`, with the first two elements of nums being 1 and 2 respectively. It doesn't matter what you leave beyond the new length.

这个问题要考虑的地方在于，不仅需要知道一个数组中不重复的项有多少个，还需要对输入的排序数组 nums 进行重新排序，将重复的项放在数组后面，不重复的项放在数组前面，所以在遍历的时候还需要进行元素的拷贝。

### JavaScript Solution

```
var removeDuplicates = function(nums) {
	if(nums.length === 0) {
		return 0;
	}
    let length = 0;
    for(let i=0; i<nums.length; i++) {
    	if(nums[length] !== nums[i]) {
    		nums[++length] = nums[i];
    	}
    }
    return ++length;
};

removeDuplicates([1, 2, 3, 4, 4, 4, 5, 5, 5, 6]);
```

### Java Solution

```
public int removeDuplicates(int[] nums) {
        if(nums.length == 0) {
            return 0;
        }
        int length = 0;
        for (int i=0; i<nums.length; i++) {
            if(nums[length] != nums[i]) {
                nums[++length] = nums[i];
            }
        }
        return ++length;
    }
```

### [27. Remove Element](https://leetcode.com/problems/remove-element/)


Given an array and a value, remove all instances of that value in place and return the new length.

Do not allocate extra space for another array, you must do this in place with constant memory.

The order of elements can be changed. It doesn't matter what you leave beyond the new length.

**Example:**
Given input array nums = `[3,2,2,3]`, val = `3`

Your function should return length = 2, with the first two elements of nums being 2.

**Hint:**

1. Try two pointers.
2. Did you use the property of "the order of elements can be changed"?
3. What happens when the elements to remove are rare?

在做这道题的想法，是从前向后遍历，如果 nums[i] === val，则从后面选取一个不等于 val 的值替代当前位置。

**这个想法需要注意的地方是，一个指针从前向后，另一个指针从后向前，需要注意前指针和后指针重合的临界位置判断。**

#### JavaScript Solution

```
var removeElement = function(nums, val) {
	let len = nums.length;
    for (let i = 0; i < nums.length; i++) {
    	while (nums[i] === val && i < len) {
    		nums[i] = nums[--len];
    	}
    }
    return len;
};
removeElement([3,2, 2, 3], 3);
```


#### Java Solution

```
public int removeElement(int[] nums, int val) {
        int back = nums.length;
        for(int i = 0; i < nums.length; i++) {
            while (nums[i] == val && i < back) {
                nums[i] = nums[--back];
            }
            if (i >= back) break;
        }
        return back;
    }
```

#### Best Solution

最好的想法是从前往后遍历，将遇到的所有不等于 val 的项从第一个开始写。

```
int removeElement(int A[], int n, int elem) {
    int begin=0;
    for(int i=0;i<n;i++) if(A[i]!=elem) A[begin++]=A[i];
    return begin;
}
```

### [35. Search Insert Position](https://leetcode.com/problems/search-insert-position/)

Given a sorted array and a target value, return the index if the target is found. If not, return the index where it would be if it were inserted in order.

You may assume no duplicates in the array.

Here are few examples.

`[1,3,5,6], 5` → 2
`[1,3,5,6], 2` → 1
`[1,3,5,6], 7` → 4
`[1,3,5,6], 0` → 0

我的想法是最基本的想法，就是从头到尾进行遍历，因为是排序好的数组，所以找到比 target 大的值，就可以返回该值。

#### JavaScript Solution

```
var searchInsert = function(nums, target) {
    if (nums.length === 0) return 0;
    let i;
    for(i = 0; i < nums.length; i++) {
    	if (nums[i] >= target) {
    		return i;
    	}
    }
    return nums.length;
};

searchInsert([1,3,5,6], 0)
```

#### Java Solution

```
public class Solution {
    public int searchInsert(int[] nums, int target) {
        if (nums.length == 0) return 0;
        int i;
        for (i = 0; i < nums.length; i++) {
            if(nums[i] >= target) return i;
        }
        return nums.length;
    }
}
```

#### Best Solution

最优解使用的是对半查找。使用对半查找有两个条件：

1. 数组已经排序好
2. 找到该数据在数组中的位置

使用对半查找，可以在很短的时间得到要查找的数据的位置，**值得学习**

```
public int searchInsert(int[] nums, int target) {
        int low = 0, high = nums.length-1;
        while(low<=high){
            int mid = (low+high)/2;
            if(nums[mid] == target) return mid;
            else if(nums[mid] > target) high = mid-1;
            else low = mid+1;
        }
        return low;
    }
```

### [53. Maximum Subarray](https://leetcode.com/problems/maximum-subarray/?tab=Description)

Find the contiguous subarray within an array (containing at least one number) which has the largest sum.

For example, given the array `[-2,1,-3,4,-1,2,1,-5,4]`,
the contiguous subarray `[4,-1,2,1]` has the largest sum = `6`.

下面是网上解法的分析：

大意是：这个问题的解法可以分成小问题进行求解，如果问题为 Q(i)，则结题思路可以变成 Q(i) 与 Q(i-1) 之间的关系是什么，以及Q(0)是什么。

Analysis of this problem:
Apparently, this is a optimization problem, which can be usually solved by DP. So when it comes to DP, the first thing for us to figure out is the format of the sub problem(or the state of each sub problem). The format of the sub problem can be helpful when we are trying to come up with the recursive relation.

At first, I think the sub problem should look like: `maxSubArray(int A[], int i, int j)`, which means the maxSubArray for A[i: j]. In this way, our goal is to figure out what `maxSubArray(A, 0, A.length - 1)` is. However, if we define the format of the sub problem in this way, it's hard to find the connection from the sub problem to the original problem(at least for me). In other words, I can't find a way to divided the original problem into the sub problems and use the solutions of the sub problems to somehow create the solution of the original one.

So I change the format of the sub problem into something like: `maxSubArray(int A[], int i)`, which means the maxSubArray for A[0:i ] which must has A[i] as the end element. Note that now the sub problem's format is less flexible and less powerful than the previous one because there's a limitation that A[i] should be contained in that sequence and we have to keep track of each solution of the sub problem to update the global optimal value. However, now the connect between the sub problem & the original one becomes clearer:

```
maxSubArray(A, i) = （maxSubArray(A, i - 1) > 0 ? maxSubArray(A, i - 1) : 0） + A[i]; 
```

#### JavaScript Solution

```
var maxSubArray = function(nums) {
    var dp = [], max = nums[0];
    dp[0] = nums[0];
    for (let i = 1; i < nums.length; i++) {
    	dp[i] = (dp[i-1] > 0 ? dp[i-1] : 0) + nums[i];
    	max = Math.max(max, dp[i]);
    }
    return max;
};  
```

#### Java Solution

```
public int maxSubArray(int[] nums) {
        int[] dp = new int[nums.length];
        int max = nums[0];
        dp[0] = nums[0];
        for (int i = 1; i < nums.length; i++) {
            dp[i] = (dp[i-1] > 0 ? dp[i-1] : 0) + nums[i];
            max = Math.max(max, dp[i]);
        }
        return max;
    }
```

### [66. Plus One](https://leetcode.com/problems/plus-one/)

Given a non-negative integer represented as a **non-empty** array of digits, plus one to the integer.

You may assume the integer do not contain any leading zero, except the number 0 itself.

The digits are stored such that the most significant digit is at the head of the list.

这个问题的逻辑并不是很难，只是要判断什么时候不需要计算，就提前结束函数。

#### JavaScript Solution

```
/**
 * @param {number[]} digits
 * @return {number[]}
 */
var plusOne = function(digits) {
    digits[digits.length-1] += 1;
    for(let i = digits.length-1; i >= 0; i--) {
    	if(digits[i] === 10) {
    		digits[i] = 0;
    		digits[i-1] += 1;
    	} else {
    		return digits;
    	}
    }
	digits[0] = 0;
	digits.unshift(1);
    return digits;
};
```

#### Java Solution

```
public int[] plusOne(int[] digits) {
        digits[digits.length-1] += 1;
        for(int i = digits.length-1; i > 0; i--) {
            if(digits[i] == 10) {
                digits[i] = 0;
                digits[i-1] += 1;
            }
        }
        if(digits[0] == 10) {
            int[] tmp= new int[digits.length+1];
            tmp[0] = 1;
            tmp[1] = 0;
            for(int i = 2; i < tmp.length; i++) {
                tmp[i] = digits[i-1];
            }
            digits = tmp;
        }
        return digits;
    }
```

#### Best Solution

他的方法好处就在如果数组中的数不全是 9 的时候，提前使用 return 函数结束，节省时间。

```
public int[] plusOne(int[] digits) {
        
    int n = digits.length;
    for(int i=n-1; i>=0; i--) {
        if(digits[i] < 9) {
            digits[i]++;
            return digits;
        }
        
        digits[i] = 0;
    }
    
    int[] newNumber = new int [n+1];
    newNumber[0] = 1;
    ...
    return newNumber;
}
```

### [88. Merge Sorted Array](https://leetcode.com/problems/merge-sorted-array/)

Given two sorted integer arrays nums1 and nums2, merge nums2 into nums1 as one sorted array.

**Note:**

You may assume that nums1 has enough space (size that is greater or equal to m + n) to hold additional elements from nums2. The number of elements initialized in nums1 and nums2 are m and n respectively.

该问题要解决的是，将 nums2 数组的数据 merge 到 nums1 数组中。如果是从前往后拷贝数据，则会导致原有数据被覆盖，所以使用的方法是从后向前写。Best Sulotion 中有一个很好的想法，是nums1 中还有数据，nums2 中没有数据的情况下，其实就不用处理了。**所以在思考问题的时候，要考虑到是不是有多处理的情况！**

#### JavaScript Solution

```
/**
 * @param {number[]} nums1
 * @param {number} m
 * @param {number[]} nums2
 * @param {number} n
 * @return {void} Do not return anything, modify nums1 in-place instead.
 */
var merge = function(nums1, m, nums2, n) {
	let j = m-1, k=n-1;
    for(let i = m+n-1; i >=0; i--) {
    	console.log(i + " " + j + " " + k);
    	if(j >= 0 && k >= 0) {
    		nums1[i] = (nums1[j] >= nums2[k] ? nums1[j--] : nums2[k--]);
    	} else if(j < 0){
    		nums1[i] = nums2[k--];
    	} else {
    		nums1[i] = nums1[j--];
    	}
    	
    }
};

merge([], 0, [7, 8], 2);
```

#### Java Solution

```
public class Solution {
    public void merge(int[] nums1, int m, int[] nums2, int n) {
        int i = m-1, j = n-1, k = m+n-1;
        while( i >= 0 && j >= 0) {
            nums1[k--] = (nums1[i] >= nums2[j] ? nums1[i--] : nums2[j--]);
        }
        while (j >= 0) {
            nums1[k--] = nums2[j--];
        }
    }
}
```

#### Best Solution

```
/**
 * @param {number[]} nums1
 * @param {number} m
 * @param {number[]} nums2
 * @param {number} n
 * @return {void} Do not return anything, modify nums1 in-place instead.
 */
var merge = function(nums1, m, nums2, n) {
	let i = m-1, j = n-1, k = m+n-1;
	while (i >= 0 && j >= 0) {
		if(nums1[i] >= nums2[j]) {
			nums1[k--] = nums1[i--];
		} else {
			nums1[k--] = nums2[j--];
		}
	}
	while(j >= 0) {
		nums1[k--] = nums2[j--];
	}
};

merge([], 0, [7, 8], 2);
```












