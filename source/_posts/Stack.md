title: Min Stack
date: 2014-11-19 20:54:53
categories: Java
tags: [stack, leetcode]
---

##Min Stack
Question : Design a stack that supports push, pop, top, and retrieving the minimum element in constant time.
<!--more-->   

- push(x) -- Push element x onto stack.
- pop() -- Removes the element on top of the stack.
- top() -- Get the top element.
- getMin() -- Retrieve the minimum element in the stack.
<br/>

***
###Best solution

```java
public class MinStackBest {
	Stack<Long> stack;
	long min = 0;
	public MinStackBest() {
		stack = new Stack<Long>();
	}
	public void push(int x) {
		if (stack.isEmpty()) {
			stack.push(0L);
			min = x;
		} else {
			stack.push(x - min);
			if (x < min) {
				min = x;
			}
		}
	}
	public void pop() {
		if (stack.isEmpty()) {
			return;
		}
		long pop = stack.pop();
		if (pop < 0) {
			min = min - pop;
		}
	}
	public int top() {
		long top = stack.peek();
		if (top > 0) {
			return (int) (min + top);
		} else {
			return (int) min;
		}
	}
	public int getMin() {
		return (int) min;
	}
```

The thought is to store the gap between the current value and min value, while the only problem is int is 4 bit
so the Max gap = Integer.MAXVALUE - Integer.MINVALUE, and that value beyond int, so the author init the min value as long.

<br/>
###Another solution
```java
public class MinStack {
	
	static class Element{
		final int value;
		final int min;
		Element(int value, int min){
			this.value = value;
			this.min = min;
		}
	}
	
	Stack<Element>stack = new Stack<Element>();
	
	public void push(int x) {
		int min = (stack.isEmpty()) ? x : Math.min(x, stack.peek().min);
		stack.push(new Element(x, min));
    }

    public void pop() {
    	stack.pop();
    }

    public int top() {
    	return stack.peek().value;
    }

    public int getMin() {
        return stack.peek().min;
    }
}
```
<br/>
##Review Stack

Here's the source code, compose by one Constructor and five Method
```java
/**
	 * 构造函数
	 */
	public Stack() {
	}

	/**
	 * push函数：将元素存入栈顶
	 */
	public E push(E item) {
		// 将元素存入栈顶。
		// addElement()的实现在Vector.java中
		addElement(item);

		return item;
	}

	/**
	 * pop函数：返回栈顶元素，并将其从栈中删除
	 */
	public synchronized E pop() {
		E obj;
		int len = size();

		obj = peek();
		// 删除栈顶元素，removeElementAt()的实现在Vector.java中
		removeElementAt(len - 1);

		return obj;
	}

	/**
	 * peek函数：返回栈顶元素，不执行删除操作
	 */
	public synchronized E peek() {
		int len = size();

		if (len == 0)
			throw new EmptyStackException();
		// 返回栈顶元素，elementAt()具体实现在Vector.java中
		return elementAt(len - 1);
	}

	/**
	 * 栈是否为空
	 */
	public boolean empty() {
		return size() == 0;
	}

	/**
	 * 查找“元素o”在栈中的位置：由栈底向栈顶方向数
	 */
	public synchronized int search(Object o) {
		// 获取元素索引，elementAt()具体实现在Vector.java中
		int i = lastIndexOf(o);

		if (i >= 0) {
			return size() - i;
		}
		return -1;
	}
```






