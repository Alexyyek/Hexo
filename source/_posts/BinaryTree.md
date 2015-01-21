title: 二叉查找树
date: 2014-12-17 17:39:28
categories: Java
tags: [Tree, coding]
description: 二叉查找树是一种数据结构，支持多种动态集合操作。在二叉查找树上执行的基本操作时间与树高成正比。
---

##二叉查找树
一颗二叉查找树是按二叉树的结构来组织的，树中每个结点都是一个对象，节点中除了`key`域外，还包含域`left`，`right`以及`parent`，分别指向当前节点的左子节点、右子节点和父节点。
<center> ![](http://7rf27k.com1.z0.glb.clouddn.com/QQ图片20141217175217.png) <center/>

在二叉查找树中：
1. 若任意节点的左子树不空，则左子树上所有结点的值均小于它的根结点的值；
2. 任意节点的右子树不空，则右子树上所有结点的值均大于它的根结点的值；
3. 任意节点的左、右子树也分别为二叉查找树。
4. 没有键值相等的节点（待定）

###遍历操作
根据二叉查找树的性质，可以由递归算法按排列顺序输出树中的所有关键字。

**中序遍历**
```java
public void inOrder(Node root){
		if (root != null) {
			inOrder(root.left);
			System.out.print(root.value +"\t");
			inOrder(root.right);
		}
	}
```
前序遍历和后续遍历形式基本一致，前序先输出root节点，后续最后输出root节点。

###查找操作
从二叉查找树根节点开始查找，并沿着树下降，对每个碰到的非空节点比较节点值与查找值大小，按情况向左右子树下降，直到找到并返回节点对象。非递归版本普遍较快。

**递归版本**
```java
public Node search(Node tree, int value){
		if (tree == null || value == tree.value) {
			return tree;
		}
		if (value <= tree.value) {
			return search(tree.left, value);
		}else {
			return search(tree.right, value);
		}
	}
```

**非递归版本**
```java
public Node searchNode(Node tree, int value){
		while (tree != null && tree.value != value) {
			if (value <= tree.value) {
				tree = tree.left;
			}else {
				tree = tree.right;
			}
		}
		return tree;
	}
```
###插入操作
插入操作与查找操作类似，将待插值与当前节点值比较，小于当前节点值则去左子树继续比较，大于当前节点值去右子树比较，直到下降直空节点，则将待插值赋给空节点并返回。
```java
public Node insertNode(Node tree, int x){
		if (tree == null) {
			return new Node(x);
		}
		
		if (x <= tree.value) {
			tree.left = insertNode(tree.left, x);
		}else {
			tree.right = insertNode(tree.right, x);
		}
		return tree;
	}
```

###删除操作
删除操作根据所删节点的子女数量可能会改变树的结构，主要分为三种情况：
1. 无children节点，直接删除
![](http://7rf27k.com1.z0.glb.clouddn.com/20130506115644277.png)
2. 有一个children节点，将子节点覆盖删除节点
![](http://7rf27k.com1.z0.glb.clouddn.com/20130506115712159.png)
3. 两个children节点都不为空，寻址其后继节点（右子树中的最小值），用后继节点值替换删除节点值，然后在其右子树中删除后继节点
![](http://7rf27k.com1.z0.glb.clouddn.com/QQ图片20141217174837.jpg)

###二叉查找树性能
对高度为**`h`**的二叉查找树，动态集合操作INSERT和DELETE的运行时间为**`O(h)`**

##完整代码
###简单版
```java
public class BinarySearchTree {
	public class Node{
		Node left;
		Node right;
		int value;
		
		public Node(int value){
			this.value = value;
			left = null;
			right = null;
		}
	}
	private Node root;
	
	public BinarySearchTree(){
		root = null;
	}
	
	public BinarySearchTree(int[] arr){
		for(int i : arr){
			insert(i);
		}
	}
	
	public void insert(int value){
		root = insert(root, value);
	}
	
	public Node insert(Node root, int value){
		if (root == null) {
			root = new Node(value);
		}else if (value <= root.value) {
			if (root.left == null) {
				root.left = new Node(value);
			}else {
				insert(root.left, value);
			}
		}else {
			if (root.right == null) {
				root.right = new Node(value);
			}else {
				insert(root.right, value);
			}
		}
		return root;
	}
	
	//中序遍历
	public void inOrder(Node tree){
		if (tree != null) {
			inOrder(tree.left);
			System.out.print(tree.value + "\t");
			inOrder(tree.right);
		}
	}
	//前序遍历
	public void preOrder(Node tree){
		if (tree != null) {
			System.out.print(tree.value +"\t");
			preOrder(tree.left);
			preOrder(tree.right);
		}
	}
	//后续遍历
	public void postOrder(Node tree){
		if (tree != null) {
			postOrder(tree.left);
			postOrder(tree.right);
			System.out.print(tree.value +"\t");
		}
	}
	//递归
	public Node search(Node tree, int value){
		if (tree == null || value == tree.value) {
			return tree;
		}
		
		if (value <= tree.value) {
			return search(tree.left, value);
		}else {
			return search(tree.right, value);
		}
		
	}
	//非递归
	public Node searchNoIter(Node root, int value){
		while (root != null && value != root.value) {
			if (value <= root.value) {
				root = root.left;
			}else {
				root = root.right;
			}
		}
		if (root == null) {
			throw new NullPointerException("Null Poninter" + value);
		}else {
			return root;
		}
	}
	//	最小值节点
	public Node minNode(Node root){
		while (root.left != null) {
			root = root.left;
		}
		return root;
	}
	//	最大值节点
	public Node maxNode(Node root){
		while (root.right != null) {
			root = root.right;
		}
		return root;
	}
	
	public Node insertNode(Node root, Node x){
		if (root == null) {
			return new Node(x.value);
		}
		
		if (x.value <= root.value) {
			root.left = insertNode(root.left, x);
		}else {
			root.right = insertNode(root.right, x);
		}
	
		return root;
	}
	
	
	public Node remove(Node root, int value){
		if (root == null) {
			return root;
		}
		if (value < root.value) {
			root.left = remove(root.left, value);
		}else if (value > root.value) {
			root.right = remove(root.right, value);
		}else if (root.left != null && root.right != null) {
			root.value = minNode(root.right).value;
			root.right = remove(root.right, root.value);
		}else {
			root = (root.left != null)? root.left:root.right;
		}
		return root;
	}
```

<br/>
***

###泛型带parent节点版本
```java
package Algorithm;


public class BiSearchTree<T extends Comparable<T>> {
	private Node<T> root;
	public class Node<T extends Comparable<T>>{
		T key;
		Node<T> left;
		Node<T> right;
		Node<T> parent;
		
		public Node(T key, Node<T> parent, Node<T> left, Node<T>right){
			this.key = key;
			this.parent = parent;
			this.left = left;
			this.right = right;
		}
		
		public T getKey(){
			return key;
		}
	}
	
	public BiSearchTree(){
		root = null;
	}
	
	public void inOrder(Node<T> root){
		if (root != null) {
			inOrder(root.left);
			System.out.print(root.key + "\t");
			inOrder(root.right);
		}
	}
	
	public Node<T> search(Node<T> tree, T key){
		if (tree == null || key == tree.key) {
			return tree;
		}
		
		int cmp = key.compareTo(tree.key);
		if (cmp < 0) {
			return search(tree.left, key);
		}else {
			return search(tree.right, key);
		}
	}
	
	public Node<T> searchNode(Node<T> tree, T key){
		while (tree != null && key != tree.key) {
			int cmp = key.compareTo(tree.key);
			if (cmp < 0) {
				tree = tree.left;
			}else if (cmp > 0) {
				tree = tree.right;
			}
		}
		return tree;
	}
	
	public Node<T> minNode(Node<T> tree){
		while (tree.left != null) {
			tree = tree.left;
		}
		return tree;
	}
	
	public Node<T> maxNode(Node<T> tree){
		while (tree.right != null) {
			tree = tree.right;
		}
		return tree;
	}
	/**
	 * 如果x没有右孩子。则x有以下两种可能：
	 * (01) x是"一个左孩子"，则"x的后继结点"为 "它的父结点"。
	 * (02) x是"一个右孩子"，则查找"是其父节点的左儿子的节点"为止，找到的这个"最低的父结点"就是"x的后继结点"。
	 * @param x
	 * @return
	 */
	public Node<T> successor(Node<T> x){
		// 如果x存在右孩子，则"x的后继结点"为 "以其右孩子为根的子树的最小结点"。
		if (x.right != null) {
			return minNode(x.right);
		}
		Node<T> before = x.parent;
		while (before != null && x == before.right) {
			x = before;
			before = before.parent;
		}
		return before;
	}
	// 如果x没有左孩子。则x有以下两种可能：
	// (01) x是"一个右孩子"，则"x的前驱结点"为 "它的父结点"。
	// (01) x是"一个左孩子"，则查找"是其父节点的右儿子节点"，找到的这个"最低的父结点"就是"x的前驱结点"。
	public Node<T> preDecessor(Node<T> x){
		if (x.left != null) {
			return maxNode(x.left);
		}
		Node<T> before = x.parent;
		while (before != null && x == before.left) {
			x = before;
			before = before.right;
		}
		return before;
	}
	/**
	 * 先找到插入位置的父节点，如果父节点为null，则插到父节点，否则插入左或右节点
	 * @param tree
	 * @param node 待插入节点
	 */
	public void insertNode(Node<T> tree, Node<T> node){
		Node<T> before = null;
		Node<T> index = tree;
		while (index != null) {
			before = index;
			if (node.key.compareTo(index.key) < 0) {
				index = index.left;
			}else {
				index = index.right;
			}
		}
		node.parent = before;
		if (before == null) {
			tree = node;
		}else if (node.key.compareTo(before.key) < 0) {
			before.left = node;
		}else {
			before.right = node;
		}
	}
	
	public Node<T> remove(Node<T> tree, T key){
		Node<T> node = search(tree, key);
		if (node == null) {
			return node;
		}
		Node<T> index = null;
		Node<T> child = null;
		if (node.left == null || node.left == null) {
			index = node;
		}else {
			index = successor(node);
		}
		
		if (index.left != null) {
			child = index.left;
		}else {
			child = index.right;
		}
		
		if (child != null) {
			child.parent = index.parent;
		}
		
		if (index.parent == null) {
			tree = child;
		}else if (index == index.parent.left) {
			index.parent.left = child;
		}else {
			index.parent.right = child;
		}
		
		if (index != node) {
			node.key = index.key;
		}
		return index;
	}
}
```