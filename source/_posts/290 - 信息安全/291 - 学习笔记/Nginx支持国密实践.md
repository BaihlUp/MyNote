---
title: Nginx支持国密实践
date: 2023-12-03
categories:
  - 信息安全
tags: 
published: false
---
# 1 简介

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

## 1.1 密钥种类

### 1.1.1 概述
采用非对称密码算法进行身份鉴别和密钥交换，身份鉴别通过后协商预主密钥，双方各自计算主密钥，进而推导出工作密钥。使用工作密钥进行加解密和完整性校验。

### 1.1.2 服务端密钥
服务端密钥为非对称密码算法的密钥对，包括签名密钥对和加密密钥对，其中签名密钥用于握手过程中服务端身份鉴别，加密密钥对用于预主密钥的协商。

### 1.1.3 客户端密钥
客户端密钥为非对称密码算法的密钥对，包括签名密钥对和加密密钥对，其中签名密钥用于握手过程中客户端身份鉴别，加密密钥对用于预主密钥的协商。

### 1.1.4 预主密钥
预主密钥（`pre_master_secret`）是双方协商生成的密钥素材，用于生成主密钥

### 1.1.5 主密钥

主密钥（`master_secret`）由预主密钥、客户端随机数、服务端随机数、常量字符串，经计算生成的 48 字节密钥素材，用于生成工作密钥。
### 1.1.6 工作密钥
工作密钥包括数据加密密钥和校验密钥。其中数据加密密钥用于数据的加密和解密,校验密钥用于数据的完整性计算和校验。发送方使用的工作密钥称为写密钥，接收方使用的工作密钥称为读密钥。

