---
title: DPDK 原理剖析
date: 2024-03-29
categories:
  - 虚拟化&云计算
tags:
  - DPDK
published: false
---
## 0 参考资料

零声教育直播课

## 1 DPDK原理和生态

## 2 DPDK底层原理剖析
### 2.1 dpdk零拷贝机制
![](https://raw.githubusercontent.com/BaihlUp/Figurebed/master/2024/7E72CCEF-15E6-4E8A-BC5C-D0532A258C2E.png)

内核协议栈需要两次拷贝，dpdk通过dma映射，实现应用程序直接读取网卡数据。

### 2.2 如何把网卡数据映射到内存

uio
vfio

### 2.3 如何把网络数据写入内核

基于kni，运行dpdk中的kni接口后，生成了一个veth0网口（基于net_device），通过veth0网口，把dpdk中的数据写入内核。
![](https://raw.githubusercontent.com/BaihlUp/Figurebed/master/2024/A7362C12-8BA3-48AF-B319-308FF274D9CC.png)
ioctl_create --> alloc_netdev

kni实现：
1. 通过/dev/kni字符设备
2. vEth0 --> net_device
3. dpdk申请了一个环形队列（ring-buffer），把数据写入队列（rx、tx）
3. 调用内核接口把数据写入内核协议栈

> ring_buffer可以实现多线程无锁

### 2.4 如何解析数据包


## 3 netmap和dpdk
![](https://raw.githubusercontent.com/BaihlUp/Figurebed/master/2024/51363E86-E2C6-42EB-BF80-C1E680C3A298.png)

netmap
dpdk官方支持更全，实现了更多功能，有上层各种框架，比如 nff

## 4 DPDK如何支持千万并发
如果要实现一个C10M的系统意味着什么？
![](https://raw.githubusercontent.com/BaihlUp/Figurebed/master/2024/A3882B77-15DD-4C5A-AD91-3C1A1FB4FABB.png)
如果是C10K、C100K通过使用IO多路复用技术可能还能实现，但是到C10M，就无法再实现了。
在实现C10M时的限制有哪些？
1. CPU、内存、磁盘、网卡这些硬件不是主要限制因素，因为可以使用堆叠硬件达到
2. 主要限制在软件上（操作系统），传统使用Linux内核协议栈，每次读写数据会调用系统调用，产生软中断，数据从内核态到用户态的copy等，在C10M条件下，每秒要达到千万级别，内核协议栈无法做到。

DPDK如何解决的？
1. DPDK通过加载UIO驱动接管网卡，网卡的数据不再交给内核协议栈处理，直接由UIO处理到用户态，不需要再出现系统调用软中断。
2. 通过DMA处理网卡数据包

dpdk的用户态协议栈：lwip、bsd4.4
基于netmap的协议栈：mtcp、ntytcp
