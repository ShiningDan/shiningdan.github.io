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

### [118. Pascal's Triangle](https://leetcode.com/problems/pascals-triangle/)

Given numRows, generate the first numRows of Pascal's triangle.

For example, given numRows = 5,
Return

```
[
     [1],
    [1,1],
   [1,2,1],
  [1,3,3,1],
 [1,4,6,4,1]
]
```

#### JavaScript Solution

```
/**
 * @param {number} numRows
 * @return {number[][]}
 */
var generate = function(numRows) {
    var pt = [];
    var row = [];
    for (let i = 0; i < numRows; i++) {
    	for (let j = row.length-1; j >0; j--) {
    		row[j] = row[j] + row[j-1]
    	}
    	row.push(1);
    	pt.push(row.concat());
    }
    return pt;
};
generate(5);
```

#### Java Solution

```
public class Solution {
    public List<List<Integer>> generate(int numRows) {
        List<List<Integer>> pt = new ArrayList<List<Integer>>();
        List<Integer> row = new ArrayList<Integer>();
        for(int i = 0; i < numRows; i++) {
            for(int j = row.size() - 1; j > 0; j--) {
                row.set(j, row.get(j) + row.get(j-1));
            }
            row.add(1);
            pt.add(new ArrayList<>(row));
        }
        return pt;
    }
}
```

### [119. Pascal's Triangle II](https://leetcode.com/problems/pascals-triangle-ii/)


Given an index k, return the kth row of the Pascal's triangle.

For example, given k = 3,
Return `[1,3,3,1]`.

Note:
Could you optimize your algorithm to use only O(k) extra space?

#### JavaScript Solution

```
/**
 * @param {number} rowIndex
 * @return {number[]}
 */
var getRow = function(rowIndex) {
    let row = [];
    for (let i = 0; i <= rowIndex; i++) {
    	for (let j = row.length - 1; j > 0; j--) 
    		row[j] = row[j-1] + row[j];
    	row.push(1);
    }
    return row;
};

getRow(3);
```

#### Java Solution

```
public class Solution {
    public List<Integer> getRow(int rowIndex) {
        List<Integer> row = new ArrayList<>();
        for (int i = 0; i <= rowIndex; i++) {
            for(int j = row.size() - 1; j > 0; j--) {
                row.set(j, row.get(j) + row.get(j-1));
            }
            row.add(1);
        }
        return row;
    }
}
```

### [121. Best Time to Buy and Sell Stock](https://leetcode.com/problems/best-time-to-buy-and-sell-stock/)

Say you have an array for which the ith element is the price of a given stock on day i.

If you were only permitted to complete at most one transaction (ie, buy one and sell one share of the stock), design an algorithm to find the maximum profit.

**Example 1:**

```
Input: [7, 1, 5, 3, 6, 4]
Output: 5

max. difference = 6-1 = 5 (not 7-1 = 6, as selling price needs to be larger than buying price
```

**Example 2:**

```
Input: [7, 6, 4, 3, 1]
Output: 0

In this case, no transaction is done, i.e. max profit = 0.
```

#### JavaScript Solution

```
/**
 * @param {number[]} prices
 * @return {number}
 */
var maxProfit = function(prices) {
    let min = prices[0], maxProfit = 0, profit;
    for (let i = 1; i < prices.length; i++) {
    	profit = prices[i] - min;
    	maxProfit = profit > maxProfit ? profit : maxProfit;
    	min = prices[i] < min ? prices[i] : min;
    }
    return maxProfit;
};

maxProfit([7, 1, 5, 3, 6, 4]);
```

#### Java Solution

```
public class Solution {
    public int maxProfit(int[] prices) {
        if (prices.length == 0) return 0;
        int min = prices[0], maxProfit = 0, profit;
        for (int i = 1; i < prices.length; i++) {
            profit = prices[i] - min;
            maxProfit = maxProfit > profit ? maxProfit : profit;
            min = prices[i] < min ? prices[i] : min;
        }
        return maxProfit;
    }
}
```

### [122. Best Time to Buy and Sell Stock II](https://leetcode.com/problems/best-time-to-buy-and-sell-stock-ii/)

Say you have an array for which the ith element is the price of a given stock on day i.

Design an algorithm to find the maximum profit. You may complete as many transactions as you like (ie, buy one and sell one share of the stock multiple times). However, you may not engage in multiple transactions at the same time (ie, you must sell the stock before you buy again).

