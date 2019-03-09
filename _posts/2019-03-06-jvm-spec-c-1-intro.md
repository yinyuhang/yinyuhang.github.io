---
layout: post
title: "JVM Specification 中文版 第一章-简介"
subtitle: "翻译"
date: 2019-03-06 08:24:11
catalog: true
tag: 
    - 翻译
    - JVM Specification
---
### 一点点历史
Java是一门通用的，支持并发且面相对象的编程语言。它的语法和 C、C++ 有些类似，但是它舍弃了 C/C++ 中许多复杂、难以理解且不安全的特性。最初，Java平台是为了解决那些面向网络用户的应用程序构建问题。它旨在支持多种主机架构，并允许软件组件的安全交付。为了满足这些需求，编译后的代码必须能通过网络传输，并能保证在任意的客户机上安全的运行。

随着万维网(World Wide Web)的普及，Java 的这些属性变得更加有趣。网络浏览器使成千上万的人能便捷的访问各种丰富多彩的媒体内容。无论你使用的是什么机器，什么操作系统，是快速网络还是慢速的调制解调器(slow modem)，WWW能保证每个人获取到的信息是一样的。

网络爱好者很快发现HTML的格式支持的内容太有限了。HTML的扩展，例如表单，仅仅是强调了这些限制，当人们发现所有浏览器都无法同时包含用户想要的所有特性。扩展性变成了唯一的出路。

HotJava(一种浏览器)因为使用了Java平台，可以在HTML页面中嵌入可执行程序。程序和HTML页面一起下载到客户的浏览器中，再被浏览器解析之前，程序会仔细地检查环境，确保安全。就像 HTML页面一样，编译后的程序是网络，宿主机无关的。同一个程序，无论它是通过什么途径下载，也无论它是在什么样的机器上被加载运行，它们的行为总是一致的。

一个结合了Java平台的Web浏览器不再局限于一组功能。用户在浏览动态内容的网页时，Java平台可以确保他们的机器不会被破坏。开发者只需要把程序编译一次，就能在所有提供了Java运行环境的机器上执行。

--------------------------

### Java 虚拟机(JVM)
JVM是Java平台的基础。它从技术角度上使Java程序能够跨硬件，跨操作系统执行。编译后的代码产物会很小，并且它能保证用户不被恶意程序所影响。

JVM是一个抽象的计算机器。就像一个真正的计算机一样，它有指令集并且可以在运行时操控不通的内存空间。在虚拟机上来实现一个编程语言是很常见的事；最有名的虚拟机可能就是 UCSD Pascal 的 P-Code 了。

JVM的第一款原型是由 Sun公司实现，**emulated the Java Virtual Machine instruction set in software hosted by a handheld device that resembled a contemporary Personal Digital Assistant (PDA)**. Oracle当前的实现可以在手机，桌面电脑与服务器中模拟JVM，**but the Java Virtual Machine does not assume any particular implementation technology, host hardware, or host operating system. It is not inherently interpreted, but can just as well be implemented by compiling its instruction set to that of a silicon CPU. It may also be implemented in microcode or directly in silicon.**

JVM不关心Java语言的细节，它只关心一个特定的二进制格式，class 文件。一个 class 文件包含了JVM的指令(或者字节码)，一个符号表和其它的一些辅助信息。

因为安全考虑，JVM对class文件的语法与结构有着严格的约束。但是，只要能被编译为一个有效的class文件，任何编程语言都可以在JVM上运行。由于JVM的通用与跨平台的特性，开发者可以基于JVM来实现自己的编程语言。

这里对JVM的所有说明与JAVA(SE8版本)平台兼容，并支持在Java语言规范(SE 8版本)中的编程语言。

--------------------------

###  文档结构
* 第二章会对JVM架构做一个整体介绍
* 第三章会为大家介绍由Java语言编写的代码如何汇编成JVM的指令集
* 第四章定义了被编译后的类与接口 -- 即class 文件的格式--一种硬件与平台无关的二进制格式
* 第五章定义了JVM如何启动，加载，链接和如何对类和接口进行初始化
* 第六章定义了JVM的指令集，指令会以操作码助记符的字母顺序排序
* 第七章会列出JVM操作码助记符的表格并以操作码的值来作为索引

在第二版的JVM规范第二章中，概述了Java编程语言，该语言支持JVM规范，但其本身并不是规范。在Java(SE8版本)的JVM规范中，读者可以参考Java(SE8版本)的语言规范，来学习如何书写Java代码。**References of the form: (JLS §x.y) indicate where this is necessary.**

在第二版JVM规范第八章中详细说明了JVM线程与共享主内存(shared main memory)交互的底层操作。而在JAVA(SE8版本)的JVM规范中，读者可以参考Java(SE8版本)语言规范的第十七章，了解有关线程和锁的信息。第17章会对 JSR133专家组制定的Java内存模型和线程规范进行阐述。

--------------------------


### 一点点约定
本规范引用了来自 JavaSE平台API的类和接口。在所有使用了(除了在例子中声明的那些)，标记 *N*的地方，**the intended reference is to the class or interface named N in the package java.lang.** 除了`java.lang`以外，从其它包引用的类或者接口，会使用全限定名。

每当我们使用名为`java`的子包下的类或者接口的引用时，那么我们会默认其是由 `bootstrap` 类加载器来确定的。

在本规范中，某些特殊字体所代表的含义如下(见注1)：
* 固定宽度的字体用于JVM数据类型、Exception、errors、类文件结构、Prolog code和Java代码块。
* 斜体代表 JVM 汇编语言中的操作码和操作数，**as well as items in the Java Virtual Machine's run-time data areas.**同时也用于介绍新的特性或者表示强调。

用来额外阐述本规范内容的非规范信息，会以较小的缩进文本给出。

-------------------

### 如何联系我们
读者如发现本规范有技术上的错误或者含糊不清的地方，非常欢迎指出，邮件地址： `jls-jvms-spec-comments@openjdk.java.net`。

如有关于 `javac` (Java语言指定的编译器) 如何生成和操作 `class` 文件的疑问，可发邮件至 `compiler-dev@openjdk.java.net`

------------------------

#### 译者注
* 注1: 因为本文档是由Markdown编写的，所以关于字体格式及其代表的意义可能会有变动，暂时还没确定，这里翻译的是官网的设置。

[原文地址](https://docs.oracle.com/javase/specs/jvms/se8/html/jvms-1.html)