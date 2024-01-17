---
title: Ubuntu系统常用命令
date: 2024-01-17
categories:
  - 工具使用
tags:
  - Ubuntu
published: false
---
# 1 软件安装
## 1.1 APT 命令
1. 安装软件包

```bash
apt-get install PackageName // 普通安装  
apt-get install PackageName=VersionName // 安装指定包的指定版本  
apt-get --reinstall install PackageName // 重新安装

apt-get build-dep PackageName // 安装源码包所需要的编译环境  
apt-get -f install // 修复依赖关系

apt-get source PackageName // 下载软件包的源码  
```
2. 卸载软件包

```bash
apt-get remove PackageName // 删除软件包, 保留配置文件

apt-get --purge remove PackageName // 删除软件包, 同时删除配置文件  
apt-get purge PackageName // 删除软件包, 同时删除配置文件

apt-get autoremove PackageName // 删除软件包, 同时删除为满足依赖  
// 而自动安装且不再使用的软件包

apt-get --purge autoremove PackageName // 删除软件包, 删除配置文件,  
// 删除不再使用的依赖包

apt-get clean && apt-get autoclean // 清除 已下载的软件包 和 旧软件包  
```

3. 更新软件包

```bash
apt-get update // 更新安装源(Source)  
apt-get upgrade // 更新已安装的软件包  
apt-get dist-upgrade // 更新已安装的软件包(识别并处理依赖关系的改变)  
```

4. 查询软件包

```bash
dpkg -l // 列出已安装的所有软件包

apt-cache search PackageName // 搜索软件包  
apt-cache show PackageName // 获取软件包的相关信息, 如说明、大小、版本等

apt-cache depends PackageName // 查看该软件包需要哪些依赖包  
apt-cache rdepends PackageName // 查看该软件包被哪些包依赖

apt-get check // 检查是否有损坏的依赖
```
5. 软件查看

```bash
 [dpkg](https://so.csdn.net/so/search?q=dpkg&spm=1001.2101.3001.7020) -S softwarename 显示包含此软件包的所有位置，
 dpkg -L softwarename 显示安装路径。
 dpkg -l softwarename 查看软件版本
```
