title: Java复用类
date: 2015-01-14 20:48:39
categories: Java
tags: coding
description: One of the most compelling features about Java is code reuse. But to be revolutionary, you’ve got to be able to do a lot more than copy code and change it.
---

多数时候我们不希望重新编写代码，而是希望使用已有的代码并加以更改或添加，类的复用为我们提供了这种机制，这也是面向对象语言重要的特点之一。复用类的主要方法有两种，组合与继承。继承和组合都是一种随思想渗透而下的编码方式，其根本目的都是为了复用类，减少重复代码。要实现一个类的复用，可以分为组合语法和继承语法。

##组合语法
> 《Thinking in Java》：在新的类中产生现有类的对象。

组合就是通过将一个对象置于一个新类中，将其作为新类的成员变量，组成类的一部分。由于新的类是由现有类的对象组成，所以称此方法为组合。

来看一段代码：

```java
class WaterSource{
	private String s;
	WaterSource(){
		System.out.println("WaterSource()");
		s = "constructed";
	}
	public String toString(){
		return s;
	}
}

public class SprinklerSystem {
	private String name = "Alex";//Initializing at point of definition
	private String sex;
	private WaterSource wSource;
	private double d;
	public  SprinklerSystem(){
		System.out.println("In Sprinkler");
		d = 1.0;    //Initailizing in the constructor of class
		wSource = new WaterSource();
	}
	public String toString(){
		if (sex == null) { sex = "male"; }//Delayed initialization
		return 
			"string" + "=" + name + " " +
			"sex" + "=" + sex + " " +
			"double" + "=" + d + " " +
			"WaterSource" + "=" + wSource;
	}
	
	public static void main(String[] args){
		SprinklerSystem sprinklers = new SprinklerSystem();
		System.out.println(sprinklers);
	}
}
```
我们可以看到，类SprinklerSystem中直接产生现有的类WaterSource，以此可以使用关于WaterSource的各种方法。

但注意这里两个类中都有一个很特殊的方法：toString()。

当编译器需要一个String而你只有一个对象wSource时，由于只能将两个String对象相加，因此编译器会调用toString方法，将wSource转化为一个String。当然如果没有写toString方法，这里会打印对象的地址。

打印输出：
```java
In Sprinkler
WaterSource()
string=Alex sex=male double=1.0 WaterSource=constructed
```

此外，编译器并不是简单的为每一个引用都创建默认对象，避免不必要的负担。如果想初始化这些引用，可以在例子中的这些位置进行：

1. 定义对象的地方
2. 在类的构造器中
3. 在正要使用这些对象之前
4. 使用实例初始化

由此可以看出：组合是在新类产生现有类的对象，组合出功能给更强的新类。

**如果将继承比作Is-A的关系（什么是什么..），组合则是Has-A(什么有什么)的关系。**

##继承 

> 《Thinking in Java》：通过现有类的类型创建新类，采用现有类的形式并在其中添加新代码。

