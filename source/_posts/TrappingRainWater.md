title: Trapping Rain Water
date: 2014-12-04 15:17:18
categories: Java
tags: [Arrays, leetcode]
description : Given n non-negative integers representing an elevation map where the width of each bar is 1, compute how much water it is able to trap after raining.
---

##Trapping Rain Water
Question : Given n non-negative integers representing an elevation map where the width of each bar is 1, compute how much water it is able to trap after raining.
<!--more-->  
For example:
Given [0,1,0,2,1,0,1,3,2,1,2,1], return 6.

![](http://leetcode.com/wp-content/uploads/2012/08/rainwatertrap.png)

###Best Solution
```java
public class Solution {
    public int trap(int[] A) {
		int a = 0;
		int b = A.length - 1;
		int max = 0;
		int leftmax = 0;
		int rightmax = 0;
		while (a <= b) {
			leftmax = Math.max(leftmax, A[a]);
			rightmax = Math.max(rightmax, A[b]);
			if (leftmax < rightmax) {
				max += (leftmax - A[a]); 
				a++;
			} else {
				max += (rightmax - A[b]);
				b--;
			}
		}
		return max;
    }
}
```
**Analyze** : I calculated the stored water at each index a and b in my code. At the start of every loop, I update the current maximum height from left side (that is from A[0,1...a]) and the maximum height from right side(from A[b,b+1...n-1]). if(leftmax < rightmax) then, at least (leftmax-A[a]) water can definitely be stored no matter what happens between [a,b] since we know there is a barrier at the rightside(rightmax > leftmax).