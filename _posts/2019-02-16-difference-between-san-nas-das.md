---
layout: post
title: "SAN, NAS 和 DAS的区别"
subtitle: "ServerFault"
date: 2019-02-16 12:12:33
catalog: true
tag: 
    - 翻译
    - 存储
---
[原文地址](https://serverfault.com/questions/81723/what-is-the-difference-between-san-nas-and-das)

-------------------------

他们之间的差异就像快设备与文件系统一样。如果你对UNIX很熟的话，这是非常容易理解的，因为在UNIX中，这两者的区别很明显。不过在Windows中也一样。
* 块设备是裸盘的句柄。比如；```/dev/sda``` 是一个磁盘， ```/dev/sda1``` 是在该磁盘上的分区
* 文件系统是建立在在块设备的基础上的。你可以把它挂载出来，然后存储数据。比如 ```mount /dev/sda1 /mnt/somepath```

有了上面这些概念后，我们就能理解他们之间的区别了：

* DAS 是在宿主机上物理上直接连接的块设备。首先你必须初始化一个文件系统，然后才可以使用。与之相关的技术有 IDE，SCSI，SATA等等
* SAN 是通过网络连接的块设备。就像DAS一样，你必须先初始化文件系统然后才能使用。相关的技术有 FibreChannel(FC), iSCSI, FoE等等
* NAS 是通过网络连接的一个文件系统。你可以直接使用。相关技术有 NFS，CIFS，AFS等等
![DAS,SAN,NAS](/img/cloudcomputing/san-das-nas.png)


-----------------

### 译者附
其实IDE与SATA在日常很常见，在以前我们的主机硬盘连接方式大多是以IDE为主，但随着计算机的发展，现在在PC中，无论是机械还是SSD大多都采用了速度更快，功耗更低的SATA接口。关于 SATA与IDE，[这里](/2019/03/06/name-of-ata-sata-in-linux)还有更多。还有一种价格较贵，但相应速度更快的磁盘，使用了 PCIe 接口，译者公司就有这么一台服务器用来做性能测试，那速度没话说，使用 FIO ，zfs 16K 8线程 顺序读能飚到 12G/s。而 SCSI 大多用在服务器，一般家用电脑很少看到。