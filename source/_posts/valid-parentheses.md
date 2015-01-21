title: Valid Parentheses
date: 2014-11-28 22:42:18
categories: Java
tags: [stack, leetcode]
description : Given a string containing just the characters '(', ')', '{', '}', '[' and ']', determine if the input string is valid. The brackets must close in the correct order, `()` and `()[]{}` are all valid but `(]` and `([)]` are not.
---

##Valid Parentheses
Question：Given a string containing just the characters '(', ')', '{', '}', '[' and ']', determine if the input string is valid.
The brackets must close in the correct order, `()` and `()[]{}` are all valid but `(]` and `([)]` are not.
<!--more-->  

###Best Solution
```java
public static boolean isValid(String s){
		HashMap<Character, Character>map = new HashMap<Character, Character>();
		map.put('(', ')');
		map.put('[', ']');
		map.put('{', '}');
		
		Stack<Character>stack = new Stack<Character>();
		for (int i = 0; i < s.length(); i++) {
			char head = s.charAt(i);
			
			if (map.containsKey(head)) {
				stack.push(head);
			}else if (map.values().contains(head)) {
				if (!stack.isEmpty() && map.get(stack.peek()).equals(head)) {
					stack.pop();
				}else {
					return false;
				}
			}
		}
		return stack.isEmpty();
	}
```
**Analyze**：思想非常简单，当看到左括号的时候就push入栈，看到右括号时与stack.peek相匹配，匹配成功则pop，如果字符串valid，则stack.isEmpty会为true，即所有左括号都匹配到对应的右括号。
PS. 一定要在else if内加栈空判断，防止字符串以右括号开始，如`}[()]`
<br/>

###My Solution
```java
public static boolean isValid(String s){
		HashMap<String, String>map = new HashMap<String, String>();
		map.put("(", ")");
		map.put("[", "]");
		map.put("{", "}");
		
		if (s.length()%2 != 0) {
			return false;
		}
		
		Stack<String>stack = new Stack<String>();
		for (int i = s.length(); i > 0; i--) {
			stack.push(s.substring(i-1, i));
		}
		List<String> list = new ArrayList<String>();
		
		if (!map.containsKey(stack.peek())) {
			return false;
		}
		
		while (!stack.isEmpty()) {
			String head = stack.pop();
			if (map.containsKey(head)) {
				list.add(head);
			}else {
				if (head.equals(map.get(list.get(list.size() - 1)))) {
					list.remove(list.size() - 1);
				}else {
					return false;
				}
			}
		}
		
		if (!list.isEmpty()) {
			return false;
		}else {
			return true;
		}
	}
```