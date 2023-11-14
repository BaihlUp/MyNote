
# 0 参考资料
## 0.1 开源库

**uiautomator2**：https://github.com/openatx/uiautomator2


# 1 环境部署

## 1.1 工具
**需要依赖以下工具：**
1. 模拟器：夜神模拟器：开启桥接模式，让手机和PC在同一个网段
2. 抓包工具：Fiddler、mitmproxy、packet capture（手机端）
3. adb工具
4. Uiautomator2、Appium：用于执行自动化移动应用程序测试的工具

## 1.2 Android调试桥（adb）
使用adb工具可以连接手机，通过PC操作手机，进行调试。adb是一个C/S架构，如下：
adb clilent：命令行程序“adb” 用于从shell或脚本中运行adb命令
adb server：ADB Server是运行在PC上的一个后台进程

- **adb相关命令**

adb devices：展示adb中连接的手机，默认展示本地环境中的手机
adb shell pm list packages：展示手机中安装的包
adb install/uninstall ：使用adb在手机中安装apk
adb -s [设备号] shell ：进入手机底层系统
adb push [文件路径] [手机存储位置] ：从PC传文件到手机
adb pull [手机存储位置] [文件路径] ：从手机传文件到PC
adb screencap /sdcard/test.png ：手机截图

- **adb命令执行后，看不到手机，可能原因如下**：

1. 需要在手机中打开开发者模式，设置->关于手机->版本号（多次点击）
2. 检查adb版本，安卓版本在4.x上的版本都要求adb版本必须是1.0.31版本及以上
3. 若连接电脑本地的模拟器也是无法连接的话，可以查看下模拟器的adb.exe和电脑环境配置的adb.exe文件的版本是不是一致的(查看adb版本：adb version)
4. 修改手机型号，然后重启，多试几次


- **ubuntu安装Adb工具**

`adb tcpip 5555` 设置tcp连接
`adb connect 192.168.1.8` 连接手机，通过IP连接手机


## 1.3 Uiautomator2和Appium
### 1.3.2 介绍
- **UIAutomator2**

1. **专注于Android平台**：UIAutomator2是由Google提供的Android平台上的UI自动化测试框架，专门用于测试Android应用程序。
2. **直接操作底层**：UIAutomator2能够直接与Android设备交互，可以访问应用程序的任何元素，包括应用外部的系统应用和通知栏。
3. **基于Java**：UIAutomator2主要使用Java编写，对于使用Java语言的开发者来说，学习和使用起来比较自然。
4. **仅限于Android 7.0以上**：UIAutomator2支持Android 7.0及更高版本。之前版本的UIAutomator框架有一些限制。

- **Appium**

1. **跨平台兼容**：Appium是一种跨平台的自动化测试工具，可以测试多种移动平台，包括Android和iOS。
2. **支持多种语言**：Appium支持多种编程语言，如Java、Python、JavaScript等，这使得开发者可以根据自己的偏好选择合适的语言进行测试脚本编写。
3. **使用WebDriver协议**：Appium使用WebDriver协议来执行测试，这意味着熟悉WebDriver的开发人员可以很容易地迁移到Appium。
4. **更广泛的设备支持**：由于其跨平台性质，Appium可以测试多种设备和模拟器，而不仅仅是Android设备。

### 1.3.3 Uiautomator2使用
uiautomator2是一个自动化测试开源工具，仅支持Android平台的原生应用测试。它本来是Google提供的一个自动化测试的Java库，后来发展了python-uiautomator2，封装了谷歌自带的uiautomator测试框架，提供便利的python接口，用它可以很便捷的编写python脚本来实现app的自动化测试
原理解析：
- python端：运行脚本，往移动端发送HTTP请求
- 移动端：安装atx-agent，然后atx-agent启动uiautomator2服务进行监听，并识别python脚本，转换为uiautomator2的代码。

> 移动设备通过WIFI(同一网段)或USB接收到PC上发来的HTTP请求，执行制定的操作

1. 下载Android SDK，里边有adb工具
2. 安装uiautomator2
3. 使用 `adb devices` 查看本地手机设备
4. 在手机上安装 atx-agent，执行 `python -m uiautomator2 init`

包括uiautomatorviewer、uiautomator
用来做UI测试，点击控件看是否符合预期。

可以定位手机界面的元素

替换使用：高级版uiautomatorviewer-master

可以使用uiautomatorviewer定位手机的元素