当创建一个类时，总是在继承，因此除非已明确指出要从其他类中继承，否则就是隐式的从Java的标准根类Object进行继承。继承通过`extends`关键字自动获得基类中的所有域和方法。在看下面的例子前建议先理解`super`关键字的 → [用法](http://docs.oracle.com/javase/tutorial/java/IandI/super.html)。

```java
class Base {
	static int s1 = prt("s1 initialized.", 11);
	int i1 = prt("i1 initialized.", 12);

	Base() {
		System.out.println("Base construcor");
		System.out.println("s1 = " + s1 + " ,i1 = " + i1);
		draw();// （d）
	}

	void draw() {
		System.out.println("base.draw：s1 = " + s1 + " ,i1 = " + i1);
	}

	static int prt(String s, int num) {
		System.out.println(s);
		return num;
	}
}

class Cleanser extends Base {
	private String s = "Cleaner";
	static int s2 = prt("s2 initialized.", 21);
	int i2 = prt("i2 initialized.", 22);

	public Cleanser() {
		System.out.println("Cleanser constructor");
	}

	public Cleanser(int i) {
		System.out.println("Cleanser constructor init i:" + i);
		System.out.println("s2 = " + s2 + " ,i2 = " + i2);
		draw();
	}

	public void append(String a) {
		s += a;
	}

	public void dilute() {
		append(" dilute()");
	}

	public void scrub() {
		append(" scrub()");
	}

	void draw(){
        System.out.println("Cleanser.draw：s2 = " + s2 + " ,i2 = " + i2);
    }
	
	public String toString() {
		return s;
	}

	public static void main(String[] args) {
		Cleanser x = new Cleanser();
		x.dilute();
		x.scrub();
		System.out.println(x);
	}
}

public class Detergent extends Cleanser {
	static int s3 = prt("s3 initialized.", 31);
    int i3 = prt("i3 initialized", 31);
    
	public Detergent() {
		super(1024);
		System.out.println("Detergent construcor");
		System.out.println("s3 = " + s3 + " ,i3 = " + i3);
		draw();
	}

	public void scrub() {
		append(" Detergent.scrub()");
		super.scrub();
	}

	public void foam() {
		append(" foam()");
	}
	
	void draw(){
        System.out.println("Detergent.draw：s3 = " + s3 + " ,i3 = " + i3);
    }

	public static void main(String[] args) {
		Detergent detergent = new Detergent();
		detergent.dilute();
		detergent.scrub();
		detergent.foam();
		System.out.println(detergent);
		System.out.println("Test base class:");
		Cleanser.main(args);
	}
}
```

这个例子糅合了继承中许多重要的点，我们首先分析一下基本框架，程序包含三个类，Base类、继承Base的Cleanser类，继承Cleanser的Detergent类。类中包含了静态成员，非静态成员，以及类内部的方法。下面来梳理程序流程。

### 继承的初始化
$1$. 装载Detergent.class文件
$2$. 发现Detergent有关键字extends，装载Cleanser.class文件
$3$. 发现Cleanser有关键字extends，装载Base.class文件
$4$. 初始化Base class中的静态成员。初始化Cleanser class中的静态成员。初始化Detergent class中的静态成员。**如果Detergent的main函数为空，到这里已经初始化完毕。**输出：
```java
s1 initialized.
s2 initialized.
s3 initialized.
```
> 《Thinking in Java》：继承并不只是复制基类的接口。当创建了一个导出类的对象时，该对象包含了一个基类的子对象。

$5$. 为detergent对象分配存储空间，并把存储空间初始化为0。在Detergent的构造器中调用super(1024)，试图产生一个Cleanser class实例。
$6$. 为Cleanser对象分配存储空间。由于Cleanser类继承自Base类，会在Cleanser类的构造器中调用Base的无参构造函数。
$7$. 产生一个Base class实例。先初始化成员变量，再调用构造函数。回到Cleanser类执行同样步骤，Detergent类亦如此。

至创建detergent对象结束后，初始化的输出结果如下，其中要注意几点：

* 继承中的构建过程是“向外”扩散的，先初始化基类，后初始化导出类。
* 注意Detergent类构造函数中`super`的使用，super用于导出类的构造函数，可以调用基类的构造方法。如果基类**没有默认构造器或者带有参数**，则需要显式构造，构造方法则是上面提到过的super，当然super要放在构造函数内第一行。
* 注意Base类和Cleanser类构造函数中的draw()方法，并没有按想象中直接调用类内部的draw()，而是统一调用了子类Detergent中的draw()方法。

```java
i1 initialized.
Base construcor
s1 = 11 ,i1 = 12
Detergent.draw：s3 = 31 ,i3 = 0
i2 initialized.
Cleanser constructor init i:1111
s2 = 21 ,i2 = 22
Detergent.draw：s3 = 31 ,i3 = 0
i3 initialized
Detergent construcor
s3 = 31 ,i3 = 31
Detergent.draw：s3 = 31 ,i3 = 31
```


###继承语法
在创建完detergent对象后，detergent对象开始调用不同方法，在Cleanser的接口中有一组方法：append()，dilute()，scrub()，draw()和toString()。由于Detergent是由关键字`extends`从Cleanser导出的，所以它可以再其接口中自动获得这些方法，因此可以将继承视作是对类的复用。

在Detergent中，我们复用了部分Cleanser中的方法，但改写了其中的scrub()，如果需要在新版本中调用从基类继承而来的方法，可以利用`super`关键字，正如开头链接中`super`的第一种使用方法：Accessing Superclass Members。

当然在继承中并不一定非要使用基类的方法，可以在导出类中添加新的方法，如Detergent类中新增的foam()方法。后续输出结果如下：

```java
Cleaner dilute() Detergent.scrub() scrub() foam()
Test base class:
i1 initialized.
Base constructor
s1 = 11 ,i1 = 12
Cleanser.draw：s2 = 21 ,i2 = 0
i2 initialized.
Cleanser constructor
Cleaner dilute() scrub()
```

###清理
清理过程是先执行导出类的清理代码，再执行基类的清除代码。这就像一棵大树，生长的时候要从根向叶子生长，但剪除的时候要从叶子向根剪除。如果一下子去掉了根，那么叶子你也就都找不到了。这种清理与以前说过的finalize方法不同，如果要执行这种清理工作，最好还是自己编写清理代码。

```java
class Shape {
    Shape(int i) { print("Shape constructor"); }
    void dispose() { print("Shape dispose"); }
}
class Circle extends Shape {
    Circle(int i) {
        super(i);
        print("Drawing Circle");
    }
    void dispose() {
        print("Erasing Circle");
        super.dispose();
    }
} 
```

###名称屏蔽
如果Java的基类拥有某个已被多次重载的方法名称，那么在导出类中重新定义该方法名称并不会屏蔽其在基类中的任何版本（与C++不同）。Java SE5新增了`@Override`注释，它并不是关键字，但是可以当做关键字使用。当你想要**`覆写`**某个方法时，可以选择添加这个注解，防止你不留心重载了该方法。通过下面表格区分重载和覆写：
<br/>

| 范围        | 重载（Overload）   | 覆写（Override）  | 
| :--------:   | :-----:  | :-----:  | 
| 范围 | 类内定义方法的不同版本 |  子类为满足需要重复定义某个方法的不同实现 |
| 方法名 | 相同 |  相同 |
|  参数列表 | 必须不同 |  相同 |
| 返回值类型 | 可以不同 |  相同 |

##组合与继承的选择
组合与继承都允许在新的类中放置子对象，组合是显示的这样做，而继承是隐式地做。

组合技术通常用于想在新类中使用现有类的功能，而非它的接口。即在新类中嵌入某个对象，让其实现所需要的功能，但新类的用户只能看到为新类定义的接口，而非所有嵌入对象的接口。为取得此效果，**需要在新类中嵌入一个现有类的private对象**。

继承常用于你在使用一个通用类时，为了某种特殊需求而将其特殊化，开发一个它的特殊版本。就像开头说的：**如果将继承比作Is-A的关系（什么是什么..），组合则是Has-A(什么有什么)的关系。**

##Protected关键字

讲完了继承，关键字protected也终于有了意义。在实际项目中，经常会想要境某些事物尽可能**对这个世界隐藏起来，但仍然允许导出类或者其他位于同一个包内的类来说，它却是可以访问的**。（protected也提供了包内访问权限）

尽管可以创建protected域，但是最好的方式还是将域保持为private；你应当一直保留**更改底层实现的权利。**然后通过protected方法来控制类的继承者的访问权限。

> The protected modifier specifies that the member can only be accessed within its own package (as with package-private) and, in addition, by a subclass of its class in another package.


##向上转型
> 《Thinking in Java》：The most important aspect of **inheritance** is not that it provides methods for the new class.It’s the **relationship** expressed between the new class and the base class. 

Wind类继承了基类Instrument，并创建了Wind对象，但是Instrument类仍可以使用Wind对象，程序代码可以对Instrument和它所有的导出类起作用。由于继承可以确保基类中所有的方法在导出类中也同样有效，所以能够向基类发送的所有信息同样也可以向导出类发送。这种将Wind引用转换为Instrument引用的动作，称为向上转型。
```java
class Instrument {
    public void play() {}
    static void tune(Instrument i) {
    i.play(); }
}
    public class Wind extends Instrument {
        public static void main(String[] args) {
        Wind flute = new Wind();
        Instrument.tune(flute); // Upcasting
    }
}  
```

由于向上转型是从一个较专用类型向较通用类型转换，所以总是很安全的。在向上转型中，类接口中唯一可能发生的事情是丢失方法，而不是获取它们。

```java
class Game{
	Game(int i){
		System.out.println("Game constructor");
	}
}

class BoardGame extends Game{
	BoardGame(int i){
		super(i);
		System.out.println("BoardGame constructor");
	}
}

public class Chess extends BoardGame{
	Chess(){
		super(250);
		System.out.println("Chess constructor");
	}
	
	public void f(){};
	
	public static void main(String[] args){
		Game game = new Chess();    //向上转型
		/* Error 
		 * The method f() is undefined for the type Game
		   game.f(); */
	}
}
```
如果运行Chess，输出的是Game constructor 还是三个类全部初始化？

答案是全部初始化，即创建了子类Chess对象。这是因为game实际上指向的是一个子类对象。当然，你不用担心，Java虚拟机会自动准确地识别出究竟该调用哪个具体的方法。不过，由于向上转型，game对象会遗失和父类不同的方法，例如f()。

向上转型可以简化程序代码，例如这里的Monitor类 → [栗子](http://blog.csdn.net/shanghui815/article/details/6088588)

##final关键字
为了设计或效率这两个截然不同的目的，我们可能会使用到final关键字。

###final data
相比于C/C++中的const关键字，Java使用final来声明数据。对于基本类型，final保证数值恒定不变；用于对象引用，则保证引用恒定不变，也就是说无法再把该引用指向另一个对象。通常声明一个常量：
```java
private static final int VALUE = 10;
```
定义为static强调只有一份，定义为final说明它是一个常量。

* final声明的变量，意味着无法将其引用对象再次指向另一个新的对象，而不是无法改变它的值。
* 带有恒等初始值的final static基本类型全用大写字母命名。
* final成员变量必须在声明的时候初始化，或在构造器中初始化。

```java
class Vincent{
	int i;
	public Vincent(int i){
		this.i = i;
	}
}

public class FinalData{
	
	private String id;
	private static Random random = new Random(25);
	FinalData(String id){
		this.id = id;
	}
	
	private final int first = 1024;
	private static final int SECOND = 2048;
	public static final int THIRD = 3072;
	
	private Vincent v1 = new Vincent(1024);
	private final Vincent v2 = new Vincent(2048);
	private static final Vincent V3 = new Vincent(3072);
	
	private final int rand = random.nextInt(100);
	private static final int RAND = random.nextInt(100); 
	
	public String toString(){
		return id + "\t" + rand + "\t" + RAND; 
	}
	public static void main(String[] args){
		FinalData fData = new FinalData("first trial");
		
		/*	Error: Can't change value
		 * 	fData.first++;
		 */
		
		fData.v1.i++;
		fData.v2.i++;
		
		fData.v1 = new Vincent(1); //withour final
		
		/* Error: Can't change reference 
		 * fData.v2 = new Value(2);
		 * fData.V3 = new Value(3);
		 */
		
		System.out.println(fData);	//first trial	28	81
		FinalData fData2 = new FinalData("second trial");
		System.out.println(fData2); //second trial	47	81
	}
}
```
1. 对final修饰的基本类型视为常量，不能再修改（如first、SECOND和THIRD）
2. final修饰的引用对象，可以改变值，如`fData.v2.i++;`，但不可修改指向对象。
3. 不能因为数据是final的就认为在编译时已知了它的值，如rand和RAND，用static修饰的RAND在最开始便初始化完毕，故两次值不变。

###final method
现在使用final方法的原因只有一个：把方法锁定，以防止任何继承类修改他的含义，想要确保继承中使方法行为**保持不变**，并且**不会覆盖**。

* 对于实例方法，final意味着在子类**不能重写**该方法，但**可以重载**。
```java
class Father{
    public final void f(){};
}
class Child extends Father{
    /* Error: Cannot override the final method from Father
     * public final void f(){};
     */
    public final void f(int i){};
}
```

* 对于静态方法final的含义为，子类中**不能隐藏（覆盖或重写）**这个方法。
```java
class Father{
    public static void f1(){};
    public static final void f2(){};
}

public class Child extends Father{
	/* Error: This instance method cannot override the static method from Father
	 * public void f(){System.out.println("Alex");}
	 */
	
    public void f1(int i){System.out.println("Alex");}
    
    /* Error: Cannot override the final method from Father
     * public static void f2(){};
     */
    
    public static final void f2(int i){};
}
```
静态方法是可以继承的，但是在子类中即使可以定义一个与父类方法签名相同的方法覆盖掉父类的方法，但是这并不是重写。重写应该在多态性上有所体现，但是覆盖父类静态方法并不会体现[多态性](http://segmentfault.com/blog/zhaobing/1190000000669897)。

###final classes
当将某个类整体定义为final时，就表明你不打算继承该类，而且也不允许别人这么做。换句话说，你对该类的设计用不需要做出任何变动，或者是出于安全考虑，你不希望它有子类。

由于final类禁止继承，所以final类中所有的方法都隐式指定为final的，因为无法覆盖他们。