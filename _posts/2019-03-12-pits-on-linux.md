---
layout: post
title: "Linux 踩坑小计"
subtitle: "随记"
date: 2019-03-12 21:12:05
tag: 
    - 随记
    - Linux
---

### Docker 内容器可以ping通宿主机IP，但是无法访问其他容器映射在宿主机上的端口
问题描述：现宿主机A内Docker运行了两个容器B与C。B的80端口映射到宿主机A的80端口，外部PC可以访问B提供的服务。但在C容器内，经测试能ping通宿主机A的IP，但是却无法访问B提供的服务

问题原因：docker的一个已知Bug

解决方法：Centos7环境下，向`/etc/firewalld/zones/public.xml`中添加一条规则：
```
# 这里的address需要根据容器的实际IP与子网掩码来进行修改。
<rule family="ipv4">
  <source address="172.18.0.0/16"/>
  <accept/>
</rule>
```
然后重启防火墙`systemctl restart firewalld`即可。

--------------------------

### 为虚拟机添加硬盘后，`fdisk -l`中不显示
先`cat /proc/scsi/scsi`
我机器上的输出为
```
Attached devices:
Host: scsi2 Channel: 00 Id: 00 Lun: 00
  Vendor: ATA      Model: KINGSTON SA400S3 Rev: 71B1
  Type:   Direct-Access                    ANSI  SCSI revision: 05
```
这里根据文件内容，使用命令
使用命令
```
echo "scsi add-single-device 2 0 1 0">/proc/scis/scsi 
# echo "scsi add-single-device w x y z" > /proc/scsi/scsi
# 参数值 w、x、y 、 z，含义如下：
# w（Host）是主机适配器标识，第一个适配器为零（0）
# x （Channel）是主机适配器上的 SCSI 通道，第一个通道为零（0）
# y （ID）是设备的 SCSI 标识，要根据已有 Id + 1
# z （Lun）是 LUN 号，第一个 LUN 为零（0）
```
再使用 ``` fdisk -l ``` 已经可以显示出相关磁盘信息
**参考资料：**
* [虚拟机VMware新增硬盘无法识别问题](https://www.linuxidc.com/Linux/2017-03/142007.htm)

------------------

### Centos 7 上 ZFS 的安装及使用
安装命令
```
# 以下命令均以管理员权限执行
rpm -ivh epel-release-7-0.2.noarch.rpm
yum install -y dkms
yum install http://download.zfsonlinux.org/epel/zfs-release.el7_5.noarch.rpm
yum install zfs 
# 官网上给出的是 modprobe zfs, 但是会报 Module zfs not found.
/sbin/modprobe zfs
lsmod | grep zfs
```
**参考资料：**
* [zfs-on-linux-RHEL-and-CentOS](https://github.com/zfsonlinux/zfs/wiki/RHEL-and-CentOS)
* [Oracle官方使用文档](https://docs.oracle.com/cd/E24847_01/html/819-7065/zfsover-1.html#scrolltoc)

----------------------