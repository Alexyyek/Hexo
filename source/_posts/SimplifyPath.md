title: Simplify Path
date: 2014-11-30 21:13:12
categories: Java
tags: [stack, leetcode]
description: Question:Given an absolute path for a file (Unix-style), simplify it.
---

##Simplify Path

Question:Given an absolute path for a file (Unix-style), simplify it.
<!--more-->

For example,
path = `/home/` => `/home`
path = `/a/./b/../../c/` => `/c`
<br/>

| Situation        |   Meaning   |  Solution  |
| --------   |  :-----: | :----:  |
| /..     | 返回上级目录 |   若stack不空，stack.pop()     |
| /.        |   留在本目录   |   无操作   |
| /         |   多余的slash  |   无操作   |
| other     |   有效目录名   |    入栈    |


###My solution
```java
public static String soluition(String path) {
        String[] unit = path.split("/");
        StringBuffer line = new StringBuffer();
        Stack<String> stack = new Stack<String>();
        for (int i = 0; i < unit.length; i++) {
        	if (!unit[i].equals("")) {
        		if (unit[i].equals(".")) {
    				continue;
    			}else if (unit[i].equals("..")) {
    				if (!stack.isEmpty()) {
    					stack.pop();
        				stack.pop();
					}
    			}else {
    				stack.push("/");
    				stack.push(unit[i]);
    			}
			}
		}
        if (stack.isEmpty()) {
			return "/";
		}else {
			while (!stack.isEmpty()) {
				line.insert(0, stack.pop());
			}
			return line.toString();
		}
    }
```  