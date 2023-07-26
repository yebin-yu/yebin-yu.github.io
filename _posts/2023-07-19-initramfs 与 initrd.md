---
layout:     post
title:      initramfs 与 initrd
subtitle:   我们用的initrd到底是什么，为什么需要这个？
date:       2023-07-19
author:     yebin-yu
header-img: img/tag-bg.jpg
catalog: false
tags:
    - Linux 启动
---

### initramfs 与 initrd

> wikipedia: 

> [initramfs](https://zh.wikipedia.org/w/index.php?title=Initramfs&action=edit&redlink=1)是initrd的替代品。initrd是一个被加载的块设备，内部有[ext2](https://zh.wikipedia.org/wiki/Ext2)一类文件系统的存在，于是由于Linux内核的缓存机制，其中的内容还会被缓存到内存上，造成一定的内存空间浪费。而initramfs本身就是一个[tmpfs](https://zh.wikipedia.org/wiki/Tmpfs)的[RAM disk](https://zh.wikipedia.org/wiki/RAM_disk)，拥有最小化的设计，绕过了缓存机制，也消除了多余的内存占用。[[2\]](https://zh.wikipedia.org/wiki/Initrd#cite_note-2)
>
> initramfs的生成方式也远比initrd简单。对于initramfs，只需 `geninitramfs() { cd "$1"; find . -depth| cpio -o -H newc | ${3-cat}> "$2"; }` 就可以利用[cpio](https://zh.wikipedia.org/wiki/Cpio)生成这样一个文件，同时使用一些程序进行压缩（通过额外的管道实现，其中使用cat仅用于无压缩时转发输出，可省去）。对于initrd，则涉及生成一定大小的空文件，然后创建文件系统，挂载并添加文件等等诸多步骤。

##### initrd

整体来说，通过initrd，可以降低kernel image本身的设计复杂度，把很多开机启动代码从内核转移到用户空间。

initrd 字面上的意思就是"boot loader initialized RAM disk"，换言之，这是一块特殊的RAM disk，在载入Linux kernel 前，由boot loader予以初始化，启动过程会优先执行initrd的init程序，initrd完成阶段性目标后，kernel 会挂载真正的root file system ，并执行/sbin/init 程式。

首先，由kernel在内核空间完成与硬件相关的初始化工作，在适当的时机点，kernel读取并挂载initrd，切换到用户空间，以执行存放于RAM disk 中的init 程序（这需要完整的c运行时)。

第一阶段的initrd 步入尾声后，再回到kernel mode，initrd 所在的内存空间会被释放，之后就是执行真正的rootfs 中的init 程式。

##### initramfs

initrd对kernel来说，本身是个真实的block device。而block device需要在某个文件系统下工作，如ext2。ext2系统对于initrd来说过于复杂了。

基于这些想法，Linus Torvalds 做了ramfs，随后在其他核心开发者的改进下，成为tmpfs，而initramfs 就是建构于tmpfs的基础上。

initrd在执行结束后，会释放内存空间。而在Linux 2.6引入initramfs的设计后，一开机，kernel就执行位于initramfs中的/init，作为PID=1的init process，之后通过通过chroot重定位根文件系统。




### 为什么需要initrd/initramfs?

首先现在linux已经使用initramfs来代替initrd。首先我们要知道内核的功能有一项叫做文件管理，也就是说文件系统是由内核来进行管理的。

linux启动需要加载内核文件，现在有个问题，加载内核文件需要我们能够使用文件系统，但使用文件系统又需要内核，这样就形成一个套娃循环。

为了打破这个循环，我们添加一个虚拟内存文件系统，也就是initramfs。通过这个虚拟的内存文件系统，我们将真正的内核加载进内存，之后再去加载真正的根文件系统，然后切换文件系统。这样完成基本的内核加载，因此initramfs是不可或缺的，也是无法写入内核的。
