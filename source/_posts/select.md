title: 中位数和顺序统计学
date: 2014-12-08 21:38:01
categories: Java
tags: coding
description: 从一个由N个不同数值构成的集合中选择其第i个顺序统计量。
---

##期望线性时间选择问题
从集合中选出第N大的数，RANDOMIZED-SELECT算法与快速排序类似，对输入数组进行递归划分，但RANDOMIZED-SELECT只处理划分的一边，因此快排的期望运行时间是`O(nlgn)`，而RANDOMIZED-SELECT`期望运行时间`只需要`O(n)` 

**伪代码**
```java
RANDOMIZED-SELECT(A,p,r,i)
if p==r
    return A[p]
q = RANDOM-PARTITION(A,p,r)
k = q - p + 1
if i=k
    return A[q]
elseif i < k
    return RANDOMIZED-SELECT(A,p,q-q,i)
else
    return RANDOMIZED-SELECT(A,q+1,r,i-k)
```
***
**代码**
```java
public static void main(String[] args){
		int[] data ={1,3,2,8,6,5,4,7,9,0,3};
		RandomizedSelect select = new RandomizedSelect();
		int rank = 9;
		System.out.println(select.select(data, 0, data.length-1, rank));
	}
	
	public int select(int[] data, int start, int end, int rank){
		if (start == end) {
			return data[start];
		}
		int threshold = randonPartition(data, start, end);
		int k = threshold - start + 1;
		if (rank == k) { 
			return data[threshold];
		}else if (rank < k) {
			return select(data, start, threshold-1, rank);
		}else {
			return select(data, threshold+1, end, rank-k);
		}
		
	}
	
	public int randonPartition(int[] data,int start,int end){
 		int q = start + (int)Math.random()*(end - start + 1);
 		int temp = data[q];
 		data[q] = data[end];
 		data[end] = temp;
		return partition(data, start, end);
	}
	
	public int partition(int[] data, int start, int end){
		int value = data[end];
		int init = start - 1;
		for (int i = start; i < end; i++) {
			if (data[i] < value) {
				init++;
				int temp = data[init];
				data[init] = data[i];
				data[i] = temp;
			}
		}
		int temp = data[end];
		data[end] = data[init+1];
		data[init+1] = temp;
		return init+1;
	}
```
在最坏的情况下，例如：总是找到最小元素时，总是在最大元素处划分，这是时间复杂度为`O(n^2)`。但平均时间复杂度与n呈线性关系，为`O(n)`
##最坏线性时间选择算法  
该算法利用了RANDOMIZED-SELECT对输入数组进行划分的思维，但不同之处在于该算法保证了数组的划分是好的划分。SELECT算法最坏情况运行时间为`O(n)`
<center>![](http://alexyoung.qiniudn.com/1357624243_8923.jpg)<center/>

算法思路：如果能在线性时间内找到一个划分基准使得按这个基准所划分出的2个子数组的长度都至少为原数组长度的`ε`倍`(0<ε<1)`,那么就可以在最坏情况下用`O(n)`时间完成选择任务。例如，当`ε=9/10`，算法递归调用所产生的子数组的长度至少缩短1/10。所以，在最坏情况下，算法所需的计算时间`T(n)`满足递推式`T(n)<=T(9n/10)+O(n)`。由此可得`T(n)=O(n)`

<br/>
***
**代码**
```java
public class Select {
	public static void main(String[] args){
		int length = 37; int rank = 10;/* 37个元素中找出第10小的元素 */  
		int[] data = new int[length];
		Random rm = new Random();
		for (int i = 0; i < length; i++) {
			data[i] = rm.nextInt(100);
		}
		
		Select select =  new Select();
		System.out.println(Arrays.toString(data));
		System.out.println(select.select(data, 0, length-1, rank));
		Arrays.sort(data);
		System.out.println(Arrays.toString(data));
	}
	
	private int select(int[] data,int low, int high, int rank){
		if ((high-low)<5) {
			insertSort(data, low, high);
			return data[low+rank-1];
		}
		int group = (high - low + 5) / 5;
		/* 第一步：将输入数组的n个元素划分为n/5组，每组5个元素，且至多只有一个组由剩下的n mod5个元素组成 */  
		for (int i = 0; i < group; i++) {
			int left = low + i*5;
			int right = (low + i*5 + 4) > high ? high : (low + i*5 + 4);
			int mid = (left + right) / 2;
		/* 第二步：寻找(n+4)/5个组中每一组的中位数。首先对每组中的元素（至多为5个）进行插入排序，然后从排序过的序列中选出中位数 */  
			insertSort(data, left, right);
			swap(data, low+i, mid);// 将中位数置前 
		}
		/* 第三步：对第二步中找出的(n+4)/5个中位数，递归调用select以找出其中位数x*/  
		int pivot = select(data, low, low+group-1, (group+1)/2);
		/* 第四步：利用修改过的partition过程，按中位数的中位数x对输入数组进行划分 */
		int threshold = partition(data, low, high, pivot);
		/* 第五步：判断threshold位置是否为要找的数，若不是则在低区或者高区递归select*/ 
		int k = threshold - low + 1;
		if (k == rank) {
			return data[threshold];
		}else if (k > rank) {
			return select(data, low, threshold-1, rank);
		}else {
			return select(data, threshold+1, high, rank-k);
		}
	}
	
	public int partition(int[] data, int low, int high, int pivot){
		int index = 0;
		/* 找到枢纽的位置index */  
		for (int i = low; i <= high; i++) {
			if (data[i] == pivot) {
				index = i;
				break;
			}
		}
		swap(data, index, high);
		int init = low - 1;
		for (int i = low; i < high; i++) {
			if (data[i] < pivot) {
				init++;
				swap(data, init, i);
			}
		}
		swap(data, init+1, high);
		return init+1;
	}
	
	private void swap(int[] data, int a, int b){
		int temp = data[a];
		data[a] = data[b];
		data[b] = temp;
	}
	
	public void insertSort(int[] data,int low,int high){
		for (int i = low+1; i <= high; i++) {
			int key = data[i];
			int k = i - 1;
			while (k >= low && data[k] > key) {
				data[k+1] = data[k];
				k--;
			}
			data[k+1] = key;
		}
	}
}
```
