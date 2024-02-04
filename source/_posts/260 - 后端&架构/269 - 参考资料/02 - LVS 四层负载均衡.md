---
title: LVS 四层负载均衡
date: 2024-02-04
categories:
  - 后端&架构
tags:
  - LVS
  - 四层负载均衡
published: true
---
## OSI七层和TCP/IP四层模型
![](https://raw.githubusercontent.com/BaihlUp/Figurebed/master/blog/20230206163904.png)

## 四层和七层负载均衡的定义
四层：所谓四层就是基于IP+端口的负载均衡
七层：所谓七层就是基于应用层（URL等）信息的负载均衡

七层负载均衡时，数据包在经过TCP/IP内核协议栈后会进入用户空间，然后根据端口号交给不同的用户程序进行处理，如下图：

![](https://raw.githubusercontent.com/BaihlUp/Figurebed/master/blog/20230206163932.png)

四层负载均衡不会进入用户空间，在内核空间处理完就转发出去。

![](https://raw.githubusercontent.com/BaihlUp/Figurebed/master/blog/20230206163950.png)

### 四层和七层负载均衡对比

| 四层负载均衡 | 七层负载均衡 |
| --- | --- |
| 在大型站点使用，接入层最前端 | 中小站点 |
| 结合四层+七层使用 | 只使用七层负载均衡 |

**优缺点**
- 四层比七层可以承载更大的并发量，大型站点使用
- 七层可以实现更为复杂的负载均衡控制，比如：基于URL、基于Session、动静分离等
- 七层会占用大量的CPU事件，更耗费性能，所以承载的并发量小


## LVS介绍
### LVS的组成

![](https://raw.githubusercontent.com/BaihlUp/Figurebed/master/blog/20230206164009.png)

![](https://raw.githubusercontent.com/BaihlUp/Figurebed/master/blog/20230206164029.png)

### LVS相关术语
- DS Director Server 目标服务器，即负载均衡器
- RS Real Server 真实服务器，即后端服务器
- VIP 直接面对用户的IP地址，通常为公网IP地址
- DIP Director Server IP 主要用于和内部主机通信的IP地址
- RIP Real Server IP 后端真实服务器的IP地址
- CIP Client IP 客户端IP

### LVS类型
- NAT 修改目标IP地址为后端的RealServer的IP地址
- DR 修改目标MAC地址为后端的RealServer的MAC地址
- TUNNEL 较少使用，常用与于异地容灾 

## LVS NAT模型原理
NAT是Network Address Translation 网络地址转换

LVS NAT：指修改目标IP地址为挑选出新的RS的IP地址，即请求进入负载均衡器时做DNAT，响应出负载均衡器时做SNAT
**LVS NAT模型图解：**

![](https://raw.githubusercontent.com/BaihlUp/Figurebed/master/blog/20230206164051.png)

**LVS NAT的特性：**
1. RS必须使用私有地址，并且将网关指向DS的DIP（否则数据包无法响应）
2. RIP和DIP必须为同一网段内
3. 支持端口映射（DS可以设为80映射到RS的8080）
4. RS可以使用任意操作系统，例如Linux、UNIX、Windows等
5. 请求和响应报文都要经过DS，高负载场景中，DS易成为瓶颈

### ipvsadm的使用
ipvsadm是可以在用户空间给ipvs编写规则，常用的命令参数如下：
1. 集群服务相关
```
-A 添加集群服务
-E 修改集群服务
-D 删除集群服务
—s 指定调度算法，例如rr/wrr/lc/wlc等
```
 
2. RS相关
```
-a 向指定的集群服务添加RS
-r 指定RS的IP地址，包含IP:PORT
-g|-i|-m 指定LVS类型，分别为DR|TUN|NAT
-w 指明RS的权重
-e 修改指定的RS属性
-d 删除RS
```

3. 管理相关
```
-C 清空集群服务
-L 查看IPVS规则
-Z 计数器清零
-S 使用ipvsadm -S保存规则到磁盘
-R 使用ipvsadm -R从磁盘载入规则
```

## LVS DR模型
DR：Direct Routing直接路由
LVS DR：将请求报文的目的MAC地址设定为选出来的RS的MAC地址，即做MAC地址转换

![](https://raw.githubusercontent.com/BaihlUp/Figurebed/master/blog/20230206164112.png)

### arp内核参数

在配置DR模型时，RS上会配置VIP地址，但是不能被客户机学习到，所以要修改内核arp参数，具体修改的arp内核参数如下：
arp_ignore 控制系统在收到外部的arp请求时，是否要返回arp响应
arp_announce 控制系统在对外发送arp请求时，如何选择arp请求数据包的源IP地址

- arp_ignore
  0 响应任意网卡上接收到的对本机IP地址的arp请求
  1 只响应目的IP地址为接收网卡所配置IP地址的arp请求
  2 只响应目的IP地址为接收网卡所配置IP地址的arp请求，并且arp请求的源IP必须和接收网卡同网段

- arp_announce
    0 允许使用任意网卡上的IP地址做为arp请求的源IP
    1 尽量避免使用不属于该发送网卡子网的本地地址做为发送arp请求的源IP地址
    2 忽略IP数据包的源IP地址，选择该网卡上最合适的本地地址做为arp请求的源IP地址

### LVS DR的特性
1. Direct 和 RS的所有节点必须位于同一物理网络中
2. RS不能将网关指向Director的DIP
3. 不支持端口映射
4. RS在lo接口上配置VIP
5. 请求报文经由DS转发，但响应必须不能经过DS（由于响应不用经过DS，所以性能比NAT更高）

## DR和NAT模型对比
### NAT优缺点
1. NAT模型支持任意操作系统，配置简单
2. NAT模型适用小规模集群环境，后端RS通常在5-20个
3. 扩展有限，并发大场景下，DS易成为性能瓶颈

### DR优缺点
1. 配置复杂
2. 客户端请求必须路由至DS，在路由器上配置静态地址绑定时，未必有权限
3. 由于只有请求包经过DS，因此性能很高，生产环境广泛使用
4. 只支持部分操作系统
5. DS和RS必须在同一物理网段内，不能有路由器隔离
6. 适用于大并发业务环节，此模型多为互联网公司生产首选


## 搭建Nginx集群

## 搭建MySQL集群

![](https://raw.githubusercontent.com/BaihlUp/Figurebed/master/blog/20230206164133.png)

![](https://raw.githubusercontent.com/BaihlUp/Figurebed/master/2024/2D41D180-F9CF-4DC6-B2B1-BC97BDF9B2CE.png)

![](https://raw.githubusercontent.com/BaihlUp/Figurebed/master/2024/C72EB368-B241-4B77-BDDB-50D579CD6239.png)

- 技术要点总结
1. Real Server不能将网关指向Director的DIP
2. RS上关闭ARP响应功能（arp_ignore）
3. RS响应数据包必须使用lo上配置的VIP（arp_announce+强制路由）
4. VIP在DS上配置到实体网卡别名上，在RS上则配置到lo别名上


### 配置DS的自动化脚本
```bash
#!/bin/bash
#

VIP="192.168.184.10"
NM="255.255.255.0"
PORT="3306"
RS1="192.168.184.50"
RS2="192.168.184.60"

case $1 in
    start)
        /usr/sbin/ifconfig ens33:0 $VIP netmask $NM up
        route add -host $VIP dev ens33:0
        ipvsadm -A -t $VIP:$PORT -s wrr
        ipvsadm -a -t $VIP:$PORT -r $RS1:$PORT -g -w 1
        ipvsadm -a -t $VIP:$PORT -r $RS2:$PORT -g -w 1
        ;;
    stop)
        /usr/sbin/route del $VIP
        /usr/sbin/ifconfig ens33:0 down
        ipvsadm -C
        ;;
    *)
        echo "Usage: sh $0 {start|stop}"
        ;;
esac
```

### 配置RS的自动化脚本
```bash
#!/bin/bash

VIP="192.168.184.10"
NM="255.255.255.255"

case $1 in
    start)
        echo 1 > /proc/sys/net/ipv4/conf/all/arp_ignore
        echo 1 > /proc/sys/net/ipv4/conf/ens33/arp_ignore
        echo 2 > /proc/sys/net/ipv4/conf/all/arp_announce
        echo 2 > /proc/sys/net/ipv4/conf/ens33/arp_announce

        /usr/sbin/ifconfig lo:0 $VIP netmask $NM broadcast $VIP up
        /usr/sbin/route add -host $VIP dev lo:0
        ;;
    stop)
        echo 0 > /proc/sys/net/ipv4/conf/all/arp_ignore
        echo 0 > /proc/sys/net/ipv4/conf/ens33/arp_ignore
        echo 0 > /proc/sys/net/ipv4/conf/all/arp_announce
        echo 0 > /proc/sys/net/ipv4/conf/ens33/arp_announce
        /usr/sbin/route del $VIP
        /usr/sbin/ifconfig lo:0 down
        ;;
    *)
        echo "Usage: sh $0 {start|stop}"
        ;;
esac
```

1. route命令详解
2. 配置网卡别名

## 虚拟路由冗余协议VRRP
下图是VRRP的示例图：

![](https://raw.githubusercontent.com/BaihlUp/Figurebed/master/blog/20230206164214.png)


VRRP实现一个虚拟路由器的功能，在后边的Master和Backup两台设备上可以进行配置，当Master出现故障后，可以立即转移到Backup上继续提供服务

核心功能如下：
1. 虚拟路由器：有一个Master和多个Backup组成
2. Master路由器 ： 实际承载报文转发的节点，主节点
2. Backup路由器：主节点故障后转移节点，备用节点
3. 优先级：VRRP根据优先级确定主备节点角色（通过配置优先级的值）
4. 虚拟IP地址：虚拟路由对外提供服务的IP地址
5. IP地址拥有者：真实提供服务的节点，通常为主节点
6. 虚拟MAC地址：回应ARP请求时使用的虚拟MAC地址

### KeepAlived功能
KeepAlived就是VRRP的实现，具体使用如下：
1. 实现服务器的故障转移
2. 通常用于对负载均衡器做高可用

**KeepAlived架构图：**
![](https://raw.githubusercontent.com/BaihlUp/Figurebed/master/2024/0552EE1F-823B-4C71-BD12-565386BE1F6C.png)

- vrrp stack  vrrp协议的实现
- ipvs wrapper  为集群内的节点生成ipvs规则
- checkers 对集群内所有的RS做健康状态检测
- 控制组件  配置文件解析和加载

**KeepAlived适用场景：**
1. 高可用LVS，高可用服务
2. RS健康状态检测
3. 编写脚本实现服务启动/停止
4. 生成ipvs规则
5. 虚拟IP的转移

在部署环境时需要设置关闭下边的服务：
setenforce
firewalled
yum install keeplived