### 1.3.4 Appium使用
appium类库封装了标准Selenium客户端类库。是一个开源测试自动化框架
是个C/S架构

bootstrap.jar

- inspector菜单
	Automatic Server
	desired capability 
	
获取appPackage、appActivity
执行：`aapt.exe dump badging [APP安装包]`
使用adb shell获取


## 1.4 抓包工具
### 1.4.1 Fiddler安装和使用

是一个Web调试代理平台，可以监控和修改web数据流

安装证书后，配置代理
![](https://raw.githubusercontent.com/BaihlUp/Figurebed/master/2023/202311131750472.png)
![](https://raw.githubusercontent.com/BaihlUp/Figurebed/master/2023/202311131754614.png)
以上设置完Fiddler证书和代理配置后，浏览器中安装switchyOmega 代理软件，基于Fiddler代理访问，如果是手机模拟器中，需要先开启桥接模式，与本机PC处在同一网络，然后在手机中设置代理，指定为Fiddler。

访问Fiddler:PORT（PC的IP和代理端口），会显示安装Fiddler证书的界面，正常安装证书，就可以访问HTTPS网站。

- Fiddler 工具栏

打开以下配置：
![](https://raw.githubusercontent.com/BaihlUp/Figurebed/master/2023/202311131757049.png)

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

### 1.4.2 mitmproxy安装和使用

安装完后，有三个工具：mitmproxy、mitmdump（抓取数据写入文件）、mitmweb
通过代理截获数据，拦截请求，修改请求，拦截返回，修改返回，**可以载入自定义python脚本**
通过pip 可以直接安装

mitmproxy的启动：执行mitmproxy
mitm.it进行证书下载

- **mitmproxy使用**

1. **过滤流量**:
    
    - 使用 `~b` 命令可以进行基于正则表达式的流量过滤。例如，`~b example.com` 可以仅显示包含 "example.com" 的流量。
    - `~q` 命令用于对查询进行过滤，例如 `~q POST` 可以只显示包含 POST 请求的流量。
    - 使用 `~m` 可以根据内容类型进行过滤，例如 `~m image` 可以只显示图片内容的流量。
2. **设置断点**:
    
    - 使用 `b [pattern]` 命令可以设置断点，例如 `b example.com` 可以使 mitmproxy 在匹配 "example.com" 的请求时暂停。
    - `be` 可以设置对响应的断点。
    - `bd` 可以删除断点。
    - `bl` 可以列出当前所有的断点。
3. **查找**:
    
    - 使用 `/[pattern]` 进行内容查找，例如 `/login` 可以在流量中查找包含 "login" 的内容。
    - 可以使用 `n` 和 `N` 来分别查看下一个和上一个匹配项。

1. **修改流量**:
    
    - 使用 `e` 命令可以编辑请求和响应，允许你修改它们。
    - `tab` 键可以在请求和响应之间切换，方便修改两者。
    - 使用 `A` 来接受修改并应用到流量中。
2. **保存和导出**:
    
    - 使用 `w` 命令可以将流量保存到文件中，例如 `w filename.txt`。
    - `r` 命令可以重新加载保存的文件，以便重新查看和修改流量。
    - 可以使用 `--set filter` 选项将过滤器应用到加载的文件上。
3. **TLS设置**:
    
    - 使用 `set tls_passthrough [pattern]` 可以绕过 TLS，将流量直通而不进行 TLS 解密。这对于特定域的安全连接非常有用。
4. **查看详细信息**:
    
    - 按下 `i` 键可以查看关于请求/响应的详细信息，包括头部和元数据。
    - 使用 `v` 可以查看流量的可视化预览，尤其是图像和其它媒体类型。
5. **快捷键和交互**:
    
    - `:` 键可以进入命令行模式，在其中输入 mitmproxy 命令。
    - 使用 `q` 键可以退出 mitmproxy。


`mitmdump -p 8889 -s test.py`


### 1.4.3 packet capture

运行在手机上的app，可以捕获http/https网络流量


# 2 实战项目
## 抓取抖音数据

## App应用数据抓取
使用Fiddler抓包，然后进行分析
编写Python脚本，爬取数据
测试APP为 豆果美食

代码地址：

## 移动端自动化控制工具
1. 安装JDK环境
2. 安装Android 开发工具：

Android SDK Tools


Extras

[adb工具详讲1]
adb start-server
> 如果出现错误，PC上的跟手机上的adb工具版本不一致，可以使用PC中的adb覆盖模拟器的adb






































