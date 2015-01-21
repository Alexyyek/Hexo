title: Java处理字符串-删除所有标点
date: 2014-11-15 14:45:40
categories: Java
tags: [skill]
---
###代码
```java
str = str.replaceAll("[\\pP‘’“”]", "");   
```
在这里利用的是Unicode编码，Unicode 编码并不只是为某个字符简单定义了一个编码，而且还将其进行了归类。

<!--more-->
\pP 其中的小写 p 是 property 的意思，表示 Unicode 属性，用于 Unicode 正表达式的前缀。

大写 P 表示 Unicode 字符集七个字符属性之一：标点字符。其他六个包括：
- L：字母 
- M：标记符号（一般不会单独出现）
- Z：分隔符（比如空格、换行等）      
- S：符号（比如数学符号、货币符号等）
- N：数字（比如阿拉伯数字、罗马数字等）
- C：其他字符

Java 中用于 Unicode 的正则表达式数据都是由 Unicode 组织提供的。
Unicode 正则表达式[标准](http://www.unicode.org/reports/tr18/)（可以找到所有的子属性）
这个[文本文档](http://www.unicode.org/Public/UNIDATA/UnicodeData.txt)一行是一个字符，第一列是 Unicode 编码，第二列是字符名，第三列是 Unicode 属性，以及其他一些字符信息。

###实例
```javascript
 String str = ",.!，，D_NAME。！；‘’”“《》**dfs  #$%^&()-+1431221中国123漢字かどうかのjavaを決定";  
 str = str.replaceAll("[\\pP‘’“”]", "");  
 System.out.println(str); 
```
输出结果: DNAMEdfs  $^+1431221中国123漢字かどうかのjavaを決定
