title: 红黑树
date: 2014-12-23 16:13:09
categories: Java
tags: [Tree, coding]
description: 一颗高度为H的二叉树可以实现任何一种基本的动态集合操作，时间都为O(h)，当数的高度较低时，这种操作较快，但是当树的高度较高时，红黑树由于其平衡树的特点，保证了基本动态操作时间为O(lgn)。
---

##红黑树
红黑树(Red-Black Tree，简称R-B Tree)，它一种特殊的二叉查找树。
红黑树是特殊的二叉查找树，意味着它满足二叉查找树的特征：任意一个节点所包含的键值，大于等于左孩子的键值，小于等于右孩子的键值。
除了具备该特性之外，红黑树还包括许多额外的信息。

###性质
红黑树的每个节点上都有存储位表示节点的颜色，颜色是红(Red)或黑(Black)。
红黑树的特性:
1. 每个节点或者是黑色，或者是红色。
2. 根节点是黑色。
3. 每个叶子节点是黑色。 [注意：这里叶子节点，是指为空的叶子节点！]
4. 如果一个节点是红色的，则它的子节点必须是黑色的。
5. 从一个节点到该节点的子孙节点的所有路径上包含相同数目的黑节点。

关于它的特性，需要注意的是：
第一，特性(3)中的叶子节点，是只为空(NIL或null)的节点。
第二，特性(5)，确保没有一条路径会比其他路径长出俩倍。因而，红黑树是相对是接近平衡的二叉树。

