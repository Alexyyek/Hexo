title: 对象和引用
date: 2014-12-29 21:43:15
categories: Java
tags: coding
description: When you're passing primitives into a method，you get a distinct copy of the primitive. When you're passing a reference into a method, you get a copy of the reference.
---

Java中对象和引用的东西之前看了不少，但一方面因为网上知识过于繁杂，不能很好的梳理，另一方面不同的博客讲解专业程度不一样，可能会导致一定的误解。此篇博客争取以最精炼，最具条理的方式理清其中的难点和疑点。

##对象和引用

###对象

>《Java编程思想》：按照通俗的说法，每个对象都是某个类（class）的一个实例（instance），这里，‘类’就是‘类型’的同义词。

**理解本质**：对象是类的实例。例如[Alex](http://ww2.sinaimg.cn/bmiddle/53539f83jw1emh2fiopjtj20u01hc4qp.jpg)是人类，Alex是对象，就是“人类”的实例。对象一般存储在堆中。

###引用

>《Java编程思想》： 每种编程语言都有自己的数据处理方式。有些时候，程序员必须注意将要处理的数据是什么类型。你是直接操纵元素，还是用某种基于特殊语法的间接表示（例如C/C++里的指针）来操作对象。所有这些在 Java 里都得到了简化，一切都被视为对象。因此，我们可采用一种统一的语法。尽管将一切都“看作”对象，但操纵的标识符实际是指向一个对象的“引用”（reference）。

**理解本质**：引用是指向对象的标识符。可以理解对象是一个个气球，我们通过引用这条绳子和它链接。引用存储在速度更快的堆栈中。 

###实例讲解

首先看下面的代码：
```java
public class People {
	String name;
	String sex;
	int age;
	
	//构造函数
	public People(){
	
	}
	
	public People(String name, String sex, int age){
		this.name = name;
		this.sex = sex;
		this.age = age;
	}
}
```

people是一个类，包含属性 name、sex、age，有了这个类，我们创建对象：
```java
    People one = new People();
```
这条语句的动作称为创建一个对象，它包含了四个动作:
1. 右边的new People，是以Pelple类为模板，在堆空间中创建一个People类对象
2. 结尾的()，在对象创建后，立刻调用People类的构造函数，对刚生成的对象初始化。这个例子中包含了构造函数，如果没写，java会自动帮你补上。
3. 左侧的People one创建了一个People类的引用变量。即one是指向People对象的引用。
4. “=”操作符使对象引用**指向**刚创建的People对象。注意是指向，不是赋值。

可以将上面的话拆成两句，效果一样：
```java
    People one;
    one = new People();
```

如果只执行了第一条语句，还没执行第二条，此时创建的引用变量one还没指向任何一个对象，它的值是null。引用变量可以指向某个对象，或者为null。然后看下面这段：

```java
    People two;
    two = one;
```

首先又创建了一个引用two，此时two指向null，第二句将one指向的地址传给two，发生了复制行为，注意这里复制的是`对象引用`，此时two也指向one所指的对象，具体如何传递后面会讲。

```java
    two = new People();
```

这句将two指向了一个新的对象，所以可以分析如下两条性质：

1. 一个引用可以指向0个或1个对象（一根绳子可以不系汽球，也可以系一个汽球）；
2. 一个对象可以有N个引用指向它（可以有N条绳子系住一个汽球）。 

如果发生这样的情况：
```java
    People one = new People("张三");
    People two = new People("李四");
    one = two;
```

按上面的逻辑，引用one先指向对象“张三”，然后又指向了引用two指向的“李四”，所以最后one和two都指向对象“李四”，问题是之前的对象“张三”呢？还在堆里么？

多数书里说，它被Java的垃圾回收机制回收了。这不确切。

> 正确地说，它已成为垃圾回收机制的处理对象。至于什么时候真正被回收，那要看垃圾回收机制的心情了。 

<br/>
***

###参数传递

参数传递是个大坑，面试时常常会被问及，但其实并不难，看透本质即可。

> 《thinking in Java》：When you're passing primitives into a method，you get a distinct copy of the primitive. When you're passing a reference into a method, you get a copy of the reference.

**本质**：不管JAVA参数类型是什么，一律传递参数的**`副本`**

<br/>
####例一
```java
public static void main(String[] args){
		boolean test = true;
		System.out.println("before test() :" + test);
		test(test);
		System.out.println("after test() :" + test);
	}
	
	public static void test(boolean test){
		test = !test;
		System.out.println("In test() :" + test);
	}
```

**分析**：首先声明一个基本类型为boolean的对象true，并用test指向它，在函数test内，引用test将值的副本传递进去，所以原本的test不改变，在test内输出false，而after test仍然还是true。
**注意**：基本类型传值的副本，对象变量传引用的副本
<br/>
####例二
```java
public static void main(String[] args){
		StringBuffer string = new StringBuffer("Hello");
		test(string);
		System.out.println(string);
	}
	
	public static void test(StringBuffer str){
		str.append(",world!");
	}
```
**分析**：首先声明一个stringbuffer类型的对象型变量，string指向对象Hello，在test内将string的引用副本传给str，因此string和str都在栈内指向同一个堆地址，只不过所处栈地址不同。test内改变了str指向的对象，因此string指向的对象也改变了，变成Hello,world!.
<br/>
####例三
```java
class value{
	public int i = 15;
}

public class dataType {
	public static void main(String[] args){
		dataType type = new dataType();
		type.first();
	}
	
	public void first(){
		int i = 5;
		value v = new value();
		v.i = 25;
		second(v, i);
		System.out.println(v.i);
	}
	
	public void second(value v, int i){
		i = 0;
		v.i = 20;
		value val = new value();
		v = val;
		System.out.println(v.i + " " + i);
	}
}

```
1. 在first内，首先程序在栈内存中开辟了一块地址编号为AD9500内存空间，用于存放`v`的引用地址，里边放的值是堆内存中的一个地址，示例中的值为BE2500
<center> ![](http://alexyoung.qiniudn.com/QQ图片20141229225902.png) <center/>

2. 调用函数second，程序在栈内开辟地址为AD9600内存空间存放v的副本，v的副本同样指向堆地址为BE2500的空间，然后将v的副本传入second，并且在second内，将v的副本所指对象`i=25`改为`i=20`
<center> ![](http://alexyoung.qiniudn.com/QQ图片20141229225924.png) <center/>

3. 在second内，程序新建一个对象放在地址为BE2600的堆内，并用新的引用val（栈中地址为AD9700）指向它，所以second中输出结果为：15 0
<center> ![](http://alexyoung.qiniudn.com/QQ图片20141229225934.png) <center/>

4. 但原v并未改变，改变的只是它传入second的副本，所以在first中仍然输出`i=20`
<br/>
***

**PS**：在JAVA里，“=”不能被看成是一个赋值语句，它不是在把一个对象赋给另外一个对象，它的执行过程实质上是将右边对象的地址传给了左边的引用，使得左边的引用指向了右边的对象。JAVA表面上看起来没有指针，但它的引用其实质就是一个指针，引用里面存放的并不是对象，而是该对象的地址，使得该引用指向了对象。在JAVA里，“=”语句不应该被翻译成赋值语句，因为它所执行的确实不是一个赋值的过程，而是一个传地址的过程，被译成赋值语句会造成很多误解，译得不准确。 