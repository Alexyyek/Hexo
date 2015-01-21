title: 线性时间排序
date: 2014-12-07 21:43:08
categories: Java
tags: coding
description: 不同于比较排序O(lgn)的时间复杂度，线性时间排序可以在O(n)的时间完成排序。
---

##快速排序算法
* 分解 
数组`A[p..r]`被划分为两个子数组，`A[p..q-1]`和`A[q+1..r]`，使得`A[p..q-1]`中每个元素都小于等于`A(q)`，且小于等于`A[q+1..r]`中元素。
```java
QUICKSORT(A,p,r)
//正常方法
if (p < r){
    then q = PARTITION(A,p,r)
    QUICKSORT(A,p,q-1)
    QUICKSORT(A,q+1,r)
}
//尾递归方法
while (start < end) {
	int q = partition(data, start, end);
	quickSort(data, start, q-1);
	start = q + 1;
}
```
* 解决
通过递归调用，两个子数组各自内部排序
```java
PARTITION(A,p,r)
x = A[r]
i = p-1
for j=p to r-1
    if A[j] < x
    i++
    A[j]=A[i]
exchange A[i+1] ↔ A[r]
return i+1
```
因为子数组是内部排序，所以无需合并操作，平均时间复杂度`O(nlgn)`

##线性时间排序
之前的排序算法都属于`比较排序`，任何比较排序在最坏的情况下都要做`Ω(nlgn)`次比较来进行排序，下面介绍三种线性时间运行的排序

###计数排序
计数排序的前提是确定输入范围大小为0～k。在这个前提下，我们可以使用计数的方法对数组进行排序，而不是使用比较。算法思想如下：因为输入数组A中的元素范围固定，因此可以使用一个大小为k的数组C对A中的元素进行映射。
<center>![](http://alexyoung.qiniudn.com/QQ图片201412007210801.png)<center/>

**伪代码**
```java
COUNTING-SORT(A,B,k)
for i←0 to k
    do C[i] = 0
for j←1 to length[A]
    do C[A[j]] = C[A[j]] + 1
    C[i]中包含等于i的元素个数
for i←1 to k
    C[i] = C[i-1] + C[i]
for j←1 to length[A]
    do B[C[A[j]]] = A[j]
    C[A[j]]--
```

###基数排序
实现原理：将所有待比较数值（正整数）统一为同样的数位长度，数位较短的数前面补零。然后，从最低位开始，依次进行一次排序。这样从最低位排序一直到最高位排序完成以后, 数列就变成一个有序序列。给定n个d位数，每一个数位有k种可能取值，基数排序能以`O(d(n+k))`的时间正确排序。
基数排序的方式可以采用LSD（Least significant digital）或MSD（Most significant digital），LSD的排序方式由键值的最右边开始，而MSD则相反，由键值的最左边开始。
<center>![](http://alexyoung.qiniudn.com/1338051031_6070.jpg)<center/>

**代码**
```java
private void sort(int[] data, int radix){
		int queue = 0;
		int bit = 1; //数位
		int position = 1;//当前数位
		int[][] reminder = new int[10][data.length];
		int[] order = new int[10];
		
		while (position <= radix) {
			for (int i = 0; i < data.length; i++) {
				int lsd = (data[i] / bit) % 10;
				reminder[lsd][order[lsd]] = data[i];
				order[lsd]++;
			}
			for (int i = 0; i < 10; i++) {
				if (order[i] != 0) {
					for (int j = 0; j < order[i]; j++) {
						data[queue++] = reminder[i][j];
					}
				}
				order[i] = 0;
			}
			bit *= 10;
			queue = 0;
			position++;
		}
	}
```
###桶排序
桶排序的思想是把区间[0,1)划分成n个相同大小的子区间，称为桶，然后将n个输入数分布到各个桶中去。因为输入数均匀且独立分布在[0,1)上，所以，一般不会有很多数落在一个桶中的情况。为了得到结果，先对各个桶中的数进行排序，然后按次序把各桶中的元素列出来。桶排序的时间复杂度为`O(n)`
<center>![](http://alexyoung.qiniudn.com/bucket_sort.PNG)<center/>

**伪代码**
```JAVA
BUCKET-SORT(A)
n = length(A)
for i=1 to n
    do insert A[i] to list Math.floor(B[n*A[i]])
for i=1 to n
    do sort list B[i] (recommand Collections.sort)
concatenate the list B[1] to B[n] INTO A
```