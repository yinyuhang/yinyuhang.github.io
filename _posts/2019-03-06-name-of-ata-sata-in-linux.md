---
layout: post
title: "Linux 下 ATA 与 SATA 的命名规则"
subtitle: "StackOverflow"
date: 2019-03-06 22:11:05
catalog: true
tag: 
    - 翻译
    - Linux
---
### 问题
[原文地址](https://unix.stackexchange.com/questions/2447/names-for-ata-and-sata-disks-in-linux)

假设现在有两块磁盘，**one master SATA and one master ATA**,他们在 `/dev` 下会如何呈现？

-----------------------------

### 回答
这个要取决与你的 SATA 驱动和系统配置 (distribution's configuration), 可能会是 `/dev/hda` 和 `/dev/hdb`, 或者 `/dev/hda` 和 `/dev/sda`，或者是 `/dev/sda` 和 `/dev/sdb`.不同的发行版和驱动正在朝着将所有磁盘命名为 `sd*` 的方向发展，但是传统的 PATA 驱动会使用 `hd*` 而且少部分的 SATA 驱动也会这么做。

设备名取决于 `udev` 的配置。比如，在 Ubuntu 14.04上，下面来自 `/lib/udev/rules.d/60-persistent-storage.rules` 的配置会把所有的 ATA 硬盘显示为 `/dev/sd*` 并会将所有的 ATA CD　设备显示为　`/dev/sr*`:

```
# ATA devices with their own "ata" kernel subsystem
KERNEL=="sd*[!0-9]|sr*", ENV{ID_SERIAL}!="?*", SUBSYSTEMS=="ata", IMPORT{program}="ata_id --export $tempnode"
# ATA devices using the "scsi" subsystem
KERNEL=="sd*[!0-9]|sr*", ENV{ID_SERIAL}!="?*", SUBSYSTEMS=="scsi", ATTRS{vendor}=="ATA", IMPORT{program}="ata_id --export $tempnode"
```

-------------------------------------

### 译者注
大部分情况下，可以认为 IDE 接口的硬盘命名为 `/dev/hd*`， SATA 接口的硬盘命名为 `/dev/sd*`。