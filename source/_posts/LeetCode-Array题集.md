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

