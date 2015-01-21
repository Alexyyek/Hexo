title: Java访问权限控制
date: 2015-01-12 22:47:08
categories: Java
tags: coding
description: 把数据和方法包装进类中，以及具体实现的隐藏，常共同被称作封装。
---

> 《Thinking in Java》：把数据和方法包装进类中，以及具体实现的隐藏，常共同被称作封装。

在Thinking in Java这本书中，并没有用封装这个词来为这一章命名，而是选择访问控制权这个更接近本质的词。

访问控制权主要解决两个问题：
1. 设定客户端程序猿可以使用和不可用使用的界限。
利用这个界限可以在结构中建立自己的内部机制，而不用担心客户端程序猿将内部机制当作可以调用的接口。
2. 接口和具体实现的分离。
客户端程序员只是调用你的接口，所以你可以任意修改所有访问权限控制不是public的代码，而不会破坏客户端的代码。

在Java中，提供了四种访问权限控制：default（包访问权限），public，private 以及 protected，具体权限范围相信大家都比较熟悉，下面主要分享一些更深入的观点。

##类访问权限

类中的访问权限只包含两种：public和包访问权限，为什么没有private和protect呢？

public就是说这个类可以随便外界使用，包访问权限就是辅助类的权限了，只允许同一个包内的别的类使用该辅助类。  

如果是要添加private控制一个类的权限，那么想表达什么意思呢？可能会说，我在辅助类的前面添加上一个Private，就想控制：仅仅允许同一个java文件内的主类使用这个类。有没有这个需求呢？肯定有，因为如果这个辅助类仅仅属于这一个主类，那么我就可以方便修改了，而不用管别的类是否在使用。 但是Java为什么没有设定这个private权限？ 

答案是：Java提供了一个更容易理解的方式－－内部类实现了这个需求。用private修饰一个辅助类，其实是不符合private的意思的。所以用内部类来表示显的更自然。

那类为什么没有protected呢？其实可以发现并没有这个需求。 别的类要想继承我这个类，必须先能看到我这个类。如果你看不到我这个类，那还继承个毛线。

假设这样一种情况：不希望其他任何人对该类拥有访问权限，可以把所有的构造器都指定为private，从而阻止任何人创建该类的对象。该如何的实现呢？看下面一段代码：

```java
class Alex1 {
	private Alex1() {
	};

	public static Alex1 makeAlex1() {
		return new Alex1();
	}
}

class Alex2 {
	private Alex2() {
	};

	private static Alex2 alex = new Alex2();

	public static Alex2 access() {
		return alex;
	}

	public void f() {
	};
}

public class Geek {
	void testPrivate() {
		/* Error : The constructor Alex1() is not visible 
		 * Alex1 alex1 = new Alex1();
		 */
	}

	void testStatic() {
		Alex1 alex1 = Alex1.makeAlex1();
	}

	void testSingleton() {
		Alex2.access().f();
	}
}
```

Alex1类和Alex2类展示了如何通过将所有的构造器指定为private来阻止直接创建某个类的实例。

但是现在别人该如何使用这个类呢？

上面的例子给出了两种选择：
1. 在Alex1中，创建一个static方法，该方法创建一个新的Alex1对象并返回一个对它的引用。如果想要在返回引用之前在Alex1上做一些额外的工作，或是如果想要记录到底创建了多少个Alex1对象，这种做法将会大有裨益。
2. Alex2用到了所谓的设计模式，这种特定的模式被称为singleton（单例），这是因为你始终只能创建它的一个对象。Alex2类的对象时作为Alex2的一个static private成员而创建的，所以有且仅有一个，而且除非调用public方法access()，否则是无法访问到它的。



##域/变量访问权限

访问权限请看下面的表格，具体[事例](http://www.cnblogs.com/dolphin0520/p/3734915.html)不再赘述。
<br/>

| 范围        | private   | default  | protect | public |
| :--------:   | :-----:  | :-----:  | :-----:  | :-----:  | 
| 在同一包的同一类 | √ |  √ |√ |√ |
| 同一包的不同类  ||√ |√ |√ |  
| 不同包的子类 |   | | √ |√ |  
| 不同包的非子类 |   | | |√ | 