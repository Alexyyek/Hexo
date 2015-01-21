title: Count And Say
date: 2014-12-06 17:31:42
categories: Java
tags: [dfs, leetcode]
description:  The count-and-say sequence is the sequence of integers beginning as follows 1, 11, 21, 1211, 111221, ... , Given an integer n, generate the nth sequence.
---

##Count and Say

Question : The count-and-say sequence is the sequence of integers beginning as follows:
`1, 11, 21, 1211, 111221, ...`

`1` is read off as `one 1` or `11`.
`11` is read off as `two 1s` or `21`.
`21` is read off as `one 2, then one 1` or `1211`.
Given an integer `n`, generate the nth sequence.
<!--more--> 

###Best solution
```java
	public static String countAndSay(int n) {
		if (n <= 0) {
			return null;
		}

		String first = "1";
		int count = 1;
		for (int i = 1; i < n; i++) {
			StringBuilder sb = new StringBuilder();
			for (int j = 0; j < first.length(); j++) {
				if (j < first.length() - 1
						&& (first.charAt(j) == first.charAt(j + 1))) {
					count++;
				} else {
					sb.append(count + "" + first.charAt(j));
					count = 1;
				}
			}
			first = sb.toString();
		}
		return first;
	}
```
**Analyze** : 题目非常简单，序列迭代规则已经列出，求第n次迭代的值。基本方法判断相邻两个字符是否一样，算出每一次的序列值，迭代到n次为止。其实也可以用递归的方法来做，给出第n-1次的规则，求第n次的值。
<br/>
***
比较有意思的是用DFS的思想来解决此题，代码如下：
###DFS
```java
    private String s = "1";
    public String countSay(int n) {
        if(n < 1) return "";
        for (int i = 1; i < n; i++)
            dfs(s.length());
        return s;
    }

    private void dfs(int length) {
        if (length == 0) return;
        int count = 1, i = 0;
        while (i < length - 1 && s.charAt(i) == s.charAt(1 + i)) {
            count++;
            i++;
        }
        s = s.substring(i + 1) + count + s.charAt(i);
        dfs(length - count);
    }
```
**Analyze** : 深度优先算法的思路也很简单，比如第4次迭代结果为1211，现在进行第5次迭代
* 树`1-2-1-1`根节点为`1`，出现一次所以值变成`11`,树变成`2-1-1`，所以整体值为`21111`；
* 第二次运算`2`出现一次，值为`12`，树还剩`1-1`，所以整体值为`111112`，注意这里前面11是待统计的树`1-1`,中间的11是第一次的运算值，最后的12是第二次的运算值；
* 由于树`1-1`中`1`出现了两次，所以值为`21`，s = s.substring(i + 1) + count + s.charAt(i) = 111221

###More
这道题在leetcode上的表述略有不清，我第一次就把此题误解为给定任意整数n，去求此整数的第n次迭代值
比如5->15->1115->3115->132115->1113122115，结果发现高估此题了...
既然都写了就列在下面吧
```java
	public static String countAndSay(int n) {
		if (n < 0) {
			return null;
		}
		String word = String.valueOf(n);
		while (n > 0) {
			int count = 1;
			StringBuffer sBuffer = new StringBuffer();
			for (int i = 0; i < word.length(); i++) {
				if (i < word.length() - 1
						&& word.charAt(i) == word.charAt(i + 1)) {
					count++;
				} else {
					sBuffer.append(count).append(word.charAt(i));
					count = 1;
				}
			}
			word = sBuffer.toString();
			n--;
		}
		return word;
	}
```