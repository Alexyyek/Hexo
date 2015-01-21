title: JAVA初始化
date: 2015-01-03 21:13:55
categories: Java
tags: coding
description: Java采用构造器的概念（在创建对象时被自动调用的特殊方法），并额外提供垃圾回收器，来确保避免不安全的编程方式，本节主要讨论初始化的相关问题。
---
初始化和清理是涉及安全的两个问题，许多C程序的错误都源于忘记初始化变量，当使用完一个元素时，也很容易忘记清理它。C++引入了构造器的概念，这是一个在创建对象时被自动调用的特殊方法，Java中也采用了构造器，并额外提供了垃圾回收器，对于不再使用的内存资源，垃圾回收器能自动将其释放。

在Java中，通过提供构造器，类的设计者可确保每个对象都会得到初始化。创建对象时，如果类具有构造器，就会自动调用相应的构造器。

本文提纲源于Thingking in Java，但主体讲解内容在其上有更深入的探讨。我们首先明确一下this关键词在构造器中的作用，然后分析构造器的初始化流程。	

##this关键字
从本质上讲，`this`是一个指向对象的指针（引用）。通常写`this`的时候，都是指这个对象或当前对象。而在构造器中，如果为`this`添加参数列表，将产生对符合此参数列表的某个构造器的明确调用，从而可以直接调用其他构造器。

