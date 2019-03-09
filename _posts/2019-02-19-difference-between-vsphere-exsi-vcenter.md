---
layout: post
title: "vSphere、ESXi和vCenter的区别"
subtitle: "搬运"
date: 2019-02-19 12:12:33
catalog: true
tag: 
    - 翻译
    - 虚拟化
---
[原文地址](http://www.mustbegeek.com/difference-between-vsphere-esxi-and-vcenter/)


-----------------------------

这些天来，对于VMware的虚拟化解决方案有很多疑问。在业内，VMware毫无疑问是首屈一指的虚拟化方案提供商。人们在刚开始接触VMware的虚拟化平台时，经常会在学习 VMware vSphere 和它的组件时感到困惑。所以，今天我在这里会对它们做一个全面的介绍。人们很难了解 vSphere 的主要组件。而去了解 vSphere、ESXi和vCenter之间的差异是非常有必要的。如果想要进一步了解vSphere，你可以在VMware Workstation 里安装一个 vSphere。

--------------------------------------------------


VMware Inc. 开发了许多产品套件，特别是在提供不同的虚拟化解决方案领域。包括很多云产品，数据中心产品，和桌面产品等等。

 vSphere是一个属于数据中心产品的软件套件。vSphere就像微软 Office 套装一样拥有许多产品，比如 Office，Excel等。vSphere同样也包括很多软件组件，比如 vCenter、ESXi、vSphere client 等等。所以这些软件的合集，就叫做vSphere。vSphere不是一种你可以安装和使用的软件，它仅仅是一个软件套件的合集。

ESXi、vSphere client 和 vCenter 都是 vSphere 的部件。ESXi server是最重要的部分，ESXi是一个[一类虚拟化管理器(type 1 hypervisor)](https://vapour-apps.com/what-is-hypervisor/)。所有的虚拟机或者客户机操作系统都安装在 ESXi 服务器上，同时，你可能还需要vSphere中的其他部件-- vSphere client 或者 vCenter。管理员可以通过 vSphere client 连接 ESXi 服务器来访问或者管理虚拟机。vSphere client 是用来从客户端机器连接 ESXi 执行任务的。所以，现在的问题是，vCenter是什么？我们为什么需要他？我们完全可以[通过 vSphere client来克隆虚拟机](http://www.mustbegeek.com/create-copy-of-existing-virtual-machine-in-esxi-server/)，而不需要 vCenter server。

vCenter server 和 vSphere client 类似，但是它是一个拥有更强大功能的服务器。vCenter server 可以安装在 Linux 或者 Window上。VMware vCenter server 是一个管理虚拟机和 ESXi 服务器的中心化管理应用。vSphere client可以通过访问 vCenter server来管理多个 ESXi 服务器。vCenter server有许多企业级特性，例如 vMotion，VMware高可用、VMware更新管理器和VMware 分布式资源调用器(DRS)。例如，你可以很方便的[使用 vCenter server 克隆个已存在的虚拟机](http://www.mustbegeek.com/clone-virtual-machine-in-vmware-vcenter/)一。所以vCenter是vSphere套装中的重要组成部分。你必须要单独购买vCenter的许可证。

![vSphere套件示意图](/img/cloudcomputing/vSphereProductSuite.png)

上面的示意图更为形象地展示了 vSphere 套件。vSphere是一个产品套装， ESXi是安装在物理机上的管理器。vSphere Client安装在一个笔记本或者桌面PC上，用于访问ESXi服务器进行虚拟机的创建和管理。vCenter server像一个虚拟机一样安装在 ESXi上面。vCenter server同样也可以安装在不同的独立物理服务器中，但为什么不适用虚拟化呢？在拥有多个ESXi服务器和数十个虚拟机时，vCenter server的应用就比较频繁了。在小环境下的管理，通常都会使用 vSphere client 来直连 ESXi 服务器。

-------------------------------

译者注：

问题1：按原文来说，ESXi 是直接运行在硬件上的虚拟化管理器，那么它与KVM之间的关系又是什么？

Hypervisor 按类型划分的话分两种，`type 1 hypervisor`是一种直接安装在裸机上的管理程序。 ，通常用在服务器领域中，如`EXSi, Hyper-V`等。`type 2 hypervisor`通常需要一个宿主操作系统来提供虚拟化服务(如IO设备支持和内存管理)，通常用在个人PC，如`VMware Workstation, VirtualBox`等等。所以如何去划分等级，不是在于操作系统和虚拟化软件的安装顺序，而是在于虚拟化软件能够做到的事情。

而KVM，是一个基础Linux内核的一个虚拟化技术，译者理解应该属于`type 1`。像是OpenStack，国产zstack都是基于KVM的云计算平台。

参考资料

* [什么是hypervisor](https://vapour-apps.com/what-is-hypervisor/)
* [KVM 是一类还是二类的hypervisor](https://serverfault.com/questions/855094/is-kvm-a-type-1-or-type-2-hypervisor)

