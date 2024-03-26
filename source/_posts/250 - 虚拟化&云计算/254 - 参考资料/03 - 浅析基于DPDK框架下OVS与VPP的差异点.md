---
title: 【转载】浅析基于DPDK框架下OVS与VPP的差异点
date: 2024-03-26
categories:
  - 虚拟化&云计算
tags:
  - DPDK
published: true
---
# 1 概述

Data Plane Development Kit简称DPDK，是一套将从网卡收上来的数据包直接映射到用户空间的开发套件，基于此许多大公司都搭建了自己的业务应用框架，如虚拟交换机、虚拟路由器等。其中影响比较大的就是OVS和VPP，本文从这两个系统的性能表现及开发难度两个维度进行详细分析，给出作者对这两个业务框架的一点认识，希望能给学习这两个框架的人有所帮助。

## 1.1 DPDK简介

DPDK原本是基于 Intel 处理器架构下，提供库函数和驱动支持用户空间高效数据包处理的开源套件，即DPDK绕过了系统的内核协议栈对报文的处理过程，报文的收发与处理在用户空间实现。因此对于内核而言，DPDK的工作进程就是一个普通的用户态进程。DPDK的编译、连接和加载方式也跟普通程序没有什么差别。DPDK的这种特性区别于以通用性设计为目的的Linux系统，DPDK专注于高性能的处理网络应用中的数据包。现在这个套件也支持ARM及Power 8。

DPDK的典型框架如下图所示：