红黑树结构图类似：
<center> ![](http://alexyoung.qiniudn.com/8394323_13013004048ddW.jpg) <center/>
<br/>
***

###基本操作
从某个结点x出发（不包括该结点）到达一个叶节点的任意一条路径上，黑色结点的个数成为该结点x的黑高度，用`bh(x)`表示。根据性质5黑高度概念是明确定义的，红黑树的黑高度定义为其根节点的黑高度。

**性质**：一颗有n个内结点的红黑树的高度至多为`2lg(n+1)`，红黑树的插入和删除操作都可以在`O(lgn)`时间内完成。
<br/>

####旋转
红黑树的旋转分为左旋和右旋，因为在插入和删除基本操作中可能改变树的结构，从而破坏红黑树的基本性质，所以需要一定的旋转操作。

<center> ![](http://alexyoung.qiniudn.com/251733282013849.jpg) <center/>

从图上可以看到，左旋操作中将Y节点顶替其父节点X，之前的父节点X成为Y的左子树，Y的右子树不动，Y的左子树移到X的右子树
```java
private void leftRotate(Node<T> x){
		//consider the left child of y
		Node<T> y = x.right;
		x.right = y.left;
		if (y.left != null) {
			y.left.parent = x;
		}
		//consider the node y
		y.parent = x.parent;
		if (x.parent == null) {
			root = y;
		}else{
			if(x == x.parent.left)
			x.parent.left = y;
			else 
			x.parent.right = y;
		}
		//consider the Node x
		y.left = x;
		x.parent = y;
	}
```

右旋同理，是左旋的逆操作：
<center> ![](http://alexyoung.qiniudn.com/251735527958942.jpg) <center/>

<br/>

***

<br/>

####插入

将一个节点插入到红黑树中，需要执行哪些步骤呢？首先，将红黑树当作一颗二叉查找树，将节点插入；然后，将节点着色为红色；最后，通过"旋转和重新着色"等一系列操作来修正该树，使之重新成为一颗红黑树。详细描述如下：
**第一步**： 将红黑树当作一颗二叉查找树，将节点插入。
红黑树本身就是一颗二叉查找树，将节点插入后，该树仍然是一颗二叉查找树。也就意味着，树的键值仍然是有序的。此外，无论是左旋还是右旋，若旋转之前这棵树是二叉查找树，旋转之后它一定还是二叉查找树。这也就意味着，任何的旋转和重新着色操作，都不会改变它仍然是一颗二叉查找树的事实。

**第二步**：将插入的节点着色为"红色"。
为什么着色成红色，而不是黑色呢？为什么呢？在回答之前，我们需要重新温习一下红黑树的特性：
(1) 每个节点或者是黑色，或者是红色。
(2) 根节点是黑色。
(3) 每个叶子节点是黑色。 注意：这里叶子节点，是指为空的叶子节点！
(4) 如果一个节点是红色的，则它的子节点必须是黑色的。
(5) 从一个节点到该节点的子孙节点的所有路径上包含相同数目的黑节点。

将插入的节点着色为红色，不会违背"特性(5)"！少违背一条特性，就意味着我们需要处理的情况越少。接下来，就要努力的让这棵树满足其它性质即可；满足了的话，它就又是一颗红黑树了。

**第三步**：通过一系列的旋转或着色等操作，使之重新成为一颗红黑树。
    第二步中，将插入节点着色为"红色"之后，不会违背"特性(5)"。那它到底会违背哪些特性呢？
    对于"特性(1)"，显然不会违背了。因为我们已经将它涂成红色了。
    对于"特性(2)"，唯一可能违背的情况是插入的节点为根节点。
    对于"特性(3)"，显然不会违背了。这里的叶子节点是指的空叶子节点，插入非空节点并不会对它们造成影响。
    对于"特性(4)"，如果插入节点的父节点是红色的，就有可能违背！
    那接下来，想办法使之"满足特性"，就可以将树重新构造成红黑树了。
    
插入操作代码见最后的完整代码，下面主要研究插入后的修正操作：

* 修复情况1：当前结点的父结点是红色，祖父结点的另一个子结点（叔叔结点）是红色。

<center> ![](http://alexyoung.qiniudn.com/Red-black_tree_insert_case_3.png) <center/>

父节点和叔叔节点都是RED，则祖父节点肯定存在，且为BLACK，改变祖父节点、父节点和叔叔节点的颜色，将N上移到祖父节点继续迭代。

* 修复情况2：当前节点的父节点是红色,叔叔节点是黑色，当前节点是其父节点的右孩子

<center> ![](http://alexyoung.qiniudn.com/Red-black_tree_insert_case_4.png) <center/>

此时当前节点的父节点做为新的当前节点，以新当前节点为支点左旋。

* 插入修复情况3：当前节点的父节点是红色,叔叔节点是黑色，当前节点是其父节点的左孩子

<center> ![](http://alexyoung.qiniudn.com/Red-black_tree_insert_case_5.png) <center/>

情况2通过一个左旋转变成情况三，因为节点N和父节点P都是红色的，所以旋转不影响数的黑高度。情况三中改变一些节点颜色，然后通过一个右旋解决。代码如下：
```java
private void insertFixUp(Node<T> node){
		//	若父节点存在，并且父节点的颜色是红色
		while(node.parent != null && isRed(node.parent)) {
			// 若父节点”是“祖父节点的左孩子
			if (parentOf(node) == parentOf(parentOf(node)).left) {
				Node<T> uncle = parentOf(parentOf(node)).right;	 //叔叔节点，即父节点的兄弟节点
				//情况一：如果叔叔节点是红色，因为父节点是红色，显然祖父节点是黑色
				if (isRed(uncle)) {									
					parentOf(node).color = BLACK;
					uncle.color = BLACK;
					parentOf(parentOf(node)).color = RED;
					node = parentOf(parentOf(node));
				}
				else {	
					if (parentOf(node).right == node) {	//情况二：叔叔节点是黑色，而Z是右孩子
						 node = parentOf(node);
						 leftRotate(node);
					}
					
					parentOf(node).color = BLACK;	//情况三：叔叔节点黑色，Z是左孩子
					parentOf(parentOf(node)).color = RED;
					rightRotate(parentOf(parentOf(node)));
				}
			}
			// 若父节点”是“祖父节点的右孩子
			else {
				Node<T> uncle = parentOf(parentOf(node)).left;
				if (isRed(uncle)) {
					parentOf(node).color = BLACK;
					uncle.color = BLACK;
					parentOf(parentOf(node)).color = RED;
					node = parentOf(parentOf(node));
				}
				else {
					if (parentOf(node).left == node) {
						 node = parentOf(node);
						 rightRotate(node);
					}
					parentOf(node).color = BLACK;
					parentOf(parentOf(node)).color = RED;
					leftRotate(parentOf(parentOf(node)));
				}
			}
		}
		/* 1.防止插入节点为根节点        
		 * 2.防止树深度较低时grandparent为root节点,从而在情况一uncle为red时，把root节点置为red 	 */
		if (node == root) {
			node.color = BLACK;
		}
	}
```
<br/>

***

<br/>

####删除
将红黑树内的某一个节点删除。需要执行的操作依次是：首先，将红黑树当作一颗二叉查找树，将该节点从二叉查找树中删除；然后，通过"旋转和重新着色"等一系列来修正该树，使之重新成为一棵红黑树。详细描述如下：
**第一步**：将红黑树当作一颗二叉查找树，将节点删除。
这和"删除常规二叉查找树中删除节点的方法是一样的"。分3种情况：
1. 被删除节点没有儿子，即为叶节点。那么，直接将该节点删除就OK了。
2. 被删除节点只有一个儿子。那么，直接删除该节点，并用该节点的唯一子节点顶替它的位置。
3. 被删除节点有两个儿子。那么，先找出它的后继节点；然后把“它的后继节点的内容”复制给“该节点的内容”；之后，删除“它的后继节点”。在这里，后继节点相当于替身，在将后继节点的内容复制给"被删除节点"之后，再将后继节点删除。这样就巧妙的将问题转换为"删除后继节点"的情况了,从而转换成为情况1或情况2

**第二步**：通过"旋转和重新着色"等一系列来修正该树，使之重新成为一棵红黑树。
因为"第一步"中删除节点之后，可能会违背红黑树的特性。所以需要通过"旋转和重新着色"来修正该树，使之重新成为一棵红黑树。

删除操作代码见最后的完整代码，下面主要研究插入后的修正操作：
当删除节点是红色的，红黑性质仍然得以保存，故考虑删除节点是黑色的情况。

* 修复情况1：兄弟节点为红色(此时父节点和兄弟节点的子节点分为黑)。

<center> ![](http://alexyoung.qiniudn.com/Red-black_tree_delete_case_2.png) <center/>

因为兄弟节点必有黑孩子，可以改变兄弟节点S和父节点P的颜色，在对P做一次左旋，从而转换为情形2-4，图中转换为情况2

* 修复情况2：兄弟是黑色且兄弟节点的两个子节点全为黑色。

<center> ![](http://alexyoung.qiniudn.com/Red-black_tree_delete_case_4.png) <center/>

因为S节点和S的两个子节点都是黑色，故需要从S支数去除一层黑色，来平衡黑高度，故将S的颜色改为RED，并将N直到P上，继续迭代。

* 修复情况3：兄弟节点是黑色，兄弟的左子是红色，右子是黑色。

<center> ![](http://alexyoung.qiniudn.com/Red-black_tree_delete_case_5.png) <center/>

交换兄弟节点S与其左孩子的颜色，然后对S进行右旋，现在N的新兄弟S是一个有红色右孩子的黑节点，进入情况4

* 修复情况4：当前节点颜色是黑-黑色，它的兄弟节点是黑色，但是兄弟节点的右子是红色，兄弟节点左子的颜色任意。

<center> ![](http://alexyoung.qiniudn.com/Red-black_tree_delete_case_6.png) <center/>

修改某些节点颜色，并对P做一次左旋，将N置为根，循环结束。代码如下：

```java
private void removeFixUp(Node<T> node){
	
		while (node != this.root && node.color == BLACK) {
			// 若节点是父节点的左孩子
			if (node == parentOf(node).left) {
				Node<T> uncle = parentOf(node).right;		
				if (isRed(uncle)) {		//情况一：叔叔节点为红色
					parentOf(node).color = RED;
					uncle.color = BLACK;
					leftRotate(parentOf(node));
					uncle = parentOf(node).right;
				}
				//情况二：叔叔节点为黑色，且有两个黑孩子
				if (isBlack(uncle.left) && isBlack(uncle.right)) {
					uncle.color = RED;
					node = parentOf(node);
				}else {										
					if (isBlack(uncle.right)) {		//情况三：叔叔节点的左孩子为红色，右孩子为黑色
						uncle.left.color = BLACK;
						uncle.color = RED;
						rightRotate(uncle);
						uncle = parentOf(node).right;
					}
					uncle.color = parentOf(node).color;		//情况四：叔叔节点的右孩子为红色
					parentOf(node).color = BLACK;
					uncle.right.color = BLACK;
					leftRotate(parentOf(node));
					node = root;
				}
			}else {
				Node<T> uncle = parentOf(node).left;
				if (isRed(uncle)) {
					parentOf(node).color = RED;
					uncle.color = BLACK;
					rightRotate(parentOf(node));
					uncle = parentOf(node).left;
				}
				if (isBlack(uncle.left) && isBlack(uncle.right)) {
					uncle.color = RED;
					node = parentOf(node);
				}else {
					if (isBlack(uncle.left)) {
						uncle.right.color = BLACK;
						uncle.color = RED;
						leftRotate(uncle);
						uncle = parentOf(node).left;
					}
					uncle.color = parentOf(node).color;
					parentOf(node).color = BLACK;
					uncle.left.color = BLACK;
					rightRotate(parentOf(node));
					node = root;
				}
			}
		}
		node.color = BLACK;
	}
```

<br/>
***
<br/>

###动态顺序统计

在之前的[**顺序统计学**](http://alexyyek.github.io/2014/12/08/select/)问题中，我们尝试在`O(n)`的时间内找到无序集合中第i小的数。这里将介绍如何修改红黑树的结构，使得任意的顺序统计量都可以在`O(lgn)`时间内确定。

一颗**顺序统计量数**通过在红黑树的每个节点中存入附加信息而成，即除了节点的key值，还包含了域`size`，如下图size的规则为：
<center> $size[x]=size[left[x]]+size[right[x]]+1$ <center/>

<center> ![](http://alexyoung.qiniudn.com/01212325-22d4dc3f9b39450e9e5254db69372047.png)<center/>


**如何扩张数据结构：**
* 选择基础的数据结构
* 确定要在基础数据结构中添加哪些信息（根据需要）
* 验证可用基础数据结构上的基础修改操作来维护新添加的信息
* 设计新的操作

<br/>

####检索给定排序的元素
为了找出顺序统计数中第i小的关键字，递归版伪代码如下：
```java
OS-SELECT(root[T],i)
r = size[left[x]] + 1
if r = i
    then return x
else if i < r
    then return OS-SELECT(left[x],i)
else 
    return OS-SELECT(right[x], i-r)
```
<br/>

####确定一个元素的秩
过程OS-RANK返回在对树进行中序遍历得到的线性序中x的位置
```java
OS-RANK(T,x)
r = size[left[x]] + 1
y = x
while y != root[T]
    do if y = right[p[y]]
        r = r + size[left[p[y]]] + 1
        y = p[y]
return r
```

<br/>

####对子树规模的维护
红黑树的插入和删除可能会改变树的结构，为了维护红黑树的结构，需要更新域size的值，左旋在原有代码下增加两行：
<center> $size[y] = size[x]$ <center/>
<center> $size[x] = size[left[x]] + size[right[x]] + 1$ <center/>

<center> ![](http://alexyoung.qiniudn.com/101600338568211.png) <center/>

点击[**具体讲解**](http://www.cnblogs.com/alan-forever/p/3657086.html)

```C
/*
 * x结点上进行左旋转，y结点（x结点的右儿子）的左儿子成为x结点的新右儿子
 * x结点成为y结点的左儿子的新父结点
 * x结点的父结点成为y结点的新父结点，y结点成为x结点的新父结点
 * x结点成为y结点的新左儿子
 */
void Left_Rotate(RB_Tree *T, RB_TreeNode *x){
    RB_TreeNode *y = x->right;       //x点右儿子          
    x->right = y->left;              //y结点的左儿子会成为x结点的右儿子
    if(y->left != NIL)
        y->left->parent = x;         //如果y有左儿子,y的左儿子的父结点为x
    y->parent = x->parent;
    if(x->parent == NIL)
        T->root = y;                //如果x的父结点为哨兵，说明x为根结点，则y成为根结点
    else if(x == x->parent->left)
        x->parent->left = y;        
    else
        x->parent->right = y;       //判断x为其父结点的左、右儿子，y成为x父结点对应的儿子
    y->left = x;                    //y的左儿子为x
    x->parent = y;                  //x的父结点为y

    /*调整size域*/
    y->size = x->size;
    x->size = x->left->size + x->right->size + 1;
}
```
<br/>

***

###完整代码
```java
package Algorithm;

/**
 * red-black tree
 * @author AlexYoung
 *
 */
public class RBTree<T extends Comparable<T>> {
	private Node<T> root;
	
	private static final boolean RED = true;
	private static final boolean BLACK = false;
	
	public class Node<T extends Comparable<T>>{
		T key;
		Node<T> left;
		Node<T> right;
		Node<T> parent;
		boolean color;
		
		
		public Node(T key, Node<T> left, Node<T> right, Node<T> parent, Boolean color){
			this.key = key;
			this.left = left;
			this.right = right;
			this.parent = parent;
			this.color = color;
		}
		
		public T getKey(){
			return key;
		}
		
		public String toString(){
			return "" + key + " " + color;
		}
	}
	
	public RBTree(){
		root = null;
	}
	
	private Node<T> parentOf(Node<T> node){
		return (node != null) ? node.parent : null;
	}
	private boolean colorOf(Node<T> node){
		return (node != null) ? node.color : BLACK;
	}
	private boolean isRed(Node<T> node){
		return (node != null) && (node.color == RED) ? true : false;
	}
	private boolean isBlack(Node<T> node){
		return !isRed(node);
	}
	
	private Node<T> miNode(Node<T> node){
		while (node.left != null) {
			node = node.left;
		}
		return node;
	}
	
	private void inOrder(Node<T> tree){
		if(tree != null) {
			inOrder(tree.left);
			System.out.print(tree.key + "\t");
			inOrder(tree.right);
		}
	}
	
	public void inOrder(){
		inOrder(root);
	}
	
	private Node<T> maxNode(Node<T> node){
		while (node.right != null) {
			node = node.right;
		}
		return node;
	}
	// 前趋
	private Node<T> succussor(Node<T> node){
		if (node.right != null) {
			return miNode(node.right);
		}
		Node<T> parent = node.parent;
		while (parent != null && node == parent.right) {
			node = parent;
			parent = parent.parent;
		}
		return parent;
	}
	// 后继
	private Node<T> preDecessor(Node<T> node){
		if (node.left != null) {
			return maxNode(node.left);
		}
		Node<T> parent = node.parent;
		while (parent != null && node == parent.left) {
			node = parent;
			parent = parent.parent;
		}
		return parent;
	}
	//	左旋
	private void leftRotate(Node<T> x){
		//consider the left child of y
		Node<T> y = x.right;
		x.right = y.left;
		if (y.left != null) {
			y.left.parent = x;
		}
		//consider the node y
		y.parent = x.parent;
		if (x.parent == null) {
			root = y;
		}else{
			if(x == x.parent.left)
			x.parent.left = y;
			else 
			x.parent.right = y;
		}
		//consider the Node x
		y.left = x;
		x.parent = y;
	}
	//	右旋
	private void rightRotate(Node<T> x){
		Node<T> y= x.left;
		x.left = y.right;
		if (y.right != null) {
			y.right.parent = x;
		}
		
		y.parent = x.parent;
		if (x.parent == null) {
			root = y;
		}else{
			if (x == x.parent.left)
			x.parent.left = y;
			else 
			x.parent.right = y;
		}
		
		y.right = x;
		x.parent =y;
	}
	//递归查找
	public Node<T> search(Node<T> tree, T key){
		if (tree == null || key == tree.key) {
			return tree;
		}
		
		if (key.compareTo(tree.key) <= 0) {
			return search(tree.left, key);
		}else {
			return search(tree.right, key);
		}
	}
	//非递归查找
	public Node<T> searchNode(Node<T> tree, T key){
		while (tree != null && tree.key != key) {
			if (key.compareTo(tree.key) <= 0) {
				tree = tree.left;
			}else {
				tree = tree.right;
			}
		}
		return tree;
	}
	
	/**
	 * 将结点插入到红黑树中
	 * @param node : 插入的结点
	 */
	private void insert(Node<T> node){
		Node<T> y = null;
		Node<T> x = this.root;
		while (x != null) {
			y = x;
			if (node.key.compareTo(x.key) <= 0) {
				x = x.left;
			} else {
				x = x.right;
			}
		}

		node.parent = y;
		if (y == null) {
			root = node;
			node.color = BLACK;
		} else {
			if (node.key.compareTo(y.key) <= 0) {
				y.left = node;
				node.color = RED;
			} else {
				y.right = node;
				node.color = RED;
			}
		}
		
		
		insertFixUp(node);
	}
	/**
	 * 
	 * @param key : 插入节点的键值
	 */
	public void insert(T key){
		Node<T> node = new Node<T>(key, null, null, null, BLACK);
		if (node != null) {
			insert(node);
		}
	}
	
	private void insertFixUp(Node<T> node){
		//	若父节点存在，并且父节点的颜色是红色
		while(node.parent != null && isRed(node.parent)) {
			// 若父节点”是“祖父节点的左孩子
			if (parentOf(node) == parentOf(parentOf(node)).left) {
				Node<T> uncle = parentOf(parentOf(node)).right;		//叔叔节点，即父节点的兄弟节点
				if (isRed(uncle)) {									//情况一：如果叔叔节点是红色，因为父节点是红色，显然祖父节点是黑色
					parentOf(node).color = BLACK;
					uncle.color = BLACK;
					parentOf(parentOf(node)).color = RED;
					node = parentOf(parentOf(node));
				}
				else {	
					if (parentOf(node).right == node) {				//情况二：叔叔节点是黑色，而Z是右孩子
						 node = parentOf(node);
						 leftRotate(node);
					}
					
					parentOf(node).color = BLACK;					//情况三：叔叔节点黑色，Z是左孩子
					parentOf(parentOf(node)).color = RED;
					rightRotate(parentOf(parentOf(node)));
				}
			}
			// 若父节点”是“祖父节点的右孩子
			else {
				Node<T> uncle = parentOf(parentOf(node)).left;
				if (isRed(uncle)) {
					parentOf(node).color = BLACK;
					uncle.color = BLACK;
					parentOf(parentOf(node)).color = RED;
					node = parentOf(parentOf(node));
				}
				else {
					if (parentOf(node).left == node) {
						 node = parentOf(node);
						 rightRotate(node);
					}
					parentOf(node).color = BLACK;
					parentOf(parentOf(node)).color = RED;
					leftRotate(parentOf(parentOf(node)));
				}
			}
		}
		/* 1.防止插入节点为根节点        
		 * 2.防止树深度较低时grandparent为root节点,从而在情况一uncle为red时，把root节点置为red 	 */
		if (node == root) {
			node.color = BLACK;
		}
	}
	/**
	 * 删除节点
	 * @param node : 待删除节点
	 * @return
	 */
	private Node<T> remove(Node<T> node){
		Node<T> replace = null;
		if (node.left == null || node.right == null) {
			replace = node;
		}else {
			replace = succussor(node);
		}
		Node<T> child = replace.left != null ? replace.left : replace.right; 
		
		if (child == null) {
			child = new Node<T>(null, null, null, replace.parent, BLACK);
		}
		child.parent = replace.parent;
		
		if (replace.parent == null) {
			root = child;
		}else {
			if(replace == replace.parent.left)
				replace.parent.left = child;
			else
				replace.parent.right = child;
		}
		
		if (replace != node) {
			node.key = replace.key;
		}
		if (isBlack(replace)) {
			removeFixUp(child);
		}
		return replace;
	}
	/**
	 * 删除某一数值
	 * @param key : 待删除值
	 */
	public void remove(T key){
		Node<T> node = null;
		if ((node = searchNode(root, key)) != null) {
			remove(node);
		}
	}
	
	private void removeFixUp(Node<T> node){
	
		while (node != this.root && node.color == BLACK) {
			// 若节点是父节点的左孩子
			if (node == parentOf(node).left) {
				Node<T> uncle = parentOf(node).right;		
				if (isRed(uncle)) {							//情况一：叔叔节点为红色
					parentOf(node).color = RED;
					uncle.color = BLACK;
					leftRotate(parentOf(node));
					uncle = parentOf(node).right;
				}
				//情况二：叔叔节点为黑色，且有两个黑孩子
				if (isBlack(uncle.left) && isBlack(uncle.right)) {
					uncle.color = RED;
					node = parentOf(node);
				}else {										
					if (isBlack(uncle.right)) {				//情况三：叔叔节点的左孩子为红色，右孩子为黑色
						uncle.left.color = BLACK;
						uncle.color = RED;
						rightRotate(uncle);
						uncle = parentOf(node).right;
					}
					uncle.color = parentOf(node).color;		//情况四：叔叔节点的右孩子为红色
					parentOf(node).color = BLACK;
					uncle.right.color = BLACK;
					leftRotate(parentOf(node));
					node = root;
				}
			}else {
				Node<T> uncle = parentOf(node).left;
				if (isRed(uncle)) {
					parentOf(node).color = RED;
					uncle.color = BLACK;
					rightRotate(parentOf(node));
					uncle = parentOf(node).left;
				}
				if (isBlack(uncle.left) && isBlack(uncle.right)) {
					uncle.color = RED;
					node = parentOf(node);
				}else {
					if (isBlack(uncle.left)) {
						uncle.right.color = BLACK;
						uncle.color = RED;
						leftRotate(uncle);
						uncle = parentOf(node).left;
					}
					uncle.color = parentOf(node).color;
					parentOf(node).color = BLACK;
					uncle.left.color = BLACK;
					rightRotate(parentOf(node));
					node = root;
				}
			}
		}
		node.color = BLACK;
	}
	
	public static void main(String[] args){
		int[] arr = {10, 40, 30, 60, 90, 70, 20, 50, 80};
		RBTree<Integer> tree = new RBTree<Integer>();
		
		for(int i : arr){
			tree.insert(i);
		}
		tree.inOrder();
		tree.remove(20);
		System.out.println();
		tree.inOrder();
	}
}
```