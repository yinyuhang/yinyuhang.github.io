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

一个合格的JVM，必须能够正确的解析 `class` 的文件格式并执行其中的操作。`JVM规范`不涉及实现细节，这样会影响后来者的发挥。比如，运行时数据区域的内存布局，垃圾回收(gc)算法的使用和其他可对JVM指令可做出的优化(例如，将指令翻译为机器码)，这些都交由实现者决定。

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
虽然JVM定义了布尔类型，但是只对其提供了有限的支持。JVM并没有为布尔值提供专门的操作指令，Java中的布尔类型在编译后，在JVM中会被视为int类型。

JVM没有对布尔类型的数字提供支持。[`newarray`](https://docs.oracle.com/javase/specs/jvms/se8/html/jvms-6.html#jvms-6.5.newarray)指令用来创建布尔数组。[`baload`](https://docs.oracle.com/javase/specs/jvms/se8/html/jvms-6.html#jvms-6.5.baload)和[`bastore`](https://docs.oracle.com/javase/specs/jvms/se8/html/jvms-6.html#jvms-6.5.bastore)指令用来访问修改布尔数组，这两个指令属于byte数组。

*布尔数组经JVM编码后，1表示true，0表示false。Java中的布尔值会被JVM映射为int类型，编译器必须使用同样的编码。*

### 2.4 引用类型
引用类型有三种：`class types, array types and interface types`。它们的值分别是动态创建的类实例，数组或者接口实现类的引用。

一个数组是由一组一维的元素组成的(长度并不是由类型确定)。数组的元素也可能是数组，除此之外，数组的元素必须是基本类型、类或者接口。

一个`reference`的值可能是个空引用，一个不指向任何对象的引用，用`null`来表示。`null`起初并不是任何运行时类型，但是可以转成任何类型。`reference`的默认值是`null`。

本规范不会对`null`的编码做要求。

### 2.5 运行时数据区域
JVM为程序的运行定义了不同的运行时数据区。其中有些会随着JVM的启动停止而创建销毁。其余的则供单个线程使用。线程内数据区的生命周期与线程保持一致。

#### 2.5.1 `pc`寄存器
JVM可以在同一时间执行多个线程(JLS §17)。每个JVM的线程都拥有自己的`pc(程序计数器)`寄存器。**At any point, each JVM thread is executing the code of a single method, namely the current method ([§2.6](https://docs.oracle.com/javase/specs/jvms/se8/html/jvms-2.html#jvms-2.6)) for that thread.**如果是非`native`方法，`pc`寄存器内是JVM当前执行的指令的地址。如果是`native`方法，`pc`寄存器里的值是无意义的。`pc`寄存器可以存储一个`returnAddress`或者**a native pointer on the specific platform.**

#### 2.5.2 JVM栈
每个JVM线程都拥有一个私有栈，会随线程一起创建。栈中存储了很多帧(frames [§2.6](https://docs.oracle.com/javase/specs/jvms/se8/html/jvms-2.html#jvms-2.6))。JVM中的栈与其他语言(比如C)中的栈类似，它存储着本地变量和局部结果，在调用函数与函数返回时也用到了栈。对于栈中的帧，只能进行`push`和`pop`操作，因此帧的空间可以是从堆上分配的。而且栈存储空间并不要求是连续的。

**在第一版JVM规范中，JVM栈也被成为Java栈**

栈的大小，可以是固定的，也可以是根据使用动态计算并扩展，本规范对此不做要求。如果JVM栈是固定大小，那么应该可以在创建时单独指定。

**JVM的实现应为使用者提供接口，用于设置初始大小，如大小可动态扩展，应可以设置大小范围。**

与栈有关的异常条件如下：
* 如果线程需要的栈的大小超过允许的范围，抛出`StackOverflowError`。
* 如果栈大小可动态扩展，但是在尝试扩展大小时却没有足够的空间，或者没有足够的空间去为新线程初始化一个JVM栈，抛出`OutOfMemoryError`。

#### 2.5.3 堆
JVM所有线程共享一个堆。堆是一个用于存储所有类实例和数组的运行时数据区。

堆在虚拟机启动之时被创建。堆中存储的对象会由一个内存管理系统(aka垃圾回收器)来进行回收；Java中的对象不需要显式的手动释放。JVM不依赖于某种特定的内存管理系统，而且实现者可以根据自己的的系统要求来选择内存管理技术。堆可以是固定大小，也可以根据实际的使用情况来进行扩展和收缩。堆的内存空间同样不要求是连续的。

**JVM的实现应该为用户或程序员提供接口，用于控制初始大小，同样，如果大小是可以动态扩展的，应可以设置大小范围**

与堆相关的异常条件如下：
* 如果由内存管理器计算后，剩下的空间小与堆需要的空间，JVM应该抛出`OutMemoryError`。

#### 2.5.4 方法区
JVM的所有线程共享一个方法区。**The method area is analogous to the storage area for compiled code of a conventional language or analogous to the "text" segment in an operating system process.**类的结构就存储在这里面，例如，常量池，字段和方法数据以及方法与构造函数的代码，包括一些用于类和实例初始化或者接口初始化时的特殊方法([§2.9](https://docs.oracle.com/javase/specs/jvms/se8/html/jvms-2.html#jvms-2.9))

方法区在虚拟机启动时被创建。虽然方法区从逻辑上来说，是堆的一部分。

#### 2.5.5 运行时常量池