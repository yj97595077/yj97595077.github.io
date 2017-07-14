---
title: QEMU and KVM
date: 2017-07-14 18:40:56
tags: [QEMU, KVM]
categories: virtualization
---

### 1. QEMU与KVM关系
[Linux入门学习教程：虚拟机体验之QEMU篇](http://www.linuxidc.com/Linux/2015-03/114461.htm)
[Linux入门学习教程：虚拟机体验之KVM篇](http://www.linuxidc.com/Linux/2015-03/114462.htm)

QEMU可以模拟一个虚拟机的所有设备，包含CPU，内存，硬盘，网卡等，但是由于是单纯用户态的模拟，CPU和内存也要经过宿主操作系统翻译后才能和硬件说上话，所以性能就非常差了。为了解决这个问题，QEMU一般不单独使用，都是和KVM在一起使用，KVM会插入到宿主机的内核里，解决CPU和内存性能差的问题。引入kvm之后，其实还是QEMU作为主程序在操作整个虚拟化的流程，kvm只在其中干增强性能的部分，所以，所有的命令基本和单纯只有QEMU的时候一样，最多就是再加上--enable=kvm罢了。kvm没有单独可以调用的命令，你man它就可以看出来，kvm命令就是QEMU带上--enable=kvm的包装。

### 2. QEMU命令
QEMU的命令就如下几种，可以man一下看具体用法
qemu-img：QEMU磁盘的
qemu-nbd：QEMU磁盘挂载到宿主机的
qemu-system-x86_64：QEMU主程序
qemu-system-x86_64 --enable=kvm：这个命令就是kvm命令

### 3. Libvirt
[Libvirt学习总结](http://blog.csdn.net/gaoxingnengjisuan/article/details/9674315)
[Libvirt 虚拟化库剖析](http://www.ibm.com/developerworks/cn/linux/l-libvirt/)

Libvirt是管理虚拟机和其他虚拟化功能，比如存储管理，网络管理的软件集合。它包括一个API库，一个守护程序（libvirtd）和一个命令行工具（virsh）。还可以通过virt-manager，启动libvirt的图形界面。
没有libvirt的话，几乎所有的虚拟机操作都是直接用qemu-kvm命令行工具（qemu-system-x86_64）来完成的，其中有很多各种各样的配置参数，这些参数对于新手来说是难以记忆和熟练配置。libvirt对qemu-kvm命令进行了封装和功能增强，从而提供了比原生的qemu-kvm命令行更加友好、高效的用户交互接口。
![](http://ogd81e9re.bkt.clouddn.com/libvirt-manage-hypervisors.jpg)

### 4. virsh
virsh是虚拟shell的缩写，提供一种从xml文件创建管理虚拟机的方法，简单方便高效。

* `virsh list`
    显示本地活动虚拟机
* `virsh list –all`
    显示本地所有的虚拟机（活动的+不活动的）
* `virsh define ubuntu.xml`
    通过配置文件定义一个虚拟机，让libvirt认识这个XML，这个xml的名字不重要，libvirt会解析并且记录xml文件里面的信息，如果没有还会生成对应的uuid等。修改配置后需要先undefine，然后再define，undefine的是domain name，define的是xml文件。
* `virsh start ubuntu`
    启动名字为ubuntu的非活动虚拟机，这个名字不是xml的名字，而是xml里面的name选项的名字，当然也可以用uuid来启动。启动之后会生成一个id，也被libvirt记录下来。
* `virsh create ubuntu.xml`
    创建虚拟机，并且立刻成为活动主机。如果这个xml配置文件已经被define过，根据xml文件里的name，uuid等信息，会报错冲突。如果这个xml文件没有被define过，这个操作会创建一个临时虚拟机，无法autostart，宿主机一重启或者虚拟机一shutdown，destroy等，虚拟机再也无法找到，需要重新添加。
* `virsh suspend ubuntu`
    暂停虚拟机
* `virsh resume ubuntu`
    恢复暂停的虚拟机
* `virsh shutdown ubuntu`
    正常关闭虚拟机
* `virsh destroy ubuntu`
    强制关闭虚拟机，相当于拔电源。这时候又分两种情况如下：
    * 如果此虚拟机先前是先define后start的，这个操作虚拟机并未从libvirt移除，彻底移除还需要undefine。
    * 如果此虚拟机是由create创建的临时虚拟机，这个操作会吧虚拟机彻底消灭，libvirt会再也不认识它。
* `virsh undefine ubuntu`
    这后面的参数可以跟name,uuid或者id(如果running的话)，让libvirt把这个虚拟机不认识。也有两种情况如下：
    * 如果关机状态下，相当于删除了虚拟机
    * 如果running状态下，相当于把这个虚拟机变成了临时的，只要一shutdown，destroy，虚拟机立马就消失了。
* `virsh dominfo ubuntu`
    显示虚拟机的基本信息
* `virsh domname 2`
    显示id号为2的虚拟机名
* `virsh domid ubuntu`
    显示虚拟机id号
* `virsh domuuid ubuntu`
    显示虚拟机的uuid
* `virsh domstate ubuntu`
    显示虚拟机的当前状态
* `virsh dumpxml ubuntu`
    显示虚拟机的当前配置文件，可能和定义虚拟机时的配置不同，因为当虚拟机启动时，需要给虚拟机分配id号、uuid、vnc端口号等等。
* `virsh setmem ubuntu 512000`
    给不活动虚拟机设置内存大小
* `virsh setvcpus ubuntu 4`
    给不活动虚拟机设置cpu个数
* `virsh edit ubuntu`
    编辑配置文件，一般是在刚定义完虚拟机之后。

### 5. XML文件
[Domain XML format](https://libvirt.org/formatdomain.html)
[KVM虚拟机的xml配置文件](http://blog.csdn.net/liukuan73/article/details/46049579)
[libvirt 启动虚拟机xml配置文件](http://www.cnblogs.com/yanghuahui/archive/2013/05/08/3067676.html)

