title: Single Number
date: 2014-11-17 16:30:25
categories: Java
tags: [Bit Manipulation, leetcode]
---
##Single Number
Question : Given an array of integers, every element appears `twice` except for one. Find that single one.  
<!--more-->    
<br/>

###Best solution

```java
public class Solution {
    public int singleNumber(int[] A) {
        int result = 0;
		for (int i = 0; i < A.length; i++) {
			result ^= A[i];
		}
		return result;
    }
}
```
###My solution
```java
public class Solution {
    public int singleNumber(int[] A) {
        Map<Integer, Integer> map = new HashMap<Integer, Integer>();
		for (int i = 0; i < A.length; i++) {
			int key = A[i];
			if (map.containsKey(key)) {
				map.put(key, map.get(key) + 1);
				if (map.get(key) == 2) {
					map.remove(key);
				}
			} else {
				map.put(key, 1);
			}
		}
		String result = map.keySet().toString();
		int out = Integer.parseInt(result.substring(1, result.length()-1));
		return out;
	}
}
```

从性能上来评判用XOR无疑使最好的，复杂度只有O(N)。

我个人的方法略显笨拙，复杂度虽然也是O(N), 但开辟的多余的内存空间，不过当element出现的次数改变时，反而有更好的鲁棒性.     
<br/>
###change question
Given an array of integers, every element appears `three` times except for one. Find that single one.
<br/>
####best answer
```java
public int singleNumber(int[] A) {
    int ones = 0, twos = 0;
    for(int i = 0; i < A.length; i++){
        ones = (ones ^ A[i]) & ~twos;
        twos = (twos ^ A[i]) & ~ones;
    }
    return ones;
}
```
<br/>

####Inference
题目中改变了element的出现次数，变成了出现三次，当然用我之前的程序改个数字就能搞定，但仍需要分析一下最优的解法
best answer思路：出现三次，可以用2个bit来表征这三个状态，分别是00→10→01→00
ones和twos分别承担了这两个bit位
如果推理一下可以发现
>ones = ones ^ A[i]; if (twos == 1) then ones = 0, that can be tansformed to ones = (ones ^ >A[i]) & ~twos.
>
>twos = twos ^ A[i]; if (ones == 1) then twos = 0 and twos = (twos ^ A[i]) & ~ones.

<br/>

##回顾
<br/>
###位移动运算符:

**<<**表示左移, 左移一位表示原来的值乘2.

例如：3 <<2(3为int型) 
>1. 把3转换为二进制数字0000 0000 0000 0000 0000 0000 0000 0011， 
>2. 把该数字高位(左侧)的两个零移出，其他的数字都朝左平移2位， 
>3. 在低位(右侧)的两个空位补零。则得到的最终结果是0000 0000 0000 0000 0000 0000 0000 1100. 转换为十进制是12。

同理**>>**表示右移. 右移一位表示除2. 

低位移出(舍弃)，高位的空位补符号位，即正数补零，负数补1 

<br/>

###位运算:

位运算符包括:　与（&）、非（~）、或（|）、异或（^）

>&：当两边操作数的位同时为1时，结果为1，否则为0。如1100&1010=1000 　

>| ：当两边操作数的位有一边为1时，结果为1，否则为0。如1100|1010=1110 　

>~：0变1,1变0 　　

>^：两边的位不同时，结果为1，否则为0.如1100^1010=0110