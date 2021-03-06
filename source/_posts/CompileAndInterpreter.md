title: 编译与解释
date: 2015-01-08 22:20:56
categories: JVM
tags: coding
description: 很多文章习惯将高级编程语言分成编译型语言和解释型语言，其实这是有误导成分的。语言一般只会定义其抽象语义，而不会强制性要求采用某种实现方式。
---

很多文章和书籍习惯将高级编程语言分成编译型语言和解释型语言，其实这是有误导成分的，如果你从编程语言底层分析其运行过程，会发现编译与解释往往并不是独立存在的，下面分析理解其内部机理。

##解析器与解释器
首先来明确两个基本概念：解析器是[parser](http://en.wikipedia.org/wiki/Parsing)，而解释器是[interpreter](http://en.wikipedia.org/wiki/Interpreter_computing)。两者不是同一样东西。

`解析器`是编译器/解释器的重要组成部分，也可以用在IDE之类的地方；其主要作用是进行语法分析，提取出句子的结构。

广义来说输入一般是程序的源码，输出一般是[语法树](http://en.wikipedia.org/wiki/Parse_tree)（syntax tree，也叫parse tree等）或[抽象语法树](http://en.wikipedia.org/wiki/Abstract_syntax_tree)（abstract syntax tree，AST）。

广义的解析器里一般包含：
1. 扫描器（scanner，也叫tokenizer或者lexical analyzer，词法分析器）。扫描器的输入一般是文本，经过词法分析，输出是将文本切割为单词的流。
2. 狭义的解析器（parser，也叫syntax analyzer，语法分析器）。狭义的解析器输入是单词的流，经过语法分析，输出是语法树或者精简过的AST。

来看一段源码：$i = a + b * c$

<center>　![](http://alexyoung.qiniudn.com/parser.png) <center/>

其中词法分析由扫描器完成，语法分析由狭义的解析器完成。 

`解释器`则是实现程序执行的一种实现方式，与`编译器`相对。它直接实现程序源码的语义，输入是程序源码，输出则是执行源码得到的计算结果；编译器的输入与解释器相同，而输出是用别的语言实现了输入源码的语义的程序。通常编译器的输入语言比输出语言高级，但不一定；也有输入输出是同种语言的情况，此时编译器很可能主要用于优化代码。 

把同样的源码分别输入到编译器与解释器中，得到的输出不同： 

<center> ![](http://alexyoung.qiniudn.com/parser1.png) <center/>

值得留意的是，编译器生成出来的代码执行后的结果应该跟解释器输出的结果一样——它们都应该实现源码所指定的语义。


##“编译器”与“编译型语言”

一种高级编程语言，从源代码开始，一直到被执行生成相应的动作，大约经历了这几个步骤：

<center> 源代码 ====> 中间表示形式（e.g obj） ====> 基本操作序列 ====> 生成最终动作 <center/>

编译型语言的代表是C，源代码被编译之后生成中间文件（.o和.obj），然后用链接器和汇编器生成机器码，也就是一系列基本操作的序列，机器码最后被执行生成最终动作。编译生成了目标文件，而这个目标文件是针对特定的 CPU 体系的，为 ARM 生成的目标文件，不能被用于 MIPS 的 CPU。这段代码在编译过程中就已经被翻译成了目标 CPU 指令，所以，如果这个程序需要在另外一种 CPU 上面运行，这个代码就必须重新编译。

解释型的语言以Ruby为例，也经历了这些步骤，不同的是，C语言会把那些从源代码“变”来的基本操作序列（保存）起来，而Ruby直接将这些生成的基本操作序列（Ruby虚拟机）指令丢给Ruby虚拟机执行然后产生动作了。即解释型语言同样也可能存在某种编译过程，但他们编译生成的通常是一种『平台无关』的中间代码，这种代码一般不是针对特定的 CPU 平台，他们是在运行过程中才被翻译成目标 CPU 指令的，因而，在 ARM CPU 上能执行，换到 MIPS 也能执行，换到 X86 也能执行，不需要重新对源代码进行编译。

所以我们看到的现象是，编译型语言要先编译再运行，而解释性语言直接“运行”源代码。

换个角度说，采用编译和解释方式实现虚拟机最大的区别就在于是否存下目标代码：编译的话会把输入的源程序以某种单位（例如基本块/函数/方法/trace等）翻译生成为目标代码，并存下来（无论是存在内存中还是磁盘上，无所谓），后续执行可以复用之；解释的话则是把源程序中的指令逐条解释，不生成也不存下目标代码，后续执行没有多少可复用的信息。


##“解释器”与“解释型语言”

很多资料会说，Python、Ruby、JavaScript都是“解释型语言”，是通过解释器来实现的。这么说其实很容易引起误解：**语言一般只会定义其抽象语义，而不会强制性要求采用某种实现方式**。

例如说C一般被认为是“编译型语言”，但C的解释器也是存在的，例如[Ch](http://www.softintegration.com/)。同样，C++也有解释器版本的实现，例如[Cint](https://root.cern.ch/drupal/content/cint)。 

一般被称为“解释型语言”的是主流实现为解释器的语言，但并不是说它就无法编译。例如说经常被认为是“解释型语言”的Scheme就有好几种编译器实现，其中率先支持R6RS规范的大部分内容的是Ikarus，支持在x86上编译Scheme；它最终不是生成某种虚拟机的字节码，而是直接生成x86机器码。 

解释器就是个黑箱，输入是源码，输出就是输入程序的执行结果，对用户来说中间没有独立的“编译”步骤。这非常抽象，内部是怎么实现的都没关系，只要能实现语义就行。你可以写一个C语言的解释器，里面只是先用普通的C编译器把源码编译为in-memory image，然后直接调用那个image去得到运行结果；用户拿过去，发现直接输入源码可以得到源程序对应的运行结果就满足需求了，无需在意解释器这个“黑箱子”里到底是什么。 

实际上很多解释器内部是以“编译器+虚拟机”的方式来实现的，先通过编译器将源码转换为AST或者字节码，然后由虚拟机去完成实际的执行。**所谓“解释型语言”并不是不用编译，而只是不需要用户显式去使用编译器得到可执行代码而已** 。
