---
title: Nginx支持国密实践
date: 2023-12-03
categories:
  - 信息安全
tags: 
published: false
---
## 1 简介

我国密码行业中有两个主要的通信协议相关的技术标准：
- [《信息安全技术 传输层密码协议(TLCP)》](https://openstd.samr.gov.cn/bzgk/gb/newGbInfo?hcno=778097598DA2761E94A5FF3F77BD66DA)：即`GB/T 38636-2020`
- [《SSL VPN 技术规范》](https://std.samr.gov.cn/hb/search/stdHBDetailed?id=8B1827F20288BB19E05397BE0A0AB44A)：即`GM/T 0024-2014`
- 上述协议关联标准：如数字证书标准`GB/T 20518`, SM2密码算法标准`GB/T 35275`等

>《SSL VPN技术规范》作为`行业标准`，部分内容已上升为《信息安全技术 传输层密码协议(TLCP)》`国密标准`。

国密TLS，又称`GMTLS1.1`，`GMTLS1.1`中的`1.1`代表国密TLS的版本号: _0x0101_。众所周知，目前TLS协议支持的版本：
```bash
	VersionTLS10 = 0x0301
	VersionTLS11 = 0x0302
	VersionTLS12 = 0x0303
	VersionTLS13 = 0x0304
```

目前主流的TLS软件程序实现中，通信实体在建立TLS连接时，会`优先`协商使用`最大版本号TLS协议` _(版本号越大意味着安全性越高、性能越好)_，因此`VersionGMSSL = 0x0101` 版本号的设定增加了软件支持的复杂度，这也是阻碍GMTLS1.1广泛应用的一个重要原因。

TLCP 显著的特点是将 TLS 协议中使用的数字证书拆分成了加密和签名两种用途的证书，加密证书和签名证书以及对应私钥均需要进行配置使用，所以 TLCP 也俗称“国密双证书”协议。

## 2 基于铜锁（Tongsuo）部署服务端

编写nginx基于tongsuo进行编译和部署


## 3 国密证书生成



## 4 客户端工具



## 5 抓包分析


