---
title: 01 - REUSEADDR 和REUSEPORT 对TCP连接的控制
date: 2024-02-05
categories:
  - 网络协议
tags:
  - SO_REUSEADDR、SO_REUSEPORT
published: false
---
## 参考资料
[REUSEADDR 与 REUSEPORT 到底做了什么？](https://zhuanlan.zhihu.com/p/492644204)

## REUSEADDR
SO_REUSEADDR 提供了一套复用地址的规则。对于 AF_INET 套接字，这意味着一个套接字可以绑定，除非有一个活动的监听套接字绑定到该地址。 当监听套接字使用特定端口绑定到 INADDR_ANY 时，就不可能为任何本地地址重新绑定到该端口。

对服务端，在创建socket后，对soket设置reuseaddr属性，可以实现同时复用ip+port

```bash
开启 REUSEADDR，并且调用 listen 使套接字称为监听状态，两个使用相同 port 进行bind，
结果如下：
    server1 ：127.0.0.1:30001, bind 成功
    server2 ：0.0.0.0:30001, bind 成功
```
如果ip+port完全相同则报错。

当作为服务端重启时，主动断开连接，会进入TIME_WAIT状态，此时启动时会报错，Address already in use，如果开启REUSEADDR，则可以正常快速重启。

## REUSEPORT
对于TCP套接字而言，这个选项通过每个监听线程使用不同的 listen fd 来改进accpet的负载分配。相对于传统的做法如只有一个 accept 线程在处理连接，或者是多个线程使用同一个listen fd 进行 accept，REUSEPORT 有助于负载均衡的改进。

```bash
不开启 REUSEADDR，开启 REUSEPORT。
server1 ：127.0.0.1:30001, bind 成功
server2 ：127.0.0.1:30001, bind 成功
```
开启REUSEPORT，可以实现两个进程监听和bind相同的ip+port，此时需要内核负载均衡方式选择处理请求的进程。

## 在 Nginx中不适用 reuseport 与 使用 reuseport的差异
- 未开启reuseport

![](https://raw.githubusercontent.com/BaihlUp/Figurebed/master/2024/20240205172646.png)

- 开启reuseport

![](https://raw.githubusercontent.com/BaihlUp/Figurebed/master/2024/20240205172807.png)

Nginx启动了16个work process，开启reuseport后，每个进程都会监听 端口。每个进程都维护自己的监听套接字，新建连接可以通过内核进行负载均衡分发，如下图：
![](https://raw.githubusercontent.com/BaihlUp/Figurebed/master/2024/20240205173127.png)




