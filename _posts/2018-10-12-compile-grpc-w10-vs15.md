---
layout: post
title: "Win10 使用 VS2015 编译 gRPC"
subtitle: "随记"
date: 2018-10-12 12:12:33
catalog: true
tag: 
    - 随记
    - gRpc
---

如果你按照GitHub上的官方指导文档安装步骤来，会出现很多错误。后来也尝试了很多博客上的方法，也都无效，但是给我提供了很好的思路。挣扎着等了一周的进度条，终于编译完成并运行了第一个HellWorld。在此记录下，希望能让后来人少走弯路。
先介绍下环境：Win10专业版 X64，VS2015社区版，GRPC
* 准备环境。下面是[官网](https://github.com/grpc/grpc/blob/master/BUILDING.md#user-content-windows)上标出的需要预装的依赖。这里使用到了 choco ，安装很简单，这里就不一一赘述了。

```
1.Install Visual Studio 2015 or 2017 (Visual C++ compiler will be used).
2.Install Git.
3.Install CMake.
4.Install Active State Perl (choco install activeperl) - required by boringssl
5.Install Go (choco install golang) - required by boringssl
6.Install yasm and add it to PATH (choco install yasm) - required by boringssl
```

* 然后从GitHub上下载源码。执行命令

```
git clone https://github.com/grpc/grpc.git
git submodule update --init --recursive 
```
这里建议自备梯子，设好代理后再下载，因为 Git 因为某些原因，下载速度限制的特别严重。这里一定要确认仓库已经完整的克隆下来，子模块依赖也已经完全更新完毕。在子模块更新时可能会有几个依赖下载不了，就需要各位发动聪明才智去找解决方法了。
* 下载完毕后，找到并打开文件 ``` ./grpc/third_party/zlib/gzguts.h ```
找到 
```
#ifdef _WIN32
#inlcude <stddef.h>
#endif
```
改为
```
#ifdef _WIN32
#include <stddef.h>
#pragma warning(disable:4996)
#endif
```
* 然后就可以安装了。按照官网文档 [grpc-cpp 安装步骤](https://github.com/grpc/grpc/blob/master/BUILDING.md) ，管理员权限打开vs2015命令行（这里我编译的是64位的，所以打开的命令行也得是64位的）
```
// 使用 cd 切换到下载源码的具体目录
mkdir .build && cd .build
cmake .. -G "Visual Studio 14 2015 Win64" -DCMAKE_BUILD_TYPE=Release
cmake --build .
```
在这里我遇到一个编译错误。具体的我没截图，但是在 GitHub 上的 [Issue](https://github.com/grpc/grpc/issues/15858) 上有记录。按照后面给出的步骤，找到一个文件并删除后面的那一行注释即可。
解决完上面的问题后，重新执行下 ```cmake --build .```
然后等待编译完成就OK啦，然后我们需要的 .lib .exe. .h 文件位置在这里就不一一赘述了。在编译过程中，可能会因为环境不同出现些奇奇怪怪的错误。同学们一定不要着急，说来编译这个的过程教给我最多的就是耐心了
* 运行 HelloWorld
打开到源码中提供的实例源码目录，找到 .proto 文件，文件里定义了服务的接口，用来生成源代码文件。命令格式如下：
```
protoc -I="./protos" --grpc_out="./protos" --plugin=protoc-gen-grpc="F:\GIT\grpc-2\grpc\.build\Debug\grpc_cpp_plugin.exe" "./protos\hw.proto"
protoc -I="./protos" --cpp_out="./protos" "./protos\hw.proto"
```
这里的插件位置需要根据你的实际情况做一下改动。命令执行完成后，proto 会根据你的 .proto 文件中定义的服务及实体来生成源码文件(protobuf 是一种数据定义的协议，可以理解成 json 一样，不过功能更为强大，详情可以参考[官方文档](https://developers.google.com/protocol-buffers/docs/overview))。我们把源码文件拷贝至 gRpc提供的HelloWorld源码目录下后，通过 VS就可以直接运行啦。