![](https://s.secrss.com/anquanneican/c26cd2d46122eeba33f9b520848368e0.jpg)

_图 1 DPDK架构简图_

从上图可以看出，DPDK将网卡收上来的报文跳过Linux Kernel，直接映射到用户空间并提供了库函数及驱动来支持用户进程对报文的控制。

## 1.2 OVS简介

OVS是Open vSwitch的简称，这是一个多核虚拟交换机平台框架，支持标准的管理接口，支持扩展的可编程接口，支持第三方的控制接入，开源Apache 2许可。

OVS由 Nicira Networks开发，主要由可移植的C语言实现。OVS支持包括Xen/XenServer、 KVM和VirtualBox等的多种Linux虚拟化技术。

OVS主要用于虚拟机环境，作为一个虚拟交换机。在这种虚拟化的环境中，一个虚拟交换机主要有两个作用：传递虚拟机之间的流量，以及实现虚拟机和外界网络的通信。

OVS内核模块实现了多个“DataPath”（类似于Bridge），每个都可以有多个“vPorts”（类似于桥内的Port）。每个DataPath也通过关联一下流表（flow table）来设置操作，而这些流表中的流都是用户空间在数据包头和元数据的基础上映射的关键信息，一般的操作都是将数据包转发到另一个vPort上。当一个数据包到达一个vPort，内核模块所做的处理是提取其流的关键信息并在流表中查找这些关键信息。当有一个匹配的流时它执行对应的操作。如果没有匹配，它会将数据包送到用户空间的处理队列中（作为处理的一部分，用户空间可能会设置一个流用于以后碰到相同类型的数据包可以在内核中执行操作）。下图为OVS的架构图：

![](https://s.secrss.com/anquanneican/f14ce7dd358d5a6539743b8215753ff8.jpg)

_图 2 OVS的架构图_

1. **ovs-vswitchd模块**：OVS守护进程，OVS的核心部件，实现交换功能，和Linux内核兼容模块一起，实现基于流的交换（flow-based switching）。它和上层 controller 通信遵从 OPENFLOW 协议，它与 ovsdb-server 通信使用 OVSDB 协议，它和内核模块通过netlink通信，它支持多个独立的 datapath，它通过更改flow table 实现了绑定和VLAN等功能。
2. **ovsdb-server模块**：轻量级的数据库服务，主要保存了整个OVS 的配置信息，包括Ports，交换内容，VLAN等等。ovs-vswitchd 会根据数据库中的配置信息处理。它与 manager 和 ovs-vswitchd 交换信息使用了OVSDB(JSON-RPC)的方式。
3. **ovs-dpctl模块****：**一个工具，用来配置交换机内核模块，可以控制转发规则
4. **ovs-vsctl模块****：**主要是获取或者更改ovs-vswitchd 的配置信息，此工具操作的时候会更新ovsdb-server 中的数据库。
5. **ovs-appctl模块：**主要是向OVS 守护进程发送命令的，一般用不上。
6. **ovsdbmonitor模块：**GUI 工具来显示ovsdb-server 中数据信息。
7. **ovs-controller模块：**一个简单的OpenFlow 控制器
8. **ovs-ofctl模块：**用来控制OVS 作为OpenFlow 交换机工作时候的流表内容。
## 1.3 VPP简介

VPP是Vector Packet Processing的简称。据传VPP是Cisco 2002年开发的商用代码。提到VPP，不得不提到FD.io。Linux基金会于2016年2月11号创建了FD.io项目。Cisco后来将VPP代码的开源版本加入了该项目，目前VPP已成为该项目的核心。VPP运行于用户空间，支持多种收包方式，这其中也包括DPDK。

VPP有两个关键特性：

框架可扩展。

成熟的交换/路由功能。

VPP官方给出的特性列表如下：

![](https://s.secrss.com/anquanneican/46fb6b8b192d8d2ae9c62f29eb359b0d.jpg)

_图 3 VPP当前支持的特性列表_

从上图可以看出，VPP支持的特性非常丰富，包括典型的IPv4和IPv6协议栈，VPN、NAT，VxLAN等交换机和路由器具体的特性。

而VPP是通过graph node串联起来形成一条datapath来处理报文，类似于freebsd的netgraph。通过插件的形式引入新的graph node或者重新排列报文的graph node。将插件添加到插件目录中，运行程序的时候就会自动加载插件。另外插件也可以根据硬件情况通过某个node直接连接硬件进行加速。VPP平台可以用于构建任何类型的报文处理应用。比如负载均衡、防火墙、IDS、主机栈。也可以是一个组合，比如给负载均衡添加一个vSwitch。其运行图如下：

![](https://s.secrss.com/anquanneican/1426ca576812f1b7bd5a2f410c7c28d9.jpg)

_图 4 VPP运行图_

# 2 开发难度对比

OVS和VPP都可以采用DPDK的收发包框架，那么两者对于开发人员来讲，如何上手呢？哪个更容易呢？

## 2.1 OVS简易开发流程

目前Ovs的最新发行版版本为v2.7.0，根据Ovs的官方文档，由于Qemu早期版本不支持vhost-usertype，所以qemu版本必须在v2.1以上。虽然qemu版本更新很快，但是我们测试机器采用的是Centos7.1系统，Centos官方升级包中支持的qemu版本较低，所以我们使用了符合要求的较低版本Qemuv2.2以防止出现新的问题。Qemu编译安装时间比较长，需要提前去qemu官网下载对应版本，并安装。

**Step1:下载Ovs和Dpdk相关版本**

wgethttp://openvswitch.org/releases/openvswitch-2.5.0.tar.gz
wgethttp://www.dpdk.org/browse/dpdk/snapshot/dpdk-16.04.tar.gz

**Step2:清理环境，**因为Ovs+Dpdk，需要Ovs在编译时link到Dpdk的lib库，所以如果环境原来已经安装有Ovs，也需要先卸载掉，重新编译安装。

**Step3:编译Dpdk**

修改配置文件config/common_linuxapp，这里我们测试vhost模式，所以需要把下面两个配置项，配置为yes。
CONFIG_RTE_BUILD_COMBINE_LIBS=y
CONFIG_RTE_LIBRTE_VHOST=y

**Step 4:安装Dpdk**

新建一个安装目录 mkdir -p /usr/src/dpdk
make installT=x86_64-native-linuxapp-gcc DESTDIR=/usr/src/dpdk

**Step 5:编译相关模块**

cd /home/dpdk-16.04/rte_vhost/eventfd_link
make

**Step 6:安装ovs**

./boot.sh
./configure --with-dpdk=/home/ dpdk-16.04/x86_64-native-linuxapp-gcc CFLAGS="-g -O2 -Wno-cast-align"
make && make install

**Step7:安装相关内核模块**

modprobe uio
insmod /home/dpdk-16.04/x86_64-native-linuxapp-gcc/kmod/igb_uio.ko
insmod /home/dpdk-16.04/lib/librte_vhost/eventfd_link/eventfd_link.ko
## 2.2 VPP简易开发流程

在本文中，三个系统命名csp2s22c03，csp2s22c04和net2s22c05使用。csp2s22c03安装了VPP 的系统用于转发数据包，而系统csp2s22c04和系统net2s22c05则用于传递流量。这三个系统都配备了英特尔®至强®处理器E5-2699 v4 @ 2.20 GHz，两个插槽（每个插槽22个内核），并运行64位Ubuntu * 16.04 LTS。英特尔®以太网聚合网络适配器XL710 10/40 GbE用于连接这些系统。有关配置图，请参见图5和图6。

![](https://s.secrss.com/anquanneican/c1eab69ce87d34ab7bb5a03492b5405b.png)

_图 5 VPP在主机上运行，_

该主机通过40 GbE NIC连接到其他两个系统

![](https://s.secrss.com/anquanneican/6d74759f75d7873a7ddc225f34abc33a.png)

_图 6 TRex流量生成器将程序包发送到运行VPP的主机_

**>>>**

### 2.2.1 生成FD.io VPP二进制文件

本节中的说明描述了如何从FD.io构建VPP软件包。如果您想改用Debian * VPP软件包，请跳至下一部分。
使用中的管理员特权帐户csp2s22c03，我们将下载稳定版本的VPP（本教程中使用的版本为17.04），然后导航到该build-root目录以构建映像：

![](https://s.secrss.com/anquanneican/0ef8ffd7e87862a0f70f1f2a7278faab.png)

要使用调试符号构建映像：

![](https://s.secrss.com/anquanneican/c57d27db8b8f750d5fb4026c5bc99fe1.png)

配置VPP之后，可以fdio.1704使用src/vpp/conf/startup.conf配置文件从目录运行VPP二进制文件：

![](https://s.secrss.com/anquanneican/57cf004ec77d48595bb9d0c25086b9bc.png)

### 2.2.2 编译Debian VPP软件包

如果您更喜欢使用Debian VPP软件包，请按照以下说明进行构建：
![](https://s.secrss.com/anquanneican/785a00f051acf010645ad4cb26737bf3.png)

在此输出中：
- vpp是数据包引擎
- vpp-api-java是Java 绑定模块
- vpp-api-lua是Lua 绑定模块
- vpp-api-python是Python 绑定模块
- vpp-dbg是VPP的调试符号版本
- vpp-dev是开发支持（标头和库）
- vpp-lib是VPP运行时库
- vpp-plugins是插件模块

接下来，安装Debian VPP软件包。至少应安装VPP，vpp-lib和vpp-plugins软件包。我们将它们安装在机器上csp2s22c03：

![](https://s.secrss.com/anquanneican/8c48dd024bc345b5aa652a070ac35841.png)

验证VPP软件包是否已成功安装：

![](https://s.secrss.com/anquanneican/c994f9663a374d59235094911382a803.png)

### 2.2.3 配置VPP

在安装过程中，将创建两个配置文件：

/etc/sysctl.d/80-vpp.conf和/etc/vpp/startup.conf/startup.conf。

该/etc/sysctl.d/80-vpp.conf配置文件用于建立庞大的网页。

该/etc/vpp/startup.conf/startup.conf配置文件用于启动VPP。

# 3 性能对比

熟悉了OVS和VPP各自的基本开发流程，那么对于数据包处理来讲，最关心的性能在这两个架构下有什么样的表现？

#### 3.1 DPDK性能

下图是DPDK官方网站上测试的原始DPDK的包转发性能：

![](https://s.secrss.com/anquanneican/3842bbf9446c5a981692899aa8424a3d.jpg)

从上图我们可以看出DPDK在基于Intel Xeon处理器的E5-2695的表现还是非常不错的，物理网卡总吞吐为40G的情况下，64字节小包有不俗的表现。

## 3.2 OVS性能

![](https://s.secrss.com/anquanneican/888f415619df67e6733aff2138c41cf2.png)

intel的实验《Intel_ONP_Release_2.1_Performance_Test_Report_Rev1.0》，测出的数据和文档基本吻合。在ovs里配置流表和不配置流表，转发效果区别很大。默认流表：正常启动ovs，不做流表配置

配置流表：ovs-ofctl del-flows br-bond_virt （删除默认流表）
ovs-ofctl add-flow br-bond_virt in_port=1,dl_type=0x800,idle_timeout=0,action=output:2
ovs-ofctl add-flow br-bond_virt in_port=3,dl_type=0x800,idle_timeout=0,action=output:1

命令中的port编号解释：1代表dpdk物理口，2代表进入vm的tap/vhostuser，3代表vm转出来的tap/vhostuser，由于实验环境只有一个物理口，所以数据进出都是同一个物理口。
vm接收数据包实验，pc发包1400万pps

**实验1**：pc->nic->nic->ovs->tap->vm 配置流表和默认流表vm接收都是80万pps左右，差距不大
**实验2**：pc->nic->nic->ovs+dpdk->vhostuser->vm 默认流表时，vm收包90万pps，配置流表600万pps
ovs+dpdk看转发效率可以使用ovs-appctl dpif-netdev/pmd-stats-show，查看每个数据包的cpu周期，配置流表后，每个数据包大概是400个cpu周期，默认流表一般是几千个cpu周期。

## 3.3 VPP性能

下图是在Haswell x86 架构的E5-2698v3 2x16C 2.3GHz上测试，图中显示了12口10GE，16核，ipv4转发：

![](https://s.secrss.com/anquanneican/e6fbb61a9f6c87d6b34892803ad63169.jpg)

## 3.4 OVS和VPP性能对比

![](https://s.secrss.com/anquanneican/d378d788c093729f1308fbe4a30ac48f.jpg)

**从上图可以看出：**
1. 由于EMC的大小问题当流表超过8192时，OVS的扩展性就表现的不是很好了。
2. 经过NEANTC的调试，VPP接近线速。
3. VPP架构实现了加速的承诺，vxLAN/NSH在SFC的VTEP下。
4. 但由于样本采集有限，数据可能还有待进一步完善。

# 4 结论

![](https://s.secrss.com/anquanneican/d763cad91e447038968fedf575493b45.jpg)

