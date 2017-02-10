---
title: LeetCode String题集
date: 2017-02-08 23:13:28
categories: coding
tags:
  - LeetCode
  - String
  - Java
  - JavaScript
---

下面所记录的是我在刷 [LeetCode](https://leetcode.com) Array 相关的题目自己的解法。

### [13. Roman to Integer](https://leetcode.com/problems/roman-to-integer/)

Given a roman numeral, convert it to an integer.

Input is guaranteed to be within the range from 1 to 3999.

罗马数字是阿拉伯数字传入之前使用的一种数码。罗马数字采用七个罗马字母作数字、即Ⅰ（1）、X（10）、C（100）、M（1000）、V（5）、L（50）、D（500）。记数的方法：

* 相同的数字连写，所表示的数等于这些数字相加得到的数，如 Ⅲ=3；
* 小的数字在大的数字的右边，所表示的数等于这些数字相加得到的数，如 Ⅷ=8、Ⅻ=12；
* 小的数字（限于 Ⅰ、X 和 C）在大的数字的左边，所表示的数等于大数减小数得到的数，如 Ⅳ=4、Ⅸ=9

#### JavaScript Solution

我的想法就是，首先把所有的数所代表的值都加起来，然后减去小叔子在大数字左边的情况。

在 JavaScript 中，获取第 i 个位置上的 char, 可以使用 `s[i]`，也可以使用 `s.charAt(1)`；
在 JavaScript 中，判断 String 中是否存在某个String `s2` ，有 `s1.indexOf(s2)`，可以通过返回值是否为 -1 判断，也可以使用 
`s1.includes(s2)` 判断 `true/false`

```
/**
 * @param {string} s
 * @return {number}
 */
var romanToInt = function(s) {
	let sum = 0;
	for (let i = s.length - 1; i >= 0; i--) {
	  	if(s[i] === "I") sum += 1;
	  	if(s[i] === "V") sum += 5;
	  	if(s[i] === "X") sum += 10;
	  	if(s[i] === "L") sum += 50;
	  	if(s[i] === "C") sum += 100;
	  	if(s[i] === "D") sum += 500;
	  	if(s[i] === "M") sum += 1000;
	}  
	if(s.includes("IV")) sum -= 2;
	if(s.includes("IX")) sum -= 2;
	if(s.includes("XL")) sum -= 20;
	if(s.includes("XC")) sum -= 20;
	if(s.includes("CD")) sum -= 200;
	if(s.includes("CM")) sum -= 200;
	return sum;
};

romanToInt("DCXXI");
```

#### Java Solution

在 Java 中，获取第 i 个位置上的 **`char`**，一般使用 `s.charAt(i)`
在 Java 中，判断一个 String `s2` 是否出现过，可以使用 `s1.indexOf(s2)`，判断值是否为 -1，也可以使用 `s1.contains(s2)`，判断 `true/false`

```
public int romanToInt(String s) {
        int sum = 0;
        for (int i = 0; i < s.length(); i++) {
            if (s.charAt(i) == 'I') sum += 1;
            if (s.charAt(i) == 'V') sum += 5;
            if (s.charAt(i) == 'X') sum += 10;
            if (s.charAt(i) == 'L') sum += 50;
            if (s.charAt(i) == 'C') sum += 100;
            if (s.charAt(i) == 'D') sum += 500;
            if (s.charAt(i) == 'M') sum += 1000;
        }
        if (s.contains("IV")) sum -= 2;
        if (s.contains("IX")) sum -= 2;
        if (s.contains("XL")) sum -= 20;
        if (s.contains("XC")) sum -= 20;
        if (s.contains("CD")) sum -= 200;
        if (s.contains("CM")) sum -= 200;
        return sum;
    }
```

### [14. Longest Common Prefix](https://leetcode.com/problems/longest-common-prefix/)

Write a function to find the longest common prefix string amongst an array of strings.

我的想法就是首先比较前两个，找到最长 prefix，然后用最长 prefix 和数组第三个比较。。。

#### JavaScript Solution

在 JavaScript 中，获取一个字符串的子串，可以使用以下的方法：

* `str.slice(beginSlice[, endSlice])`：获得 `beginSlice` 到 `endSlice` 之间的字符串
* `str.substr(start [, length])`：根据 start 和 len 获得字符串
* `str.substring(indexStart[, indexEnd])`：根据 indexStart 和 IndexEnd 获取字符串

其中三者的区别为：

![](http://ojt6zsxg2.bkt.clouddn.com/7330d51d28d7f72a9ad1ee62f34325ed.png)

例如：

```
var strValue = "javascript programing";
alert(strValue.slice(3));           //"ascript programing"
alert(strValue.substring(3));       //"ascript programing"
alert(strValue.substr(3));          //"ascript programing"
alert(strValue.slice(3,13));        //"ascript pr"
alert(strValue.substring(3,13));    //"ascript pr"
alert(strValue.substr(3,13));       //"ascript progr"
```

![](http://ojt6zsxg2.bkt.clouddn.com/d700e065a78588f2f1eae0b8b5555f18.png)

例如：

```
var strValue = "javascript programing";
alert(strValue.slice(-3));        => alert(strValue.slice(18));       //"ing"
alert(strValue.substring(-3));    => alert(strValue.substring(0));    //"javascript programing"
alert(strValue.substr(-3));       => alert(strValue.substr(18));      //"ing"
alert(strValue.slice(3,-13));     => alert(strValue.slice(3,8));      //"ascri" 
alert(strValue.substring(3,-13)); => alert(strValue.substring(0,3));  //"jav"
alert(strValue.substr(3,-13));    => alert(strValue.substr(3,0));     //""
```

```
/**
 * @param {string[]} strs
 * @return {string}
 */
var longestCommonPrefix = function(strs) {
	if (strs.length === 0) return "";
    let prefix = strs[0];
    for (let i = 1; i < strs.length; i++) {
    	for (let j = 0; j < prefix.length; j++) {
    		if(prefix[j] != strs[i][j]) {
    			prefix = prefix.slice(0, j);
       			break;
    		}
    	}
    }
    return prefix;
};

longestCommonPrefix(["asdasd", "asd12", "as234"]);
```

#### Java Solution

在 JavaScript 中，获取一个字符串的子串，可以使用以下的方法：

```
substring(int beginIndex)
substring(int beginIndex, int endIndex)
```

```
public class Solution {
    public String longestCommonPrefix(String[] strs) {
        if (strs.length == 0) return "";
        String prefix = strs[0];
        for (int i = 1; i < strs.length; i++) {
            int len = Math.min(prefix.length(), strs[i].length());
            for (int j = 0; j < len; j++) {
                if (prefix.charAt(j) != strs[i].charAt(j)) {
                    prefix = prefix.substring(0, j);
                    break;
                }
            }
            if(prefix.length() > len) {
                prefix = prefix.substring(0, len);
            }
        }
        return prefix;
    }
}
```

#### Best Solution

最优解使用的 `String.contains` 方法来进行 prefix 的删除，虽然想法一致，但是做法简单得多。

```
public String longestCommonPrefix(String[] strs) {
    if(strs == null || strs.length == 0)    return "";
    String pre = strs[0];
    int i = 1;
    while(i < strs.length){
        while(strs[i].indexOf(pre) != 0)
            pre = pre.substring(0,pre.length()-1);
        i++;
    }
    return pre;
}
```

### [20. Valid Parentheses](https://leetcode.com/problems/valid-parentheses/)

Given a string containing just the characters `'('`, `')'`, `'{'`, `'}'`, `'['` and `']'`, determine if the input string is valid.

The brackets must close in the correct order, `"()"` and `"()[]{}"` are all valid but `"(]"` and `"([)]"` are not.

我的解法是使用栈(Stack)来进行入栈和出栈。

#### JavaScript Solution

JavaScript 中栈是通过 Array 中 `push`、`pop`来实现的

```
/**
 * @param {string} s
 * @return {boolean}
 */
var isValid = function(s) {
    let arr = [];
    for (let i = 0; i < s.length; i++) {
    	let tmp = s[i];
    	if(tmp === "(" || tmp === "[" || tmp === "{") {
    		arr.push(tmp);
    	} else {
    		if(tmp === ")" && arr[arr.length-1] === "(") {
    			arr.pop();
    			continue;
    		}
    		if(tmp === "]" && arr[arr.length-1] === "[") {
    			arr.pop();
    			continue;
    		}
    		if(tmp === "}" && arr[arr.length-1] === "{") {
    			arr.pop();
    			continue;
    		}
    		return false;
    	}
    }
    if (arr.length != 0) 
    	return false;
    return true;
};

isValid("[]{}()")
```


#### Java Solution

Java 中的栈是通过 `Stack` 类实现

```
public boolean isValid(String s) {
        Stack<String> stack = new Stack<String>();
        for (int i = 0; i < s.length(); i++) {
            String tmp = String.valueOf(s.charAt(i));
            if (tmp.equals("(") || tmp.equals("[") || tmp.equals("{"))
                stack.push(tmp);
            else if (stack.size() != 0){
                if (stack.peek().equals("(") && tmp.equals(")")) {
                    stack.pop();
                    continue;
                }
                if (stack.peek().equals("[") && tmp.equals("]")) {
                    stack.pop();
                    continue;
                }
                if (stack.peek().equals("{") && tmp.equals("}")) {
                    stack.pop();
                    continue;
                }
                return false;
            } else {
                return false;
            }
        }
        if (stack.size() == 0)
            return true;
        return false;
    }
```

#### Best Solution

想法基本相同，做法在有些地方有优化

```
public class Solution {
    public boolean isValid(String s) {
        Stack<Character> stack = new Stack<Character>();
        // Iterate through string until empty
        for(int i = 0; i<s.length(); i++) {
            // Push any open parentheses onto stack
            if(s.charAt(i) == '(' || s.charAt(i) == '[' || s.charAt(i) == '{')
                stack.push(s.charAt(i));
            // Check stack for corresponding closing parentheses, false if not valid
            else if(s.charAt(i) == ')' && !stack.empty() && stack.peek() == '(')
                stack.pop();
            else if(s.charAt(i) == ']' && !stack.empty() && stack.peek() == '[')
                stack.pop();
            else if(s.charAt(i) == '}' && !stack.empty() && stack.peek() == '{')
                stack.pop();
            else
                return false;
        }
        // return true if no open parentheses left in stack
        return stack.empty();
    }
}
```

### [28. Implement strStr()](https://leetcode.com/problems/implement-strstr/)

Implement strStr().

Returns the index of the first occurrence of needle in haystack, or -1 if needle is not part of haystack.

虽然这道题直接使用 indexOf 就可以 accept，但是作者的意图应该是让我们实现一个 indexOf 方法。

#### JavaScript Solution

这个方法是 Best Solution，想法也是一个一个比较，但是在 for 循环中没有设置溢出条件，而是在 if 语句中设置了溢出条件，因为 for 循环中的溢出条件基本用不上。

```
/**
 * @param {string} haystack
 * @param {string} needle
 * @return {number}
 */
var strStr = function(haystack, needle) {
    for (let i = 0; ; i++) {
    	for (let j = 0; ; j++) {
    		if (j === needle.length) return i;
    		if ((i+j) === haystack.length) return -1;
    		if (haystack[i+j] !== needle[j]) break;
    	}
    }
};

strStr("sdasd", "asd");
```

#### Java Solution

```
public int strStr(String haystack, String needle) {
        for (int i = 0; ; i++) {
            for (int j = 0; ; j++) {
                if (j == needle.length()) return i;
                if ((i + j) == haystack.length()) return -1;
                if (haystack.charAt(i+j) != needle.charAt(j)) break;
            }
        }
    }
```

### [38. Count and Say](https://leetcode.com/problems/count-and-say/)

The count-and-say sequence is the sequence of integers beginning as follows:
`1, 11, 21, 1211, 111221, ...`

`1` is read off as `"one 1"` or `11`.
`11` is read off as `"two 1s"` or `21`.
`21` is read off as `"one 2`, then `"one 1"` or `1211`.
Given an integer n, generate the nth sequence.

Note: The sequence of integers will be represented as a string.

#### JavaScript Solution

```
/**
 * @param {number} n
 * @return {string}
 */
var countAndSay = function(n) {
	let seq = "1";
    for (let i = 1; i < n; i++) {
    	let tmp = "", num = 1, value = seq[0];
    	for (let j = 1; j < seq.length; j++) {
    		if (seq[j] != value) {
    			tmp += num;
    			tmp += value;
    			num = 1;
    			value = seq[j];
    			continue;
    		}
    		num++;
    	}
    	tmp += num;
    	tmp += value;
		seq = tmp;
    }
    return seq;
};

countAndSay(4)
```

#### Best Solution

如果是针对经常进行变动的 String，可以使用 StringBuilder 类

StringBuilder 类中常用的方法有：

* append
* charAt
* delete(int start, int end)
* deleteCharAt(int index)
* insert(int offset, String str)
* replace(int start, int end, String str)
* substring

String 中常用的方法有：

* charAt
* compareTo
* concat
* contains
* indexOf
* join
* match
* replace
* split
* valueOf
* substring
* toLowerCase/toUpperCase

```
public class Solution {
    public String countAndSay(int n) {
	    	StringBuilder curr=new StringBuilder("1");
	    	StringBuilder prev;
	    	int count;
	    	char say;
	        for (int i=1;i<n;i++){
	        	prev=curr;
	 	        curr=new StringBuilder();       
	 	        count=1;
	 	        say=prev.charAt(0);
	 	        
	 	        for (int j=1,len=prev.length();j<len;j++){
	 	        	if (prev.charAt(j)!=say){
	 	        		curr.append(count).append(say);
	 	        		count=1;
	 	        		say=prev.charAt(j);
	 	        	}
	 	        	else count++;
	 	        }
	 	        curr.append(count).append(say);
	        }	       	        
	        return curr.toString();
        
    }
}
```

### [58. Length of Last Word](https://leetcode.com/problems/length-of-last-word/)


Given a string s consists of upper/lower-case alphabets and empty space characters `' '`, return the length of last word in the string.

If the last word does not exist, return 0.

**Note:** A word is defined as a character sequence consists of non-space characters only.

For example,

Given s = `"Hello World"`,
return `5`.

#### JavaScript Solution

```
/**
 * @param {string} s
 * @return {number}
 */
var lengthOfLastWord = function(s) {
    let len = 0, isSpace = false;
    for (let i = 0; i < s.length; i++) {
    	if (isSpace && s[i] != " ") {
    		len = 0;
    		isSpace = false;
    	}
    	if (s[i] === " ") isSpace = true;
    	else len ++;
    }
    return len;
};

lengthOfLastWord("Hello World");
```

#### Best Solution

Best Solution 很简单的方法，就是先对 s 进行 trim，将前后的空格都去掉。简化了逻辑中对最后一位是 `" "` 的判断。

```
public int lengthOfLastWord(String s) {
        s = s.trim();
        int len = 0;
        for (int i = 0; i < s.length(); i++) {
            if (s.charAt(i) == ' ') len = 0;
            else len ++;
        }
        return len;
    }
```

### [67. Add Binary](https://leetcode.com/problems/add-binary/)

Given two binary strings, return their sum (also a binary string).

For example,
a = `"11"`
b = `"1"`
Return `"100"`.

#### JavaScript Solution

解决这个问题，有几个要点。第一个要点是 `a[i] - "0"` 直接获得数字值。第二个要点是，为了不用判断 a 和 b 谁的长度厂，可以在 `while` 中使用 `if` 分别处理，尽量避免 `a[i]` 和 `b[j]` 的直接联系。第三点是使用 Array 存储，再使用 `join` 合成 string。

JavaScript string 没有翻转的函数，但是可以使用以下的方法实现该功能：

```
s.split("").reverse().join();
```

```
/**
 * @param {string} a
 * @param {string} b
 * @return {string}
 */
var addBinary = function(a, b) {
    let i = a.length-1, j = b.length-1, carry = 0;
    let result = [];
    while (i >=0 || j >= 0) {
    	let sum = carry;
    	if (i >= 0) sum += (a[i--] - "0");
    	if (j >= 0) sum += (b[j--] - "0");
    	result.unshift(sum % 2);
    	carry = Math.floor(sum / 2);
    }
    if (carry !== 0) result.unshift(carry);
    return result.join("");
};

addBinary("1", "1101");
```

#### Java Solution

```
public class Solution {
    public String addBinary(String a, String b) {
        StringBuilder sb = new StringBuilder();
        int i = a.length() - 1, j = b.length() -1, carry = 0;
        while (i >= 0 || j >= 0) {
            int sum = carry;
            if (j >= 0) sum += b.charAt(j--) - '0';
            if (i >= 0) sum += a.charAt(i--) - '0';
            sb.append(sum % 2);
            carry = sum / 2;
        }
        if (carry != 0) sb.append(carry);
        return sb.reverse().toString();
    }
}
```