其实这个问题的解法就是把每天赚的钱都累计起来即可。如果当天没赚钱，则当天的累计为0（解法见 Java Solution）；

#### JavaScript Solution

```
/**
 * @param {number[]} prices
 * @return {number}
 */
var maxProfit = function(prices) {
    let min = prices[0], profit = 0, tmp = 0;
    for (let i = 1; i < prices.length; i++) {
    	if (prices[i] > prices[i-1]) {
    		tmp = prices[i] - min;
    	} else {
    		profit += tmp;
    		tmp = 0;
    		min = prices[i];
    	}
    }
    profit += tmp;
    return profit;
};

maxProfit([7, 1, 5, 3, 6, 4]);
```

#### Java Solution

```
public class Solution {
    public int maxProfit(int[] prices) {
        if(prices.length == 0) return 0;
        int profit = 0;
        for (int i = 1; i < prices.length; i++) {
            if (prices[i] > prices[i-1])
                profit += (prices[i] - prices[i-1]);
        }
        return profit;
    }
}
```
### [167. Two Sum II - Input array is sorted](https://leetcode.com/problems/two-sum-ii-input-array-is-sorted/)

Given an array of integers that is already **sorted in ascending order**, find two numbers such that they add up to a specific target number.

The function twoSum should return indices of the two numbers such that they add up to the target, where index1 must be less than index2. Please note that your returned answers (both index1 and index2) are not zero-based.

You may assume that each input would have exactly one solution and you may not use the same element twice.

**Input**: numbers={2, 7, 11, 15}, target=9
**Output**: index1=1, index2=2

我的解法是将 numbers 的数值作为 key，将数值对应数组的下标作为 value，放在 Hashmap 中

#### JavaScript Solution

```
var map = new Map();
    for(let i = 0; i < numbers.length; i++) {
    	if(map.has(target-numbers[i])) {
    		return [map.get(target-numbers[i]), i+1];
    	} else {
    		map.set(numbers[i], i+1)
    	}
    }
```

#### Java Solution

```
public class Solution {
    public int[] twoSum(int[] numbers, int target) {
        Map<Integer, Integer> map = new HashMap<Integer, Integer>();
        for(int i = 0; i < numbers.length; i++) {
            if(map.containsKey(target-numbers[i])) {
                return new int[]{map.get(target-numbers[i]), i+1};
            } else {
                map.put(numbers[i], i+1);
            }
        }
        return null;
    }
}
```

#### Best Solution

最优解没有使用 HashMap。而是利用了数组是一个有序数组的条件，从数组前后使用指针分别遍历数组，找到相加等于 target 的两个数。

```
/**
 * @param {number[]} numbers
 * @param {number} target
 * @return {number[]}
 */
var twoSum = function(numbers, target) {
    let start = 0, end = numbers.length-1;
    while(start < end) {
    	sum = numbers[start] + numbers[end];
    	if(sum == target) {
    		return [start+1, end+1];
    	} else if(sum < target) {
    		start ++;
    	} else {
    		end --;
    	}
    }
};
```

### [169. Majority Element](https://leetcode.com/problems/majority-element/)

Given an array of size n, find the majority element. The majority element is the element that appears **more than `[ n/2 ]` times**.

You may assume that the array is non-empty and the majority element always exist in the array.

解决这个问题的想法，就是利用大多数的值大于一半这个条件。

#### JavaScript Solution

```
/**
 * @param {number[]} nums
 * @return {number}
 */
var majorityElement = function(nums) {
    let count = 1, major = nums[0];
    for(let i = 1; i < nums.length; i++) {
    	if(count === 0) {
    		count++;
    		major = nums[i];
    	} else if(major === nums[i]) {
    		count++;
    	} else {
    		count--;
    	}
    }
    return major;
};

majorityElement([3, 2, 3])
```

#### Java Solution

```
public class Solution {
    public int majorityElement(int[] nums) {
        int count = 1, major = nums[0];
        for (int i = 1; i < nums.length; i++) {
            if(count == 0) {
                count++;
                major = nums[i];
            } else if(major == nums[i]) {
                count++;
            } else count--;
        }
        return major;
    }
}
```

### [189. Rotate Array](https://leetcode.com/problems/rotate-array/)

Rotate an array of n elements to the right by k steps.

For example, with n = 7 and k = 3, the array `[1,2,3,4,5,6,7]` is rotated to `[5,6,7,1,2,3,4]`.

**Note:**
Try to come up as many solutions as you can, there are at least 3 different ways to solve this problem.

