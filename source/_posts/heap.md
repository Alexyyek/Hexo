title: HeapSort
date: 2014-12-07 14:52:35
categories: Java
tags: [heap, coding]
description: 堆数据结构是一种数组对象，它可以被视为一颗完全二叉树，树中每个节点与数组中存放该节点值的元素对应。
---

##堆排序
堆数据结构是一种数组对象，它可以被视为一颗完全二叉树，树中每个节点与数组中存放该节点值的元素对应。堆分为大根堆和小根堆；大根堆是指除根节点以外每个节点`i`都有`A[parent(i)]>=A[i]`，即每个节点的值最多和父节点的值一样大，这样堆中的最大值就在根节点中，且以某个节点为根的子树中，各个节点的值都不大于该子树根节点的值，图1就是一个大根堆。小根堆的刚好相反，每个节点的值都不小于其父节点。

<center>![](http://alexyoung.qiniudn.com/B44BD10DE99C61C6E965337C8EFFBE6A_B500_900_392_301.PNG)</center>

###保持堆的性质
**maxHeapify**是对大根堆进行操作的重要子程序，其输入为一个数组A和一个下标i，maxHeapify被调用时假定left(i)和right(i)都满足大根堆的性质，但是A[i]有可能小于其子女而违反了大根堆性质，maxHeapify使A[i]下降，使以A[i]为根的堆成为大根堆。其过程如图 2所示：
![](http://alexyoung.qiniudn.com/CBDE478800037BD91BF72E29598EFEBD_B500_900_500_341.PNG)
<br/>
```java
public void maxHeapify(int[] data, int length, int index){
		int left = getLeftChildIndex(index);
		int right = getRightChildIndex(index);
		int largest = index;
		
		if (left < length && data[left] > data[index]) {
			largest = left;
		}
		
		if (right < length && data[right] > data[largest]) {
			largest = right;
		}
		
		if (largest != index) {
			int temp = data[index];
			data[index] = data[largest];
			data[largest] = temp;
			maxHeapify(data, length, largest);
		}
	}
```
###建堆
**buildMaxHeap**算法是利用maxHeapify算法来进行的。用数组存储一个有n个元素堆时，叶节点的下标是n/2+1、n/2+2……n（这里就不做证明了，有兴趣可以自己证明），建立大根堆就是利用这个性质。叶节点可以看做只有一个元素的堆，只有一个元素也就自然满足大根堆的性质，所以以叶节点为根的堆都是大根堆。但是以其它节点为根的堆就不一定是大根堆，为了使他们满足大根堆的性质，就在节点上调用maxHeapify（叶节点是大根堆就满足了函数输入时的条件）。所以建立大根堆的过程就是在除叶节点以为的节点上调用maxHeapify。
```java
public void buildMaxHeap(int[] data){
		int start = getParentIndex(data.length);
		for (int i = start; i >= 0; i--) {
			maxHeapify(data, data.length, i);
		}
	}
```
###堆排序
**heapSort**也是利用maxHeapify算法来进行的。在一个大根堆中，最大元素就是堆的根。堆排序就是利用的这个性质。在一个含有n个元素的数组上调用buildMaxHeap，就能得到最大的元素，然后将最大的元素和数组的尾部，再将最大元素从堆中除去，此时堆的元素为n-1个，但是不满足大根堆的性质，就在根节点上调用maxHeapify使其成为大根堆。不断重复这个过程直到堆中只剩下一个元素，就完成了排序。
```java
public void heapSort(int[] data){
		for (int i = data.length - 1; i > 0; i--) {
			int temp = data[0];
			data[0]	= data[i];
			data[i] = temp;
			maxHeapify(data, i, 0);
		}
	}
```

##优先级队列
优先级队列是一种用来维护由一组数组元素构成集合S的数据结构，每个元素都有一个关键字key。
* INSERT(S,x)：元素x插入集合S
* MAXIMUN(S)：返回S中具有最大key的元素
* EXTRACT-MAX(S)：去掉并返回S中具有最大key的元素
* INCREASE-KEY(S,x,k)：将元素x的key值增加到k，k>=key

###伪代码
HEAP-MAXIMUN(A)，时间复杂度`O(1)`
```java
    return A[1];
```
HEAP-EXTRACT-MAX(A)，时间复杂度`O(lgn)`
```java
if heap-size[A]<1
    then error "heap underflow"
max ← A[1]
A[1] ← A[heap-size[A]]
heap-size[A] ← heap-size[A]-1
MAX-HEAPIFY(A,1)
return max
```
HEAP-INCREASE-KEY(A,i,key)，时间复杂度`O(lgn)`
```java
if key<A[i]
    then error "new key is smaller than the current key"
A[i] ← key
while i>1 && A[parent[i]]<A[i]
    do exchange A[i] ↔ A[parent[i]]
    i ← parent(i)
```
HEAP-INSERT(A,key)，时间复杂度`O(lgn)`
```java
heap-size[A] ← heap-size[A]+1
A[heap-size[A]] ← -∞
HEAP-INCREASE-KEY(A,heap-size[A],key)
```