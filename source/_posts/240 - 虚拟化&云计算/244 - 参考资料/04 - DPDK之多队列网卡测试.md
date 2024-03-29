---
title: DPDK之多队列网卡测试
date: 2024-03-29
categories:
  - 虚拟化&云计算
tags:
  - DPDK
published: true
---
## 0 背景

基于环境 VMware+Ubuntu 虚拟机环境，在DPDK接管多队列网卡后，每个网卡数据接收情况。
1. 配置多队列网卡
2. 多队列网卡测试

## 1 添加多个网卡

1. 在vmware环境中对应虚拟机的虚拟机设置中，**增加网络适配器（一个或者多个），对应网卡的个数**。
![](https://raw.githubusercontent.com/BaihlUp/Figurebed/master/2024/20240329152304.png)

2. 启动虚拟机，查看网卡信息

当前生效的只有一个网卡，是桥接模式的第一个网卡：
```bash
ubuntu@ubuntu:~$ service networking restart    #不同的linux环境重启网络命令是不同的
==== AUTHENTICATING FOR org.freedesktop.systemd1.manage-units ===
Authentication is required to restart 'networking.service'.
Authenticating as: ubuntu,,, (ubuntu)
Password: 
==== AUTHENTICATION COMPLETE ===
ubuntu@ubuntu:~$ ifconfig
ens33     Link encap:Ethernet  HWaddr 00:0c:29:b2:2f:c4  
          inet addr:192.168.50.56  Bcast:192.168.50.255  Mask:255.255.255.0
          inet6 addr: fe80::20c:29ff:feb2:2fc4/64 Scope:Link
          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
          RX packets:7546 errors:0 dropped:0 overruns:0 frame:0
          TX packets:524 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000 
          RX bytes:563423 (563.4 KB)  TX bytes:86418 (86.4 KB)

ens38     Link encap:Ethernet  HWaddr 00:0c:29:b2:2f:ce  
          inet addr:192.168.11.155  Bcast:192.168.11.255  Mask:255.255.255.0
          inet6 addr: fe80::20c:29ff:feb2:2fce/64 Scope:Link
          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
          RX packets:14 errors:0 dropped:0 overruns:0 frame:0
          TX packets:14 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000 
          RX bytes:1976 (1.9 KB)  TX bytes:1566 (1.5 KB)

ens39     Link encap:Ethernet  HWaddr 00:0c:29:b2:2f:d8  
          inet addr:192.168.11.156  Bcast:192.168.11.255  Mask:255.255.255.0
          inet6 addr: fe80::20c:29ff:feb2:2fd8/64 Scope:Link
          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
          RX packets:7 errors:0 dropped:0 overruns:0 frame:0
          TX packets:9 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000 
          RX bytes:986 (986.0 B)  TX bytes:1262 (1.2 KB)

lo        Link encap:Local Loopback  
          inet addr:127.0.0.1  Mask:255.0.0.0
          inet6 addr: ::1/128 Scope:Host
          UP LOOPBACK RUNNING  MTU:65536  Metric:1
          RX packets:160 errors:0 dropped:0 overruns:0 frame:0
          TX packets:160 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1 
          RX bytes:11840 (11.8 KB)  TX bytes:11840 (11.8 KB)
```


## 2 配置网卡支持多队列

多队列网卡：就是网卡的DMA队列有多个，有自己对队列的分配机制。
1. 修改虚拟机配置，使支持多队列（关闭虚拟机）

```bash
#在Ubuntu 64 位.vmx 中打开，修改ethernet1配置，把e1000 改为 vmxnet3
#如果没有wakeOnPcktRcv 增加就好
#这里我的ethernet1 对应的是环境上的eth0
ethernet1.virtualDev = "vmxnet3"
ethernet1.wakeOnPcktRcv = "TRUE"
```

2. 修改虚拟机配置，处理器个数设为4
3. 查看是否支持多队列网卡

使用cat /proc/interrupts 命令可以查看多队列网卡的效果
```bash
ubuntu@ubuntu:~$ cat /proc/interrupts |grep eth0
  57:         13          0          4          0   PCI-MSI 1572864-edge      eth0-rxtx-0
  58:          2          0          0          0   PCI-MSI 1572865-edge      eth0-rxtx-1
  59:          6          0          0          0   PCI-MSI 1572866-edge      eth0-rxtx-2
  60:          0          0          0          0   PCI-MSI 1572867-edge      eth0-rxtx-3
  61:          0          0          0          0   PCI-MSI 1572868-edge      eth0-event-4
```
可以看到，配置了4个处理器，这里的队列个数为4个。

4. 使用cat /proc/cpuinfo，可以查看支持的cpu个数（processor+1）
```bash
#可以自己查看细节
ubuntu@ubuntu:~$ cat /proc/cpuinfo |grep processor
processor	: 0
processor	: 1
processor	: 2
processor	: 3
```

## 3 配置Nginx环境
1. 安装Nginx
2. 修改nginx配置环境，设置cpu亲和性

```bash
#在nginx配置文件中修改/增加如下  支持4个进程对应4个cpu
worker_processes  4;
worker_cpu_affinity 0001 0010 0100 1000;
```
3. 绑定中断和cpu

在/proc/interrupts 文件中查看对应的多队列网卡的中断号，可以看到这里是56-59
```bash
ubuntu@ubuntu:/usr/local/nginx$ cat /proc/interrupts |grep eth0
  56:         14          0          0         14   PCI-MSI 1572864-edge      eth0-rxtx-0
  57:          3          0          0          0   PCI-MSI 1572865-edge      eth0-rxtx-1
  58:          4          0          0          0   PCI-MSI 1572866-edge      eth0-rxtx-2
  59:          2          0          0          0   PCI-MSI 1572867-edge      eth0-rxtx-3
  60:          0          0          0          0   PCI-MSI 1572868-edge      eth0-event-4
```

4. 手动对中断号进行cpu的绑定
```bash
root@ubuntu:/proc/irq# cat /proc/irq/56/smp_affinity
00000000,00000000,00000000,00000008
root@ubuntu:/proc/irq# cat /proc/irq/57/smp_affinity
00000000,00000000,00000000,00000008
root@ubuntu:/proc/irq# cat /proc/irq/58/smp_affinity
00000000,00000000,00000000,00000008
root@ubuntu:/proc/irq# cat /proc/irq/59/smp_affinity
00000000,00000000,00000000,00000008
root@ubuntu:/home# echo 00000001> /proc/irq/56/smp_affinity
root@ubuntu:/home# echo 00000002> /proc/irq/57/smp_affinity
root@ubuntu:/home# echo 00000004> /proc/irq/58/smp_affinity
root@ubuntu:/home# echo 00000008> /proc/irq/59/smp_affinity
#查看修改后的
root@ubuntu:/home# cat /proc/irq/56/smp_affinity
00000000,00000000,00000000,00000001
root@ubuntu:/home# cat /proc/irq/57/smp_affinity
00000000,00000000,00000000,00000002
root@ubuntu:/home# cat /proc/irq/58/smp_affinity
00000000,00000000,00000000,00000004
root@ubuntu:/home# cat /proc/irq/59/smp_affinity
00000000,00000000,00000000,00000008
#可以通过查看/proc/irq/56/smp_affinity_list  最后生效的，其实是对应的smp_affinity_list 其实是绑定的cpu号，可以多个
ubuntu@ubuntu:~$ cat /proc/irq/56/smp_affinity_list 
0
ubuntu@ubuntu:~$ cat /proc/irq/57/smp_affinity_list 
1
ubuntu@ubuntu:~$ cat /proc/irq/58/smp_affinity_list 
2
ubuntu@ubuntu:~$ cat /proc/irq/59/smp_affinity_list 
3

#如果有多于4个的，是继续如下配置的
root@ubuntu:/home# echo 00000010> /proc/irq/60/smp_affinity
root@ubuntu:/home# echo 00000020> /proc/irq/61/smp_affinity
root@ubuntu:/home# echo 00000040> /proc/irq/62/smp_affinity
root@ubuntu:/home# echo 00000080> /proc/irq/63/smp_affinity
```
## 4 多队列网卡测试

使用wrk性能测试工具对多队列网卡进行测试。
查看多队列网卡中中断产生所在cpu上的情况：主要看eth0,(中断号56，57，58，59) 。
可以从现象看出，56号中断数据在最后测试时，数据在cpu0, 57在cpu1, 58在cpu2, 59在cpu3 （cpu3上中断次数过多，是因为测试前的数据）
```bash
root@ubuntu:/usr/local/nginx# cat /proc/interrupts |grep eth
  17:       5726          0          0       4420   IO-APIC   17-fasteoi   ehci_hcd:usb1, ioc0, eth2
  19:        405          0          0       3824   IO-APIC   19-fasteoi   eth1
  56:      79311          0          0     223321   PCI-MSI 1572864-edge      eth0-rxtx-0
  57:          3      38030          0      81500   PCI-MSI 1572865-edge      eth0-rxtx-1
  58:          7          0      38853      81592   PCI-MSI 1572866-edge      eth0-rxtx-2
  59:          7          0          0     116604   PCI-MSI 1572867-edge      eth0-rxtx-3
  60:          0          0          0          0   PCI-MSI 1572868-edge      eth0-event-4

```

一次中断，保证一个包的完整接收，但是并不是限定了连接就一致在这个cpu上。

# 5 其他
## 5.1 多队列网卡的发送和接收
网卡内部有多个队列，与CPU进行绑定，通过PCI总线进行数据分发。

**接收数据：**
1. 数据先进行hash
2. 根据hash后的结果，放入对应的队列中
3. 触发中断，由CPU进行处理：使用**NAPI**对单纯的硬件中断进行优化，有中断触发,数据就一直读， 因此rte_eth_rc_burst读出来得到数据是多个数组（多个队列即多个NAPI）。

**发送数据：**
1. 通过队列，把send()接口和网卡发送进行拆分，对应的QDisc就是队列，可以是多个队列。
2. 多个QDIsc队列设计方案：FIFO，可以四元组检索对应放数据。 （可以设置）

## 5.2 DPDK与多队列网卡
配置：rte_eth_dev_configure() ：对网卡进行配置
启动： rte_eth_rx_queue_setup() ： 对RX队列的配置
​            rte_eth_tx_queue_setup() ： 对TX队列的配置
接收： rte_eth_rx_burst()
发送：rte_eth_tx_burst()

