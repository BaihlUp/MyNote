
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

[adb使用-详细教程（Awesome Adb）](https://blog.csdn.net/u010610691/article/details/77663770)

- **adb相关命令**

adb kill-server ：关闭adb server
adb start-server：启动adb server
adb devices：展示adb中连接的手机，默认展示本地环境中的手机
adb shell pm list packages：展示手机中安装的包
adb install/uninstall ：使用adb在手机中安装apk
adb -s [设备号] shell ：进入手机底层系统
adb push [文件路径] [手机存储位置] ：从PC传文件到手机
adb pull [手机存储位置] [文件路径] ：从手机传文件到PC
adb screencap /sdcard/test.png ：手机截图

- **adb命令执行后，看不到手机，可能原因如下**：

1. 需要在手机中打开USB调试和开发者模式，设置->关于手机->版本号（多次点击）
2. 检查adb版本，安卓版本在4.x上的版本都要求adb版本必须是1.0.31版本及以上
3. 若连接电脑本地的模拟器也是无法连接的话，可以查看下模拟器的adb.exe和电脑环境配置的adb.exe文件的版本是不是一致的(查看adb版本：adb version)
4. 修改手机型号，然后重启，多试几次


- **ubuntu安装Adb工具**

在PC上开启tcpip模式：`adb tcpip 5555` 设置tcp连接模式和端口
在Ubuntu上进行连接：`adb connect 192.168.1.8:5555` 通过IP连接手机


## 1.3 Uiautomator2使用
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

### 1.3.3 Uiautomator2安装
uiautomator2是一个自动化测试开源工具，仅支持Android平台的原生应用测试。它本来是Google提供的一个自动化测试的Java库，后来发展了python-uiautomator2，封装了谷歌自带的uiautomator测试框架，提供便利的python接口，用它可以很便捷的编写python脚本来实现app的自动化测试
原理解析：
- python端：运行脚本，往移动端发送HTTP请求
- 移动端：安装atx-agent，然后atx-agent启动uiautomator2服务进行监听，并识别python脚本，转换为uiautomator2的代码。

> 移动设备通过WIFI(同一网段)或USB接收到PC上发来的HTTP请求，执行制定的操作

1. 下载Android SDK，里边有adb工具
2. 安装uiautomator2：`pip install -i https://pypi.tuna.tsinghua.edu.cn/simple uiautomator2`
3. 使用 `adb devices` 查看本地手机设备
4. 对手机进行初始化，就是在手机上安装 atx-agent，执行 `python -m uiautomator2 init`，手机上会提示安装操作，最后会有一个ATX的app在手机中，打开ATX，开启UIAUTOMATOR和ATXAGENT
5. ATX app加入开机启动，并设置可以后台运行

![](https://raw.githubusercontent.com/BaihlUp/Figurebed/master/2023/202311151048977.png)

> 以上`python -m uiautomator2 init`操作会安装u2包，安装了如下程序：
> uiautomator-server：就是谷歌原生的uiautomator
> atx-agent：uiautomator守护进程
> atx-agent增加远程控制的功能，依赖minicap和minitouch这两个工具
> 以上工具安装后，放在PC的家目录下的 `.uiautomator`目录下



包括uiautomatorviewer、uiautomator
用来做UI测试，点击控件看是否符合预期。

可以定位手机界面的元素

替换使用：高级版uiautomatorviewer-master

可以使用uiautomatorviewer定位手机的元素

### 1.3.4 Uiautomator2操作手机
- Uiautomator2服务

```python
import uiautomator2 as u2  
import time

d = u2.connect_usb("127.0.0.1:62025")
# 通过start方法启动uiautomator服务  
d.service("uiautomator").start()  
# time.sleep(2)  
print(d.service("uiautomator").running())
# 停止服务  
d.service("uiautomator").stop()

# 查看atx-agent运行状态,如果atx-agent真的停止了，我们可以通过connect来唤醒atx-agent服务  
print(d.agent_alive)
```

- 通过python脚本，对手机进行操控：

```python
import uiautomator2 as u2  
import time  
  
#当前的网络环境需要在一个局域网里面  
#通过手机WIFI来进行连接,需要查看手机的IP地址  
d = u2.connect_wifi("192.168.1.7")  
# print(d.info)  
# 通过手机的序列号  
d = u2.connect_usb("4bf05af7")  
# print(d.info)  
#通过adb wifi也就是adb tcpip模式,注意不要丢掉端口号  
d = u2.connect_adb_wifi("192.168.1.7:5555")  
#device_info可以获取详细的设备信息  
print(d.device_info)  
# 查看获取到的wifi地址  
print(d.wlan_ip)
# 查看设备的分辨率  
print(d.window_size())

#启动手机上的app,通过aapt工具（在夜神模拟器中包含）来获取包名  
#appt获取包名的时候，在aapt dump badging xxx.apk,命令输出的第一行，有个package，获取到的是包名  
d.app_start("com.ss.android.ugc.aweme")  
#运行七秒  
time.sleep(7)  
#停止抖音app  
d.app_stop("com.ss.android.ugc.aweme")
```
启动手机APP，需要安装 aapt工具，通过aapt工具执行命令 `aapt dump badging xxx.apk` 获取包名。

- 操作APP，安装APP，清除缓存

```python
import uiautomator2 as u2  
  
# 通过USB进行连接  
d = u2.connect_usb("127.0.0.1:62025")  
# 通过app_install方法安装apk，path="xxx.apk"  
d.app_install("./imooc7.3.410102001android.apk")  
# 启动app  
d.app_start(package_name="cn.com.open.mooc")  
# 获取当前前台运行的app的信息  
print(d.app_current())  
d.app_stop("cn.com.open.mooc")  
# 获取app详细信息  
print(d.app_info("cn.com.open.mooc"))  
# 清除app缓存  
# 尤其是我们后面要进行的视频数据抓取，会产生一定的缓存  
d.app_clear("cn.com.open.mooc")  
# 卸载app  
d.app_uninstall("cn.com.open.mooc")  
# 获取所有app列表  
print(d.app_list())  
# 获取所有正在运行的app的列表  
print(d.app_list_running())  
# 停止所有app  
d.app_stop_all()  
# 卸载所有app,卸载所有第三方app,u2项目包不会卸载'pm', 'list', 'packages', '-3  
# d.app_uninstall_all()
```

## 1.4 Weditor使用
### 1.4.1 什么是weditor
定位app控件的一种工具，相当于selenium，可以快速定位app以及清晰的看到他们之间的层级关系，抓取app数据之前可以通过它先了解app的结构以及一些信息(就相当于饭前洗手虽然没有必要关联，但是有助于你的健康)它虽然对抓取app数据没太大相关，但可以帮助了解app的组成以及实现它的逻辑

### 1.4.2 怎么使用weditor
1. 安装adb
2. 安装uiautomator2以及weditor

```shell
pip install --upgrade --pre uiautomator2
pip install weditor
```

在Windows环境下安装Weditor报错，需要设置下边环境变量：
![](https://raw.githubusercontent.com/BaihlUp/Figurebed/master/2023/202311151420469.png)

3. 初始化移动端设备安装插件python -m uiautomator2 init
4. 连接手机(打开模拟器)DOS中输入adb devices
5. DOS中输入weditor连接浏览器中就会弹出元素定位工具

### 1.4.3 weditor页面介绍
一共有四个区域：移动设备选取区域、控件属性区域、代码展示区域、层级关系和结果展示区域。有了这个定位工具就方便我们测试app以及定位等操作(更加轻便，模拟多种状态，快速定位)

![](https://raw.githubusercontent.com/BaihlUp/Figurebed/master/2023/202311151236590.png)

### 1.4.4 定位控件
用u2连接手机，普通只能通过查找包名来定位现在可以通过weditor来定位更加方便和快捷
![](https://raw.githubusercontent.com/BaihlUp/Figurebed/master/2023/202311151237045.png)
使用adb devices获取手机设备，填入Connect，进行连接。

- text：全文本匹配
- textContains：文本包含匹配(注意要唯一才能够操作)
- textMatches：正则表达式匹配
- textStartsWith：起始文本匹配
- ClassName：如果有多个要根据索引值限定(注意层级关系)
- classNameMatches：正则表达式匹配
- resourceId：资源Id匹配如果有多个要根据索引值限定(注意层级关系)
- resourceIdMatches：正则表达式(可能定位多个，用混合使用避免定位不到)

坐标点定位：
![](https://raw.githubusercontent.com/BaihlUp/Figurebed/master/2023/202311151238052.png)

ClassName有多个层级：
![](https://raw.githubusercontent.com/BaihlUp/Figurebed/master/2023/202311151238062.png)


```python
import uiautomator as u2
import time

d = u2.connect_usb('127.0.0.1:62001')
d.app_start('com.baidu.searchbox') # 通过weditor更加快速定位，想开启谁去找就好了
time.sleep(2) # 防止页面跳转之后控件没有加载出来而造成找不到的情况
d(text="登录并进入百度APP").click(timeout=4) # 在weditor中找到text定位超过4秒就抛出异常
d(text="登录并进入百度APP").click_exists(timeout=4) # 判断text存不存在，不存在就直接跳过
print(d(text='登录并进入百度APP').exists) # 判断是否有此控件
d(textContains='并进').click()
d(textMatches='.{2}APP').click()
d(textStartsWith='登录并').click()
d(className='android.widget.TextView')[2].click()
d(className='android.widget.TextView', instance=2).click() # 跟上一种方法一样
d(classNameMatches='.\w{2}APP', text='登录并进入百度APP').click()
d(resourceId='com.baidu.searchbox:id/login')[0].click()
d(resourceId='com.baidu.searchbox:id/login', instance=0).click() # 跟上一种方法一样
d(resourceIdMatches='.*?\/login', text='登录并进入百度APP').click()
d(resourceId='com.baidu.searchbox:id/login', text='	登录并进入百度APP').click() # 混合使用
# 混合定位之链式定位,不推荐使用
d(className='android.widget.TextView').child(text='登录并进入百度APP').click()
d(className='android.widget.TextView').child_by_text('登录并进入百度APP').click() # 意义同上定位控件
d.click(533, 687) # 将坐标点写入
```

### 1.4.5 解锁图案验证
常用操作：

```python
import uiautomator2 as u2  
  
d = u2.connect_usb("127.0.0.1:62025")  
d.service("uiautomator").start()  
print(d.service("uiautomator").running())  
# 滑动解锁操作  
# 息屏  
d.screen_off()  
# 点亮屏幕  
d.screen_on()  
# 解锁  
d.unlock()  
# 获取屏幕状态  
print(d.info.get("screenOn"))  
d.unlock()  
# home键  
d.press("home")  
# 返回键  
d.press("back")  
  
d.swipe_ext("left")  
d.swipe_ext("right")  
# 滑动解锁  
# swipe_points  
  
# 先解锁调出九宫格界面  
# 0.224, 0.393  
# (0.493, 0.395)  
# (0.781, 0.396)  
# (0.501, 0.551)  
# (0.218, 0.705)  
# (0.501, 0.703)  
# (0.773, 0.703)  
# duration0.2 是0.2秒  
d.unlock()  
d.swipe_points(points=[  
		(0.224, 0.393),  
		(0.493, 0.395),  
		(0.781, 0.396),  
		(0.501, 0.551),  
		(0.218, 0.705),  
		(0.501, 0.703),  
		(0.773, 0.703)  
		], duration=0.2)  
d.screen_off()
```

通过坐标定位模拟人的行为给手机解锁(图案锁)

```python
import uiautomator2 as u2

d = u2.connect_usb('127.0.0.1:62001')
# 将坐标点以列表方式传入，0.2毫秒延迟
d.swipe_points(points=[(10,20),(30,40),(40,50)],duration=0.2)
```

- 获取控件源代码（html）

```python
import uiautomator2 as u2  
  
d = u2.connect_usb("127.0.0.1:62025")  
with open("phone.file", 'w', encoding='utf-8') as f:  
	# 通过这个方法来获取到控件的源代码文件  
	f.write(d.dump_hierarchy())  
d.xpath('//*[@text="蓝牙"]').click()
```
通过XPath定位元素。

### 1.4.6 模拟登录并操作考研帮
熟悉通过python操作app各种控件，模拟人的行为，为以后爬取app数据做铺垫
```python
import uiautomator2 as u2


class HandleKaoyanbang(object):
    def __init__(self, apk, serial="127.0.0.1:62025"):
        # 当前是通过usb的方法来连接移动设备的
        self.d = u2.connect_usb(serial=serial)
        app_list = self.d.app_list()
        # 如果APP已安装，则跳过
        if "com.tal.kaoyan" not in app_list:
            self.d.app_install(apk)
        self.size = self.get_windowsize()
        self.handle_watcher()

    def handle_watcher(self):
        """定义一个监控器"""
        # 监控器会单独的起一个线程
        # 允许权限
        self.d.watcher.when('//*[@resource-id="com.android.packageinstaller:id/permission_allow_button"]').click()
        # 用户隐私协议
        self.d.watcher.when('//*[@resource-id="com.tal.kaoyan:id/tip_commit"]').click()
        # 广告
        self.d.watcher.when('//*[@resource-id="com.tal.kaoyan:id/tv_skip"]').click()
        # 监控器写好之后，要通过start方法来启动
        self.d.watcher.start()

    def get_windowsize(self):
        """获取手机屏幕的大小"""
        return self.d.window_size()

    def close_app(self):
        # 监控器关闭
        self.d.watcher.stop()
        # 停止考研帮app
        self.d.app_stop("com.tal.kaoyan")
        # 清理缓存
        self.d.app_clear("com.tal.kaoyan")

    def handle_kaoyanbang_app(self):
        """启动考研帮app,并实现自动化操作"""
        # aapt这个工具
        # 通过weditor
        self.d.app_start(package_name="com.tal.kaoyan")
        # 在点击之前需要判断是否有这个控件
        self.d(text="密码登录").click_exists(timeout=10)
        # 通过找到相关控件之后,文本控件，set_text这个方法来输入文字
        self.d(resourceId="com.tal.kaoyan:id/login_email_edittext").set_text("450120127@163.com")
        # 输入密码
        self.d(resourceId="com.tal.kaoyan:id/login_password_edittext").set_text("abcd1234")
        # self.d(resourceId="com.tal.kaoyan:id/login_login_btn").click()
        self.d(text="登录").click()
        # 在10秒钟如果这个界面启动了
        if self.d.wait_activity("com.tal.kaoyan.ui.activity.HomeTabActivity", timeout=10):
            self.d(text="研讯").click_exists(timeout=10)
            # 获取到屏幕的中心点，x轴
            # 在获取到y轴远方点，获取到y轴近点
            x1 = int(self.size[0] * 0.5)
            y1 = int(self.size[1] * 0.9)
            y2 = int(self.size[1] * 0.15)
            while True:
                # get toast，是安卓系统系统的一个信息提示操作
                if self.d.toast.get_message(0) == "内容已经全部加载完了":
                    self.close_app()
                    return
                # 开始滑动研讯
                self.d.swipe(x1, y1, x1, y2)

if __name__ == '__main__':
    k = HandleKaoyanbang("./kaoyanbang_3.5.6.272.apk")
    k.handle_kaoyanbang_app()
```


## 1.7 Appium使用
appium类库封装了标准Selenium客户端类库。是一个开源测试自动化框架
是个C/S架构

bootstrap.jar

- inspector菜单
	Automatic Server
	desired capability 
	
获取appPackage、appActivity
执行：`aapt.exe dump badging [APP安装包]`
使用adb shell获取


使用docker部署 Appium
在PC上通过命令 `adb -s [设备ID] tcpip 5555` 修改为网络连接模式


## 1.6 抓包工具
### 1.6.1 Fiddler安装和使用

是一个Web调试代理平台，可以监控和修改web数据流

安装证书后，配置代理
![](https://raw.githubusercontent.com/BaihlUp/Figurebed/master/2023/202311131750472.png)
![](https://raw.githubusercontent.com/BaihlUp/Figurebed/master/2023/202311131754614.png)
以上设置完Fiddler证书和代理配置后，重启Fiddler，浏览器中安装switchyOmega 代理软件，基于Fiddler代理访问，如果是手机模拟器中，需要先开启桥接模式，与本机PC处在同一网络，然后在手机中设置代理，指定为Fiddler。

手机浏览器中访问Fiddler:PORT（PC的IP和代理端口），会显示安装 Fiddler 证书的界面，正常安装证书，就可以访问HTTPS网站。
![](https://raw.githubusercontent.com/BaihlUp/Figurebed/master/2023/202311151753051.png)

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

### 1.6.2 mitmproxy安装和使用

安装完后，有三个工具：mitmproxy、mitmdump（抓取数据写入文件）、mitmweb
通过代理截获数据，拦截请求，修改请求，拦截返回，修改返回，**可以载入自定义python脚本**
通过pip 可以直接安装

mitmproxy的启动：执行mitmproxy
浏览器中输入：mitm.it ，进行证书下载

mitmproxy是模拟中间人，进行通信，如下：
![](https://raw.githubusercontent.com/BaihlUp/Figurebed/master/2023/202311151621503.png)

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


### 1.6.3 packet capture（移动端）

运行在手机上的app，可以捕获http/https网络流量


### 1.6.4 App无法抓包探秘
**可能有如下原因：**
- App限定了系统代理接口

App通过获取当前代理配置，判断是通过了代理访问，则让无法访问网站。

有如下三种解决方法：
1. 通过安装 xposed 框架，编写代码解决。
2. 使用全局代理 proxydroid 软件，需要root
3. 使用packet capture，无需root


- App启用了ssl-pinning技术防止中间人攻击

App中内置了证书，验证服务端的证书，导致无法进行中间人代理。

解决方法：
1. 通过安装 xposed 框架，安装 JustTrustMe 模块，屏蔽客户端验证证书


- App采用了双向证书绑定技术

暂无解决方法，需要破解APP，进行逆向

## 1.7 atxserver2 多设备管理
### 1.7.1 介绍
atxserver2 是一个移动设备管理平台，主要是 Python3+NodeJS+RethinkDB开发，项目地址：[https://github.com/openatx/atxserver2](https://github.com/openatx/atxserver2)

通过多设备管理工具，可以同时在PC上实时控制多个手机，并且获取对应手机的信息。

![](https://raw.githubusercontent.com/BaihlUp/Figurebed/master/2023/202311161124217.png)
1. RethinkDB：这是一个轻量级的数据库，用来存储数据
2. atxserver2：主要负责处理数据，显示与用户的前端交互等等，单独运行atxserver2也可以看到效果
3. atxserver2-android-provider：接入安卓系统必须启动的项目，主要负责安卓设备和平台交互
4. atxserver2-ios-provider：接入IOS设备必须启动的项目，负责IOS设备和平台的交互工作

> 也有很多开源的设备控制工具，实现手机的群控功能，如：[https://github.com/Genymobile/scrcpy](https://github.com/Genymobile/scrcpy)，可以多多参考

**下边是部署过程：**

1. 安装rethinkdb：[安装手册](https://rethinkdb.com/docs/install/)，修改rethinkdb配置文件 default.conf，bind配置监听 0.0.0.0，拷贝到 instance.id 目录下，通过 `rethinkdb --config-file [配置文件]`，启动后，通过浏览器访问 `http://[IP:8080]`
2. 安装 atxserver2：github项目：[https://github.com/openatx/atxserver2](https://github.com/openatx/atxserver2)
3. 安装 atxserver2-android-provider：依赖nodejs，Github项目：[https://github.com/openatx/atxserver2-android-provider](https://github.com/openatx/atxserver2-android-provider)
4. atxserver2-android-provider 会通过adb track-devices自动发现已经接入的设备，推送以下安装包：
	1. minicap
	2. minitouch
	3. atx-agent
	4. app-uiautomator-[test].apk
	5. whatsinput-apk
5. 启动atxserver2：通过在项目中执行 `python main.py`，，默认监听4000
6. 启动provider：在项目中执行 `python main.py --server [atxserver2监听地址]`

> 官方测试，atxserver2可以同时控制约80台移动设备


### 1.7.2 atxserver2 安装部署
下边都是在Ubuntu环境下操作，所以首先要保证，在Ubuntu上是可以通过adb查看到移动设备的，如下：
![](https://raw.githubusercontent.com/BaihlUp/Figurebed/master/2023/202311161618803.png)

下载项目源代码：[https://github.com/openatx/atxserver2](https://github.com/openatx/atxserver2)，项目目录执行如下命令：
```python
docker-compose up
```
执行完以上命令后，可以看到atxserver2监听在4000端口：
![](https://raw.githubusercontent.com/BaihlUp/Figurebed/master/2023/202311161619857.png)

通过浏览器访问，此时设备列表为空：
![](https://raw.githubusercontent.com/BaihlUp/Figurebed/master/2023/202311161620713.png)


### 1.7.3 atxserver2-android-provider 部署
通过安装 atxserver2-android-provider 接入安卓设备，通过docker部署，如下：

```shell
SERVER_URL="http://192.168.170.137:4000" # 这个修改成自己的atxserver2地址
IMAGE="codeskyblue/atxserver2-android-provider"
docker pull $IMAGE
docker run --rm --privileged -v /dev/bus/usb:/dev/bus/usb --net host \
    ${IMAGE} python main.py --allow-remote --server ${SERVER_URL}
```
执行后如下：
![](https://raw.githubusercontent.com/BaihlUp/Figurebed/master/2023/202311161622672.png)
可以看到发现一个移动设备 `10.164.17.39:5555`，并且输出了设备的信息。

刷新浏览器可以看到有一个安卓移动端设备：
![](https://raw.githubusercontent.com/BaihlUp/Figurebed/master/2023/202311161623978.png)

点击进去以后可以看到界面如下：

![](https://raw.githubusercontent.com/BaihlUp/Figurebed/master/2023/202311161611982.png)



## 1.8 连接真实手机
手机连接到PC后，可能需要安装驱动。连接上以后通过adb查看如果没有显示devices，撤销下USE调试模式，然后再次查看。

连接真实手机后，使用adb操作，与操作模拟器中的手机一样。


- 通过Ubuntu系统连接真实手机

手机插到PC上，然后连接到Ubuntu虚拟机中，后边的操作与PC上一样。


# 2 实战项目
## 2.1 抓取抖音数据

### 2.1.1 操作APP
1. 操作APP进行观看短视频
2. 查看视频作者主页
3. 操作返回
4. 滑动到下一个视频
5. 循环以上操作

**代码如下：**
```python
import uiautomator2 as u2
import time


class Douyin(object):
    # 在__init__方法里面连接设备
    def __init__(self, serial="emulator-5554"):
        self.d = u2.connect_usb(serial=serial)
        self.start_app()
        self.handle_watcher()
        self.size = self.get_windowsize()
        # 是用来获取一个初始时间
        self.t0 = time.perf_counter()

    def start_app(self):
        """启动app"""
        self.d.app_start(package_name="com.ss.android.ugc.aweme")

    def stop_app(self):
        """app退出逻辑"""
        # 先关闭监视器
        self.d.watcher.stop()
        self.d.app_stop("com.ss.android.ugc.aweme")
        self.d.app_clear("com.ss.android.ugc.aweme")

    def stop_time(self):
        """停止时间"""
        # 时间是秒
        if time.perf_counter() - self.t0 > 50:
            return True

    def handle_watcher(self):
        """监视器"""
        # 通知权限
        self.d.watcher.when('//*[@resource-id="com.ss.android.ugc.aweme:id/a4r"]').click()
        # 发现滑动查看更多
        self.d.watcher.when('//*[@text="滑动查看更多"]').click()
        # 添加一个监控器
        self.d.watcher.when('//*[@text="快速进入TA的个人中心"]').click()
        # 监控器写好之后，一定要记得启动
        self.d.watcher.start(interval=1)

    def get_windowsize(self):
        """获取窗口大小"""
        return self.d.window_size()

    def swipe_douyin(self):
        """滑动抖音视频和点击视频发布者头像的操作"""
        # 来判断是否正常的进入到了视频页面，等待20s
        if self.d(resourceId="com.ss.android.ugc.aweme:id/yy", text="我").exists(timeout=20):
            while True:
                # 到规定的时间停止循环
                if self.stop_time():
                    self.stop_app()
                    return
                # 查看是不是正常的发布者，头像下有个 + 号
                if self.d(resourceId="com.ss.android.ugc.aweme:id/u0").exists:
                    # 是正常的发布者，点击头像
                    self.d(resourceId="com.ss.android.ugc.aweme:id/tw").click()
                    # 返回
                    self.d(resourceId="com.ss.android.ugc.aweme:id/et").click()

                if self.d(resourceId="com.ss.android.ugc.aweme:id/yy", text="我").exists and \
                        self.d(resourceId="com.ss.android.ugc.aweme:id/u0").exists:
                    # 进入正常的视频页面,开始滑动
                    x1 = int(self.size[0] * 0.5)
                    y1 = int(self.size[1] * 0.9)
                    y2 = int(self.size[1] * 0.15)
                    self.d.swipe(x1, y1, x1, y2)


if __name__ == '__main__':
    d = Douyin()
    d.swipe_douyin()
```


### 2.1.2 解析数据
使用 Fiddler 抓包看接口，看抖音的数据接口，知道接口以后，编写python脚本，使用mitmdump执行脚本进行数据爬取。
执行 `mitmdump.exe -s .\decode_douyin.py -p 8888`，指定代理端口 8888，在移动端浏览器输入 `mitm.it` 下载证书并加载。

- decode_douyin.py
```python
# 特别注意：
# 在新版本的抖音中，已经加密，无法获取数据
# 当前使用的是10.0版本的抖音app，大家一定要注意
# 使用抓包工具找到如下接口：
# 个人信息页接口
# https://aweme-eagle.snssdk.com/aweme/v1/user/?user_id
# 滑动视频的接口
# https://aweme-eagle.snssdk.com/aweme/v1/feed

import json

def response(flow):
    """解析10版本抖音app返回数据"""
    # 视频
    if 'https://aweme-eagle.snssdk.com/aweme/v1/feed' in flow.request.url:
        # 使用json来loadsresponse.text
        video_response = json.loads(flow.response.text)
        video_list = video_response.get("aweme_list", [])
        for item in video_list:
            print(item.get("desc"), "")
    # 发布者页面
    if 'https://aweme-eagle.snssdk.com/aweme/v1/user/?user_id' in flow.request.url:
        person_response = json.loads(flow.response.text)
        person_info = person_response.get("user", "")
        if person_info:
            info = {
                'nickname': person_info.get("nickname", ""),
                'total_favorited': person_info.get("total_favorited", 0),
                'following_count': person_info.get("following_count", 0),
                'douyin_id': person_info.get("unique_id", ""),
                'follower_count': person_info.get("follower_count", 0)
            }
            print(info)
```


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






