**Hint:**
Could you do it in-place with O(1) extra space?

**如果是每次挪一个数，一共挪 k 次，会超时**

#### JavaScript Solutio

这个方法是我的方法，想法就是 nums[index + k] = nums[index]，然后加上边界条件得到的。

```
/**
 * @param {number[]} nums
 * @param {number} k
 * @return {void} Do not return anything, modify nums in-place instead.
 */
var rotate = function(nums, k) {
	k = k % nums.length;
	let tmp1 = nums[0], tmp2 = nums[0], step = nums.length;
	let start = 0, s = 0;
	while (step > 0) {	
		s += k;
		step--;
		if(s >= nums.length) s %= nums.length;
		tmp1 = nums[s];
		nums[s] = tmp2;
		tmp2 = tmp1;
		if(s === start) {
			start++;
			s = start;
			tmp2 = nums[s];
		}
	}
};

rotate([1, 2, 3, 4, 5, 6], 3)
```

#### Java Solution

这个是最优解

```
public void rotate(int[] nums, int k) {
        k %= nums.length;
        reverse(nums, 0, nums.length - 1);
        System.out.println(Arrays.toString(nums));
        reverse(nums, 0, k - 1);
        System.out.println(Arrays.toString(nums));
        reverse(nums, k, nums.length - 1);
        System.out.println(Arrays.toString(nums));
    }

    public void reverse(int[] nums, int start, int end) {
        while (start < end) {
            int temp = nums[start];
            nums[start] = nums[end];
            nums[end] = temp;
            start++;
            end--;
        }
    }
```

### [217. Contains Duplicate](https://leetcode.com/problems/contains-duplicate/)

Given an array of integers, find if the array contains any duplicates. Your function should return true if any value appears at least twice in the array, and it should return false if every element is distinct.

#### JavaScript Solution

这个解法是时间复杂度为 O(n*n)，空间复杂度为 O(1)的解法

```
/**
 * @param {number[]} nums
 * @return {boolean}
 */
var containsDuplicate = function(nums) {
    for (let i = 0; i < nums.length - 1; i++) {
    	for(let j = i + 1; j < nums.length; j++) {
    		if (nums[i] == nums[j]) return true;
    	}
    }
    return false;
};

containsDuplicate([1, 2, 3, 4, 1])
```

这个方法使用了 Set 类，时间复杂度为 O(n)，空间复杂度为 O(n)

```
/**
 * @param {number[]} nums
 * @return {boolean}
 */
var containsDuplicate = function(nums) {
    let len = nums.length;
    let set = new Set(nums);
    if(set.size < len) return ture;
    return false;
};

containsDuplicate([1, 2, 3, 4, 1])
```

### [219. Contains Duplicate II](https://leetcode.com/problems/contains-duplicate-ii/)

Given an array of integers and an integer k, find out whether there are two distinct indices i and j in the array such that **nums[i] = nums[j]** and the **absolute** difference between i and j is at most k.

#### JavaScript Solution

这个方法的时间复杂度为 O(n*k)， 空间复杂度为 O(1)

```
/**
 * @param {number[]} nums
 * @param {number} k
 * @return {boolean}
 */
var containsNearbyDuplicate = function(nums, k) {
    for (let i = 0; i < nums.length-1; i++) {
    	for (let j = i+1; j <= Math.min(nums.length-1, i + k); j++ ){
    		if(nums[i] == nums[j] )
    			return true;
    	}
    }
    return false;
};


containsNearbyDuplicate([1, 2], 2)
```

#### Java Solution

这种方法的时间复杂度为 O(n)，空间复杂度为 O(n)

```
public boolean containsNearbyDuplicate(int[] nums, int k) {
        Map<Integer, Integer> map = new HashMap<Integer, Integer>();
        for( int i = 0; i < nums.length; i++) {
            if (map.containsKey(nums[i]) && (i - map.get(nums[i])) <= k)
                return true;
            map.put(nums[i], i);
        }
        return false;
    }
```

#### Best Solution

这个方法中利用了 Set 类中 add 方法的返回值，如果这个数在集合中出现，则返回 true，否则返回 false。

```
public boolean containsNearbyDuplicate(int[] nums, int k) {
        Set<Integer> set = new HashSet<Integer>();
        for(int i = 0; i < nums.length; i++){
            if(i > k) set.remove(nums[i-k-1]);
            if(!set.add(nums[i])) return true;
        }
        return false;
 }
```

