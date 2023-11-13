
# 0 参考资料
## 0.1 开源库

**uiautomator2**：https://github.com/openatx/uiautomator2

## Android调试桥（adb）

adb clilent
adb server：ADB Server是运行在PC上的一个后台进程
adb devices：展示adb中连接的手机
adb shell pm list packages：展示手机中安装的包
adb install/uninstall ：使用adb在手机中安装apk

> 如果使用adb失败，需要在手机中打开开发者模式


### ubuntu安装Adb工具

`adb tcpip 5555` 设置tcp连接
`adb connect  192.168.1.8` 连接手机


## Uiautomator2连接手机








# 抓包工具
## Fiddler

是一个Web调试代理平台，可以监控和修改web数据流

安装证书后，配置代理

浏览器中安装switchyOmega 代理软件，基于Fiddler代理访问

- 设置断点

工具栏中 “规则” 菜单栏下。

使用命令行：

```
bpu
# 设置响应断点
bpafter https://www.baidu.com
# 取消断点
bpa 
```

在此设置使用本地资源替换掉响应：
![](https://raw.githubusercontent.com/BaihlUp/Figurebed/master/2023/202311131747379.png)

## mitmproxy

安装完后，有三个工具：mitmproxy、mitmdump（抓取数据写入文件）、mitmweb
通过代理截获数据，拦截请求，修改请求，拦截返回，修改返回，可以载入自定义python脚本

通过pip 可以直接安装

mitmproxy的启动：
mitm.it进行证书下载

## packet capture

运行在手机上的app，可以捕获http/https网络流量


## Appium移动端自动化测试工具
appium类库封装了标准Selenium客户端类库。













































