
# 0 参考资料
## 0.1 开源库

**uiautomator2**：https://github.com/openatx/uiautomator2

## 0.2 工具
模拟器：夜神模拟器
抓包工具：

## Android调试桥（adb）
使用adb工具可以连接手机，通过PC操作手机，进行调试。

adb clilent
adb server：ADB Server是运行在PC上的一个后台进程
adb devices：展示adb中连接的手机
adb shell pm list packages：展示手机中安装的包
adb install/uninstall ：使用adb在手机中安装apk
adb -s [设备号] shell ：进入手机底层系统
adb push [文件路径] [手机存储位置] ：从PC传文件到手机
adb pull [手机存储位置] [文件路径] ：从手机传文件到PC
adb screencap /sdcard/test.png ：手机截图

 
> 如果使用adb失败，需要在手机中打开开发者模式


- ubuntu安装Adb工具

`adb tcpip 5555` 设置tcp连接
`adb connect  192.168.1.8` 连接手机


## Uiautomator2连接手机
包括uiautomatorviewer、uiautomator
用来做UI测试，点击控件看是否符合预期。

可以定位手机界面的元素

替换使用：高级版uiautomatorviewer-master

可以使用uiautomatorviewer定位手机的元素


## Appium移动端自动化工具
appium类库封装了标准Selenium客户端类库。是一个开源测试自动化框架
是个C/S架构

bootstrap.jar

- inspector菜单
	Automatic Server
	desired capability 
	
获取appPackage、appActivity
执行：`aapt.exe dump badging [APP安装包]`
使用adb shell获取




# 抓包工具
## Fiddler安装和使用

是一个Web调试代理平台，可以监控和修改web数据流

安装证书后，配置代理
![](https://raw.githubusercontent.com/BaihlUp/Figurebed/master/2023/202311131750472.png)
![](https://raw.githubusercontent.com/BaihlUp/Figurebed/master/2023/202311131754614.png)
以上设置完Fiddler证书和代理配置后，浏览器中安装switchyOmega 代理软件，基于Fiddler代理访问，如果是手机模拟器中，需要先开启桥接模式，与本机PC处在同一网络，然后在手机中设置代理，指定为Fiddler。

访问Fiddler:PORT（PC的IP和代理端口），会显示安装Fiddler证书的界面，正常安装证书，就可以访问HTTPS网站。

### Fiddler 工具栏
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

## mitmproxy安装和使用

安装完后，有三个工具：mitmproxy、mitmdump（抓取数据写入文件）、mitmweb
通过代理截获数据，拦截请求，修改请求，拦截返回，修改返回，**可以载入自定义python脚本**
通过pip 可以直接安装

mitmproxy的启动：执行mitmproxy
mitm.it进行证书下载

### mitmproxy使用
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


### mitmdump使用

`mitmdump -p 8889 -s test.py`


## packet capture

运行在手机上的app，可以捕获http/https网络流量





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




# 实战项目
## 抓取抖音数据

