### [268. Missing Number](https://leetcode.com/problems/missing-number/)

Given an array containing n distinct numbers taken from 0, 1, 2, ..., n, find the one that is missing from the array.

For example,
Given nums = `[0, 1, 3]` return `2`.

**Note:**

Your algorithm should run in linear runtime complexity. Could you implement it using only constant extra space complexity?

要使用 O(n) 的时间和 O(1) 的复杂度。

O(n) 的时间说明不能对原数组进行排序。

#### JavaScript Solution

我的想法是，如果 nums[i] = 1，则将 nums[1] 上面的数设置成该值得绝对值的负数。最后遍历一遍，找出不是负数的值，对应的 i 没有出现。

```
/**
 * @param {number[]} nums
 * @return {number}
 */
var missingNumber = function(nums) {
	let hasZero = false;
	nums[nums.length] = nums.length;
	for (let i = 0; i < nums.length; i++) {
		if(nums[i] === 0) {
			hasZero = true;
		}
		let tmp = Math.abs(nums[i]);
		if(nums[tmp] === 0) {
			hasZero = true;
			nums[tmp] = -(nums.length-1);
		}
		nums[tmp] = -Math.abs(nums[tmp]);
	}
	if(hasZero) {
		for (let i = 1; i < nums.length; i++) {
			if(nums[i] >= 0) {
				return i;
			}
		}
		return nums.length-1;
	} else {
		return 0;
	}
};

missingNumber([0, 1, 3])
```

#### Best Solution

通过 和 减去所有的数，得到的差值就是没出现的值。

```
public int missingNumber(int[] nums) { //sum
    int len = nums.length;
    int sum = (0+len)*(len+1)/2;
    for(int i=0; i<len; i++)
        sum-=nums[i];
    return sum;
}
```

还有一种使用亦或的方法：

```
public int missingNumber(int[] nums) { //xor
    int res = nums.length;
    for(int i=0; i<nums.length; i++){
        res ^= i;
        res ^= nums[i];
    }
    return res;
}
```

### [283. Move Zeroes](https://leetcode.com/problems/move-zeroes/)

Given an array `nums`, write a function to move all `0`'s to the end of it while maintaining the relative order of the non-zero elements.

For example, given `nums = [0, 1, 0, 3, 12`], after calling your function, `nums` should be `[1, 3, 12, 0, 0]`.

**Note:**

You must do this **in-place** without making a copy of the array.
Minimize the total number of operations.

#### JavaScript Solution

我的想法是从后向前判断是否为0，如果是0，则往最后移动。

```
/**
 * @param {number[]} nums
 * @return {void} Do not return anything, modify nums in-place instead.
 */
var moveZeroes = function(nums) {
	let tmp;
    for (var i = nums.length - 1; i >= 0; i--) {
    	if (nums[i] === 0) {
    		let index = i + 1;
    		while ((index < nums.length) && (nums[index] !== 0)) {
    			nums[index - 1] = nums[index];
    			nums[index] = 0;
    			index++;
    		}
    	}
    }
};

moveZeroes([0, 1, 0, 3, 12]);
```

#### Best Solution

```
// Shift non-zero values as far forward as possible
// Fill remaining space with zeros

public void moveZeroes(int[] nums) {
    if (nums == null || nums.length == 0) return;        

    int insertPos = 0;
    for (int num: nums) {
        if (num != 0) nums[insertPos++] = num;
    }        

    while (insertPos < nums.length) {
        nums[insertPos++] = 0;
    }
}
```

### [Third Maximum Number](https://leetcode.com/problems/third-maximum-number/)

Given a **non-empty array** of integers, return the **third** maximum number in this array. If it does not exist, return the maximum number. The time complexity must be in O(n).

**Example 1:**

```
Input: [3, 2, 1]

Output: 1

Explanation: The third maximum is 1.
```

**Example 2:**

```
Input: [1, 2]

Output: 2

Explanation: The third maximum does not exist, so the maximum (2) is returned instead.
```

**Example 3:**

```
Input: [2, 2, 3, 1]

Output: 1

Explanation: Note that the third maximum here means the third maximum distinct number.
Both numbers with value 2 are both considered as second maximum.
```

#### JavaScript Solution

首先把前面的数放在 set 中，直到有三个不重复的数以后，就可以进行排序