首先看一个[stackoverflow](http://stackoverflow.com/questions/3728062/what-is-the-meaning-of-this-in-java)上的例子：
```JAVA
public class MyThisTest {
		private int a;

		public MyThisTest() {
			this(42); // calls the other constructor
		}

		public MyThisTest(int a) {
			this.a = a; // assigns the value of the parameter a to the field of
						// the same name
		}

		public void frobnicate() {
			int a = 1;

			System.out.println(a); // refers to the local variable a
			System.out.println(this.a); // refers to the field a
			System.out.println(this); // refers to this entire object
		}

		public String toString() {
			return "MyThisTest a=" + a; // refers to the field a
		}
		
		MyThisTest increment(){
			a++;
			return this; // refer to the reference of class MyThisTest
		}
	}
```

从这个例子可以总结`this`的几种关键用法：
1. 区分同名变量：`this`关键字是类内部当中对自己的一个引用，可以方便类中方法访问自己的属性。成员变量与方法内部的变量重名时，希望在方法内部调用成员变量，如`this.a = a`
2. 返回类自身的引用，在`increment()`方法中通过`this`关键字返回了对当前对象的引用
3. 在当前类的一个构造函数当中调用另一个构造函数，在无参构造器中通过`this(42)`调用构造器`MyThisTest(int a)`
<br/>
```java
class Flower {
		int petals = 0;
		String s = "initial value";

		Flower() {
			this("hello", 5);
		}

		Flower(int petals) {
			this.petals = petals;
		}

		Flower(String s) {
			this.s = s;
		}

		Flower(String s, int petals) {
			this(petals);
			/* Error : this(s);
			 * Constructor call must be the first statement in a constructor */
			this.s = s;			
		}

		void print() {
			/* Error : this(s);
			 * can not call the constructor in any method other than a constructor. */
			System.out.println(this.petals + " " + s);
		}
	}   
```
上面的例子列出了一些要注意的使用情况：
 1. 在构造方法中调用另一个构造方法，调用动作必须置于最起始的位置。
 2. 在一个构造方法内只能调用一个构造方法。
 3. 不能在构造方法以外的任何方法内调用构造方法。 


##构造器初始化

###初始化顺序
1. 变量定义的先后顺序决定了初始化的顺序
2. 但是不论变量定义在什么位置，都会在任何方法（包括构造器）被调用之前得到初始化

```java
class House{
	Window window1 = new Window(1);
	House(){
		System.out.println("In House");
		window3 = new Window(33);
	}
	Window window2 = new Window(2);
	void f(){System.out.println("f()");}
	Window window3 = new Window(3);
}
public class ExplicitStatic {
	public static void main(String[] args){
		House h = new House();
		h.f();
	}
}
```

在House类中，故意将几个Window对象的定义散布到各处，从而证明他们全部会在**调用构造函器或其他方法之前**得到初始化。此外window3在构造器内再次被初始化，第一次引用的对象将被丢弃，作为垃圾回收。

```java
Windows:1
Windows:2
Windows:3
In House
Windows:33
f()
```
<br/>
***
###静态数据初始化
无论创建多少个对象，静态数据都只占**一份存储区域**。静态数据初始化遵循以下原则：

* 初始化的顺序是先静态对象，而后是非静态对象。
* 静态数据成员的初始化只有在必要时才会进行。
    1. 创建类的对象             
    2. 引用类的静态数据成员

```java
class Window{
	Window(int marker){
		System.out.println("Window:" + marker);
	}
	void f1(int marker){
		System.out.println("f1():" + marker);
	}
}

class House{
	static Window window1 = new Window(1);
	House(){
		System.out.println("Table():");
		window2.f1(1);
	}
	void f2(int marker){
		System.out.println("f2():" + marker);
	}
	static Window window2 = new Window(2);
}

class People{
	Window window3 = new Window(3);
	static Window window4 = new Window(4);
	People(){
		System.out.println("People():");
		window4.f1(2);
	}
	void f3(int marker){
		System.out.println("f3():" + marker);
	}
	static Window window5 = new Window(5);
}
public class ExplicitStatic {
	static House house = new House();
	static People people = new People();
	public static void main(String[] args){
		System.out.println("Creating new House in main");
		new People();
		System.out.println("Creating new House in main");
		new People();
		house.f2(1);
		people.f3(1);
	}
}
```

1. 要加载main()，必须加载ExplicitStatic类，然后其静态域house和people被初始化，这导致他们对应的类也被加载。类House内先初始化静态数据成员window1和window2，然后初始化构造器，调用类Window的f1()方法；类People内同理，注意先初始化静态成员，后非静态成员，所以顺序是4,5,3.
2. main函数中生成新的People对象，由于之前类People中静态成员加载过，故不再加载。

输出结果：

```java
Window:1
Window:2
Table():
f1():1
Window:4
Window:5
Window:3
People():
f1():2
Creating new House in main
Window:3
People():
f1():2
Creating new House in main
Window:3
People():
f1():2
f2():1
f3():1
```
    
<br/>
***
###显示的静态初始化
Java允许将多个静态初始化动作组织成一个特殊的“静态子句”（静态块）。与其他静态初始化动作一样，仅执行一次。
```java
public class ExplicitStatic {
    //  static Cups cup3 = new Cups();
	public static void main(String[] args){
		Cups.cup1.f(99);
	}
}

class Cup{
	Cup(int marker){
		System.out.println("Cup:" + marker);
	}
	void f(int marker){
		System.out.println("f:" + marker);
	}
}

class Cups{
	static Cup cup1;
	static Cup cup2;
	static{                 //静态块
		cup1 = new Cup(1);
		cup2 = new Cup(2);
	}
	Cups(){
		System.out.println("Cups");
	}
}
```
程序访问Cups的静态成员cup1，类Cups初始化，首先初始化静态数据成员cup1、cup2，**由于这里并没有生成类Cups的对象**，所以类Cups的无参构造函数并不执行。
输出结果：
```java
Cup:1
Cup:2
f:99
```
如果把main方法上面的cup3的注释去掉，那么由于生成了Cups类的对象，此时输出结果变成：
```java
Cup:1
Cup:2
Cups
f:99
```
**注意**：如果类Cups里包含了非静态数据成员`Cup cup3 = new Cup();`，在执行main程序访问类Cups静态成员cup1时`Cups.cup1.f(99);`，也只会初始化静态成员cup1和cup2，输出结果不变。
<br/>
***

###非静态实例初始化
非静态初始化与静态初始化子句一模一样，只不过少了`static`关键字。这种语法对于支持“匿名内部类”的初始化时必须的，但是它也使得你可以保证无论调用了哪个显示构造器，某些操作都会发生。同时它不再具备静态对象初始化的优先级，所以可能先初始化构造器，后执行实例初始化。

<br/>


##数组初始化
Thinking in Java中数组初始化的问题讲的并不透彻，现在我们来深入一下对Java数组的认识。

###理解数组
数组也是一种数据类型，本身就是一种引用类型，我们从它的初始化方法，通过关键字new去完成定义及初始化就可以知道。

数组的长度是不变的，一旦数组完成初始化后，它的长度就固定下来了，在内存中占有的空间也就固定了，即使里面的数据被清空了，占有的空间还是保留下来了，依然是属于数组的，当然长度依旧是不变的。

数组里边存放的数据类型要一致，可以基本数据类型，也可以是引用数据类型，但是唯一的标准就是相同的类型。在Java中，类与类是支持继承关系的，所以就可能造成数组里面可以存在多中数据类型的假象：

```java
class Animal {
    public String name;
   
    public Animal(String name) {
        this.name = name;
    }
}
class Cat extends Animal {
    public Cat(String name) {
        super(name);
    }
}
class Dog extends Animal {
    public Dog(String name) {
        super(name);
    }
}
public class ArrayDemo {
    public static void main(String[] args) {
        Animal[] animals = new Animal[2];
        Cat cat = new Cat("little cat");
        Dog dog = new Dog("little dog");
        animals[0] = cat;
        animals[1] = dog;
        System.out.println(animals[0].name);
        System.out.println(animals[1].name);
    }
}
```
这样看上去，好像数组里面存放了Cat类型和Dog类型，但是实际上他们都是Animal类型，数组里面都是相同的类型，请大家不要搞混淆了。

###定义数组和初始化数组
定义数组的语法格式：
```java
type[] arrayName;
type arrayName[];
```
无论是Thinking in Java还是我个人都推荐第一种方式，直观的告诉别人定义了一个type类型数组，而不是type类型变量。

初始化数组：

$1$. 静态初始化
```java
arrayName = new type[]{element1,element2, element3......};
```
实例：
```java
int [] int_arr_1 = {1,2,3};
Integer arr = {new Integer(1), new Integer(2), 3}; // JAVA基本类型和包装类探讨过的*自动装箱功能*
```

$2$. 动态初始化
```java
arrayName = new type[length];
```

当然不同的数据类型，系统分配的初始值不同。
<br/>

| 基本类型        | 初始值   |  
| :--------:   | :-----:  | 
| 整数类型（byte、short、int、long）     | 0 |  
| 浮点类型（float、double）        |   0.0   |  
| 字符类型（char）        |    '\u0000'（代表空格）    |  
| 布尔类型（boolean）        |    false    |  
| 引用类型（类、接口、数组）       |    null    | 

###内存中的数组
们让一个引用变量指向另外一个实际的数组的时候，可能产生数组长度可变的假象，大家来看看一个例子：
```java
public class ArrayDemo3 {

    public static void main(String[] args) {
        
        int[] a = new int[]{1, 2};
        int[] b = new int[4];
        System.out.println("length of b:" + b.length);
        
        b = a;
        System.out.println("length of b:" + b.length);
    }
}
```
结果：b数组的长度好像发生了变化，但是实际上不是这样的，我们来分析一下内存中的变化：
```java
length of b:4
length of b:2
```
如果你不能明确的理解Java中对象和引用以及参数传递的本质，建议你首先看一下[之前的博客](http://alexyyek.github.io/2014/12/29/Reference/)。

上面的程序中：
1. 在堆中创建数组对象{1,2}，占用两个长度，并在堆栈中创建引用a指向堆中数组{1,2}的首地址。
2. 在堆中开辟长度为4的数组空间，初始化为{0,0,0,0}；然后在堆栈中创建引用b指向堆中数组{0,0,0,0}的首地址。
3. 将引用a指向的地址副本传递给引用b，故引用b不再指向{0,0,0,0}，而是指向引用a所指的{1,2}

**总结**：我们所说的**数组的长度不变是针对堆内存中真正数组的长度**，引用变量是可以改变指向的，指到哪里肯定就显示指到的数组的长度了，但是真正的长度是不曾改变的。

##可变参数列表

###基本使用
当调用方法时，方法的参数个数或类型未知时，称其为可变参数列表。可变长参数列表是Java 5中的一个新特性。如果方法需要传入多个同类型参数的话，这个功能就非常有用。

```java
public static void printArrays(Object[] args ){
        for (Object arg : args)
            System.out.println( arg );
}
public static void main(String[] args){
		printArrays(new Object[]{new Integer(25),new Float(3.1415),new Character('a'), new String("shit")});
		printArrays(new Object[]{new A(),new A(),new A()});
}
```
由于所有的类都直接或间接继承于Object类，所以可以创建以Object数组为参数的方法并如上调用。然而Java 5增加了对可变参数的支持，现在你可以用下面的方法更简洁的调用。

```java
class A{}
public static void printArrays(Object... args ){
        for (Object arg : args)
            System.out.println( arg );
}
public static void main(String[] args){
		printArrays(3,3.547,3.456f,"Alex",new A(),new Date(),"string"+50);
		printArrays((Object[])new Integer[]{1,2,32,4});
}
```
注意main程序最后一行，Integer数组传递给了printArrays()，因为它已经是一个数组，所以不再进行转换。
因此，如果你有一组事物，可以把它们当做列表传递；而如果你已经有了一个数组，该方法可以把其当做可变参数列表来接受。

###使用规范
$1$. 一个方法只能有一个可变长参数，并且这个可变长参数必须是该方法的最后一个参数   

正确体位：
```java
static void printArrays(String arg, String...args){
		System.out.println(arg);
		for(Object obj : args){
			System.out.print(obj + "\t");
		}
	}
```

$2$. 你应该总是只在重载方法的一个版本上使用可变参数列表，或者压根就不用丫。

```java
static void printArrays(String... args){
		for(Object obj : args){
			System.out.print(obj + "\t");
		}
		System.out.println();
	}
	static void printArrays(String arg, String...args){
		System.out.println(arg);
		for(Object obj : args){
			System.out.print(obj + "\t");
		}
	}
	public static void main(String[] args){
	/*  Compile Error
	 *  printArrays("what","the","fuck");*/
}
```
此时编译器无法知道你到底是要调用哪个方法，因为两个方法看起来都可以。除非你为这两个方法都添加一个非可变参数。

$3$. 别让null值和空值威胁到变长方法

```java
static void func(Integer...args){}
	
static void func(String...args){}

public static void main(String[] args){
            func(0);
		    func("Alex");
		    func(); //Compile Error
		    
		    String word = null;
	    	func(word); //That's OK
}
```
在每一种情况，编译器都会使用[自动包装](http://alexyyek.github.io/2014/12/29/wrapperClass/)机制来匹配重载的方法，然后调用最明确的匹配方法。但是调用`f()`时，编译器不知道应该调用哪个方法，可以通过第10、11行的修改编译通过。

最后来个简单的测试，看看你是否还清醒：
```java
	 public static void f(String arg, String... args) {
	        for (int i = 0; i < args.length; i++) {
	            System.out.println(args[i]);
	        }
	    }
	 public static void main(String[] args){   
	            f("");
                f("aaa");
                f("aaa", "bbb");
	}
```

这是看到的一个[英文版讲解](http://java-performance.info/java-varargs-performance-issues/)