## 1.2 密码套件
**GMTLS1.1**中相关的`密码套件`（图片来自*GB/T 38636-2020*）：
![](https://raw.githubusercontent.com/BaihlUp/Figurebed/master/2023/202312061009296.png)

从图中可以了解到GMTLS1.1：

- `密钥交换算法`包含ECDHE、ECC、IBSDH、IBC 以及 RSA，其中 IBC 和 IBSDH 主要涉及到 SM9 标识密码算法_。
- 加密算法只有`SM4`，加密模式涉及两种方式：`CBC`和`GCM`。
- 校验算法/哈希算法包含：SM3 和 SHA256

注：从上图中我们还可以了解到，在GMTLS1.1支持的套件中，包含了国际算法RSA和SHA256，这里从公开文献中未找到具体原因，大胆猜测主要是为了：

- 支持国际知名CA机构颁发的数字证书
- 作为一种切换到全国密TLS过度

## 1.3 协议原理

GMTLS1.1 协议主要参考了`TLS1.1`，并借鉴了`TLS1.2`的部分内容。在关键流程上基本一致，只是在密码算法使用上用`国密算法`对`国际算法`进行了替换。

![](https://raw.githubusercontent.com/BaihlUp/Figurebed/master/2023/202312061013736.png)



# 2 基于铜锁（Tongsuo）部署国密服务端

铜锁（Tongsuo）对国密双证书协议进行了支持，并统称为 NTLS。NTLS 并不是指某一种具体的符合商用密码相关技术标准要求的网络协议，而是多个协议的统称。在 铜锁（Tongsuo） 中代指 TLCP 和 0024 国密双证书协议，因为 NTLS 和标准 TLS 协议存在工作方式的不同，因此 铜锁（Tongsuo） 中增加了一些新的 API 来对其进行支持。而应用程序若想使用 NTLS 功能，就需要调用这些新增 API，给现有基于 OpenSSL API 进行适配的应用程序带来了额外的开发工作量。

1. 下载 Tongsuo

NTLS (TLCP and GM/T 0024) 基于 Tongsuo。

```bash
git clone https://github.com/Tongsuo-Project/Tongsuo.git
```


2. 下载 Tengine

```bash
git clone https://github.com/alibaba/tengine.git
```


3. 编译 Tengine

```shell
./configure --add-module=modules/ngx_tongsuo_ntls \
    --with-openssl=../Tongsuo \
    --with-openssl-opt="--strict-warnings --api=1.1.1 enable-ntls" \
    --with-http_ssl_module --with-stream \
    --with-stream_ssl_module --with-stream_sni
```

4. 配置 Tengine 开启 NTLS 

可以直接使用Tongsuo项目中已经签发的SM2双证书，仅用于测试；

根CA > 中间CA > 客户端/服务端证书，根CA证书签发中间CA证书，中间CA证书签发客户端和服务端的证书，包括签名证书和加密证书，这些证书的公钥算法都是SM2。

服务端签名证书和私钥：
[https://github.com/Tongsuo-Project/Tongsuo/blob/master/test/certs/sm2/server_sign.crt](https://github.com/Tongsuo-Project/Tongsuo/blob/master/test/certs/sm2/server_sign.crt)
[https://github.com/Tongsuo-Project/Tongsuo/blob/master/test/certs/sm2/server_sign.key](https://github.com/Tongsuo-Project/Tongsuo/blob/master/test/certs/sm2/server_sign.key)

服务端加密证书和私钥：
[https://github.com/Tongsuo-Project/Tongsuo/blob/master/test/certs/sm2/server_enc.crt](https://github.com/Tongsuo-Project/Tongsuo/blob/master/test/certs/sm2/server_enc.crt)
[https://github.com/Tongsuo-Project/Tongsuo/blob/master/test/certs/sm2/server_enc.key](https://github.com/Tongsuo-Project/Tongsuo/blob/master/test/certs/sm2/server_enc.key)

CA证书，这里面包含根CA和中间CA证书：
[https://github.com/Tongsuo-Project/Tongsuo/blob/master/test/certs/sm2/chain-ca.crt](https://github.com/Tongsuo-Project/Tongsuo/blob/master/test/certs/sm2/chain-ca.crt)
客户端签名证书和私钥：
[https://github.com/Tongsuo-Project/Tongsuo/blob/master/test/certs/sm2/client_sign.crt](https://github.com/Tongsuo-Project/Tongsuo/blob/master/test/certs/sm2/client_sign.crt)
[https://github.com/Tongsuo-Project/Tongsuo/blob/master/test/certs/sm2/client_sign.key](https://github.com/Tongsuo-Project/Tongsuo/blob/master/test/certs/sm2/client_sign.key)

客户端加密证书和私钥：
[https://github.com/Tongsuo-Project/Tongsuo/blob/master/test/certs/sm2/client_enc.crt](https://github.com/Tongsuo-Project/Tongsuo/blob/master/test/certs/sm2/client_enc.crt)
[https://github.com/Tongsuo-Project/Tongsuo/blob/master/test/certs/sm2/client_enc.key](https://github.com/Tongsuo-Project/Tongsuo/blob/master/test/certs/sm2/client_enc.key)

```bash
worker_processes  1;

events {    
	worker_connections  1024;
}

http {
	include       mime.types;
	default_type  application/octet-stream;
	server {
			listen       443 ssl;
			server_name  localhost;
			enable_ntls  on;
			ssl_sign_certificate        server_sign.crt;
			ssl_sign_certificate_key    server_sign.key;         
			ssl_enc_certificate         server_enc.crt;
			ssl_enc_certificate_key     server_enc.key;
			location / {
				return 200 "body $ssl_protocol:$ssl_cipher";
			}
	}
}

stream {
     server {
        listen       8443 ssl;
        enable_ntls  on;
        ssl_sign_certificate        server_sign.crt;
        ssl_sign_certificate_key    server_sign.key;            
		ssl_enc_certificate         server_enc.crt;        
		ssl_enc_certificate_key     server_enc.key;
        return "body $ssl_protocol:$ssl_cipher";
    }
}
```


# 3 国密证书生成



# 4 客户端工具

wrk+铜锁：[https://www.yuque.com/tsdoc/ts/kd16l0](https://www.yuque.com/tsdoc/ts/kd16l0)
curl+铜锁：[https://www.yuque.com/tsdoc/ts/xuxk18ckbtpgvfdi](https://www.yuque.com/tsdoc/ts/xuxk18ckbtpgvfdi)



# 5 测试

### 5.1 国密双证书



### 5.2 双向认证






参考资料：# [TLS原理与实践（四）国密TLS](https://www.cnblogs.com/informatics/p/17594441.html)