```
/**
 * @param {number[]} nums
 * @return {number}
 */
var thirdMax = function(nums) {
	let set = new Set(), arr = [], hasThree = false;
    for (var i = nums.length - 1; i >= 0; i--) {
    	if(!hasThree && (set.size != 3)) {
    		set.add(nums[i]);
    	}
    	if(!hasThree && (set.size === 3)) {
    		set.forEach( function(element) {
    			arr.push(element);
    		});
    		arr.sort();
    		hasThree = true;
    	} else if(hasThree) {
    		if(nums[i] < arr[0]) {
	    	} else if(nums[i] > arr[2]) {
	    		arr[0] = arr[1];
	    		arr[1] = arr[2];
	    		arr[2] =  nums[i];
	    	} else if((nums[i] < arr[2]) && (nums[i] > arr[1])) {
	    		arr[0] = arr[1];
	    		arr[1] = nums[i];
	    	} else if(nums[i] < arr[1]){
	    		arr[0] = nums[i];
	    	}
    	}
    }

    if(hasThree) return arr[0];
    else {
    	set.forEach( function(element) {
			arr.push(element);
		});
		arr.sort();
		return arr[arr.length-1];
    }		
};

thirdMax([5,2,4,1,3,6,0]);
```

#### Best Solution

```
public int thirdMax(int[] nums) {
        Integer max1 = null;
        Integer max2 = null;
        Integer max3 = null;
        for (Integer n : nums) {
            if (n.equals(max1) || n.equals(max2) || n.equals(max3)) continue;
            if (max1 == null || n > max1) {
                max3 = max2;
                max2 = max1;
                max1 = n;
            } else if (max2 == null || n > max2) {
                max3 = max2;
                max2 = n;
            } else if (max3 == null || n > max3) {
                max3 = n;
            }
        }
        return max3 == null ? max1 : max3;
    }
```

### [448. Find All Numbers Disappeared in an Array](https://leetcode.com/problems/find-all-numbers-disappeared-in-an-array/)

Given an array of integers where 1 ≤ a[i] ≤ n (n = size of array), some elements appear twice and others appear once.

Find all the elements of [1, n] inclusive that do not appear in this array.

Could you do it without extra space and in O(n) runtime? You may assume the returned list does not count as extra space.

**Example:**

```
Input:
[4,3,2,7,8,2,3,1]

Output:
[5,6]
```

#### JavaScript Solution

由于不能使用额外的空间，所以考虑对原数组进行修改，如果 i 出现，则 nums[i] 的值变成负相反数。

```
/**
 * @param {number[]} nums
 * @return {number[]}
 */
var findDisappearedNumbers = function(nums) {
    for (let i = nums.length - 1; i >= 0; i--) {
    	nums[(Math.abs(nums[i])-1)] = -Math.abs(nums[(Math.abs(nums[i])-1)]);
    }
    let arr = [];
    for (let i = 0; i < nums.length; i++) {
    	if(nums[i] > 0) arr.push(i+1);
    }
    return arr;
};
findDisappearedNumbers([1, 3, 2, 4, 2, 3, 5, 6])
```

#### Best Solution

想法和我一样，不过他没有使用 Math.abs，使得计算更加简单。

```
public List<Integer> findDisappearedNumbers(int[] nums) {
        List<Integer> ret = new ArrayList<Integer>();
        
        for(int i = 0; i < nums.length; i++) {
            int val = Math.abs(nums[i]) - 1;
            if(nums[val] > 0) {
                nums[val] = -nums[val];
            }
        }
        
        for(int i = 0; i < nums.length; i++) {
            if(nums[i] > 0) {
                ret.add(i+1);
            }
        }
        return ret;
    }
```

### [485. Max Consecutive Ones](https://leetcode.com/problems/max-consecutive-ones/)

Given a binary array, find the maximum number of consecutive 1s in this array.

**Example 1:**

```
Input: [1,1,0,1,1,1]
Output: 3
Explanation: The first two digits or the last three digits are consecutive 1s.
    The maximum number of consecutive 1s is 3.
```

**Note:**

The input array will only contain 0 and 1.

The length of input array is a positive integer and will not exceed 10,000

#### JavaScript Solution

```
/**
 * @param {number[]} nums
 * @return {number}
 */
var findMaxConsecutiveOnes = function(nums) {
	let len = maxLen = 0;
    for (var i = nums.length - 1; i >= 0; i--) {
    	if (nums[i] === 1) 
    		len++;
    	else {
    		maxLen = Math.max(len, maxLen);
    		len = 0;
    	}
    }
    return Math.max(len, maxLen);
};

findMaxConsecutiveOnes([1,1,0,1])
```



