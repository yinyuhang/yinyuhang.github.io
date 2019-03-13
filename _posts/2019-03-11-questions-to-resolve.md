---
layout: post
title: "问题，思路，答案，总结"
subtitle: "随记"
date: 2019-03-11 20:00:05
tag: 
    - 疑问
    - c++
---

------------------------------------

#### `g++`必须使用`-lxxx`才能通过编译
```
$ g++ log.cpp -llog4cplus
$ g++ log.cpp
/tmp/cc6uljhh.o: In function `main':
log.cpp:(.text+0x20): undefined reference to `log4cplus::Logger::getRoot()'
log.cpp:(.text+0x31): undefined reference to `log4cplus::Logger::~Logger()'
collect2: error: ld returned 1 exit status
```
**思路**
* -l 选项的作用
* 报错信息处于gcc编译的哪个阶段

**答案**
![gcc的编译阶段](/img/post/gcc-pharse.png)
该问题出在链接阶段，GNU链接器没有找到正确的库文件

`-l`在man page是这么描述的
```
在链接时搜索指定名称的库文件。

链接器会按照命令给出的顺序来搜索和处理库文件与obj文件。因此, 在*foo.o -lz bar.o*命令中，z会放在foo.o之后，bar.o之前。如果bar.o引用了z中的函数，这些函数可能不会被加载。

链接器会搜索库的标准目录列表下命名符合**liblibrary.a**规则的库文件。
```
问题解决。LD_LIBRARY_PATH只是指定了库文件的默认搜索路径。这里需要注意的点事，具体要使用哪个库文件，需要我们通过-l选项来指定。gcc会根据-l的顺序及后面的值，搜索命名规则为`liblibrary.a`的文件。

**验证**
```
$ g++ -c log.cpp
$ g++ log.o -o a.out -lmyname
/usr/bin/ld: cannot find -lmyname
collect2: error: ld returned 1 exit status
$ sudo cp /usr/local/lib/liblog4cplus-1.2.so.5.1.7 /usr/local/lib/libmyname.so
$ g++ log.cpp -lmyname
$ ls
a.out  log.cpp  log.o
```