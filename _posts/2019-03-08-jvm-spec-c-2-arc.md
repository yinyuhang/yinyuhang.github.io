---
layout: post
title: "JVM Specification 中文版 第二章-JVM架构"
subtitle: "翻译"
date: 2019-03-08 18:54:17
catalog: true
tag: 
    - 翻译
    - JVM Specification
---
本章只阐述了一个抽象的模型。而不会谈到JVM任何具体的实现。

为了正确的实现JVM，必须能够正确的解析 `class` 的文件格式并执行其中的操作。`JVM规范`不涉及实现细节，因为这样会为人们戴上枷锁。比如，运行时数据区域的内存布局，垃圾回收(gc)算法的使用和其他可对JVM指令可做出的优化(例如，将指令翻译为机器码)，这些都交由实现者决定。

本规范对 Unicode 的所有引用都来自 [`The Unicode Standard, Version 6.0.0`](http://www.unicode.org/)

### 2.1 类文件格式
由JVM执行编译后的代码使用了一种硬件-操作系统无关的二进制格式，通常(但不是必须的)会保存在一个`class`文件中。`class`文件精确地对类和接口转化为另一种表现形式，**including details such as byte ordering that might be taken for granted in a platform-specific object file format.**

在第四章-class文件格式会对`class`文件格式进行详细描述。

### 2.2 数据类型
和Java语言一样，JVM的操作类型有两种，基本类型与引用类型。相应的，有两种值可以存储在变量中，作为参数或者函数返回值，然后在基本类型和引用类型操作。

JVM会希望编译器而不是JVM本身在运行期间对所有的类型进行检查。基本类型的值不需要被标记或者在运行期间判断它们的类型，也不需要与引用类型分开。因为JVM会根据不同的类型调用不同的指令。比如，`iadd, ladd, fadd 和 dadd`都是JVM用来将两个数字相加并返回结果的指令，但它们分别对应了不同的操作类型：`int, long, float, double`。对于JVM指令集支持的类型，参见[2.11.1](https://docs.oracle.com/javase/specs/jvms/se8/html/jvms-2.html#jvms-2.11.1)

JVM对对象有显示的支持。一个对象要么是一个动态创建的类实例，要么就是一个数组。JVM用内置类型`reference`来表示一个对象引用。你可以把引用理解成一个指向对象的指针。一个对象可以对应多个引用。所有对象会通过`reference`类型来进行操作，传递和测试。

### 2.3 基本类型的值
JVM支持的基本类型有数值、[boolean](https://docs.oracle.com/javase/specs/jvms/se8/html/jvms-2.html#jvms-2.3.4)和[`returnAddress`](https://docs.oracle.com/javase/specs/jvms/se8/html/jvms-2.html#jvms-2.3.3)

数值类型包括整型和浮点型。

整型有：
* `byte`，8位有符号二进制补码整数，默认值0
* `short`，16位有符号二进制补码整数，默认值0
* `int`，32位有符号二进制补码整数，默认值0
* `long`，64位有符号二进制补码整数，默认值0
* `char`，用16位无符号整数来表示的，基本多文种平面(Basic Multilingual Plane)的Unicode值，以UTF-16编码，默认值是\`\\u0000\`

浮点型有：
* float，值为float value set或float-extended-exponent(如果支持)中的元素，默认值正0
* double，值为double value set或double-extended-exponent(如果支持)中的元素，默认值正0

boolean 会被编码为`true`或`false`，默认为`false`。

*在第一版的JVM规范中，JVM并没有把`boolean`作为一个类型看待。但是，JVM对`boolean`的支持确实有限。第二版JVM规范中开始将`boolean`作为类型看待。*

`returnAddress`的值是JVM指令操作码的指针。在所有基本类型中，只有`returnAddress`没有直接体现在Java语言中。

#### 2.3.1 整型
[推荐阅读原文](https://docs.oracle.com/javase/specs/jvms/se8/html/jvms-2.html#jvms-2.3.1)

整型值的范围：
* `byte`, \[-128, 127\](2的七次方)
* `short`，\[-32768, 32767](2的十五次方)
* `int`，\[--2147483648, 2147483647\](2的31次方)
* `long`，\[-9223372036854775808, 9223372036854775807\](2的63次方)
* `char`，\[0, 65565\](2的八次方)

#### 2.3.2 浮点数与值集
[推荐阅读原文](https://docs.oracle.com/javase/specs/jvms/se8/html/jvms-2.html#jvms-2.3.2)

#### 2.3.3 `returnAddress`
JVM有三个指令会用到`returnAddress`类型： [`jsr`](https://docs.oracle.com/javase/specs/jvms/se8/html/jvms-6.html#jvms-6.5.jsr), [`ret`](https://docs.oracle.com/javase/specs/jvms/se8/html/jvms-6.html#jvms-6.5.ret), [`jsr_w`](https://docs.oracle.com/javase/specs/jvms/se8/html/jvms-6.html#jvms-6.5.jsr_w)。`returnAddress`的值是一个指针，指向JVM指令的操作码。不像整型，`returnAddress`在Java中没有对应的类型，并且无法在运行时期修改。

#### 2.3.4 `boolean`

