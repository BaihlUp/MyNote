---
title: iterm2 +  oh-my-zsh 终端配置完整教程
date: 2024-01-13
categories:
  - 工具使用
tags:
  - iterm2
published: true
---
# 0 参考资料

[https://zhuanlan.zhihu.com/p/550022490](https://zhuanlan.zhihu.com/p/550022490)

# 2 安装oh-my-zsh
[oh-my-zsh](https://link.juejin.cn/?target=https%3A%2F%2Fgithub.com%2Fohmyzsh%2Fohmyzsh "https://github.com/ohmyzsh/ohmyzsh")更强大的命令行工具，解放双手，比系统自带bash更加酷炫、高效，可以实现更强大的命令补全，命令高亮等一系列酷炫功能。同时支持各种自定义选项，并支持扩展。

```bash
sh -c "$(curl -fsSL https://raw.github.com/robbyrussell/oh-my-zsh/master/tools/install.sh)"
```

## 2.1 安装配色库

```bash
git clone https://github.com/mbadolato/iTerm2-Color-Schemes.git
```
在schemes文件夹中找到 Solarized Dark Higher Contrast.itermcolors文件，此款主题配色最为流行，下面就以此主题为例进行导入和修改。

- 导入主题配色

打开配置页面，Profiles -> Colors -> Color Presets -> Import，选择到刚刚解压的主题文件。
![](https://raw.githubusercontent.com/BaihlUp/Figurebed/master/2023/20240113215608.png)

## 2.2 修改主题

[官方主题大全](https://link.juejin.cn/?target=https%3A%2F%2Fgithub.com%2Fohmyzsh%2Fohmyzsh%2Fwiki%2FThemes "https://github.com/ohmyzsh/ohmyzsh/wiki/Themes") `~/.zshrc`更换主题`ZSH_THEME="agnoster"`

![](https://raw.githubusercontent.com/BaihlUp/Figurebed/master/2023/20240113210154.png)

对所有的主题都很喜欢，可以按照上面的修改步骤，把主题修改为：
```bash
ZSH_THEME="random"
```
每启动一次终端，就会随机切换一个主题。
也可以选择几个最喜欢的主题配置在图片中红框这一行：
```bash
ZSH_THEME_RANDOM_CANDIDATES=( "robbyrussell" "agnoster" "ys")
```

也可以排除你不想要的主题，在配置文件增加如下代码，在括号中写上自己不喜欢的主题名称，以空格隔开：
```bash
ZSH_THEME_RANDOM_IGNORED=(pygmalion tjkirch_mod)
```

- 第三方主题

如果对自带的主题不太满意，可以进入下面的网站看下oh-my-zsh的部分第三方主题显示效果

[https://github.com/ohmyzsh/ohmyzsh/wiki/External-themes](https://github.com/ohmyzsh/ohmyzsh/wiki/External-themes)

## 2.3 安装字体库
iTerm2 修改主题之后，因为某些主题含有特殊字符或者表情，在操作的时候会出现乱码的情况，因此需要安装Meslo字体来兼容解决。

```bash
# clone
git clone https://github.com/powerline/fonts.git --depth=1
# install
cd fonts
./install.sh
# clean-up a bit
cd ..
rm -rf fonts
```
安装好之后，选择一款Powerline字体了：`iterm2 -> Preferences -> Profiles -> Text -> Font -> Change Font`（我用的是Meslo LG）
![](https://raw.githubusercontent.com/BaihlUp/Figurebed/master/2023/20240113210820.png)

## 2.4 安装插件
### 2.4.1 命令行高亮
此款插件在我们使用命令行的时候如果遇到特殊命令或者错误命令，会有高亮显示，可以及时进行提醒。

```bash
git clone https://github.com/zsh-users/zsh-syntax-highlighting.git ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-syntax-highlighting
```

在 `~/.zshrc` 中添加插件：
```bash
plugins=(zsh-syntax-highlighting)
```

### 2.4.2 命令自动补全
此款插件非常实用，大大加快了我们敲命令的速度。

```bash
git clone https://github.com/zsh-users/zsh-autosuggestions ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-autosuggestions

#编辑配置文件
vim ~/.zshrc

#找到plugins配置，在括号内增加zsh-autosuggestions,与其他插件之间使用空格分隔开
plugins=(zsh-autosuggestions)

#退出编辑后执行使配置生效
source ~/.zshrc
```

# 3 autojump自动跳转工具

## 3.1 安装

autojump提供了一种快速进行文件目录导航的方式。 它会把你在命令行中最常用的目录保存到一个数据库里，然后根据你访问的频次添加不同的权重。 访问越频繁，权重越高，排名就越先前，跳转的命令就越简洁。

**注意：目录在通过autojump跳转之前必须先访问，然后在autojump的数据库中才有记录**

j是autojump命令的简写，任何可以用autojump的地方都可以以j命令替换
```bash
#github镜像 
git clone https://github.com/wting/autojump.git

#进入目录，执行安装命令 
./install.py
```

在安装过程中，会在～/下建立.autojump文件夹：
```bash
#编辑配置文件
vim ~/.zshrc

#找到plugins配置，在括号内增加autojump,与其他插件之间使用空格分隔开
plugins=(autojump)

#在文件最后一行或者plugins=()后另起一行添加如下内容
[[ -s ~/.autojump/etc/profile.d/autojump.sh ]] && . ~/.autojump/etc/profile.d/autojump.sh

#退出编辑后执行使配置生效
source ~/.zshrc 
```

验证安装成功：
```bash
autojump --version
```

## 3.2 使用
1. **j** 跳转到指定目录下

```bash
j ~/Desktop/dxlWorkspace # 跳转到~/Desktop/dxlWorkspace目录下，下次直接输入 j dxl就可以直接跳转
```

2. **jo** 跳转到该目录，并使用终端打开，相当于`cd ~/Desktop/dxlWorkspace && open ./`

```bash
jo ~/Desktop/dxlWorkspace
```

3. 查看记忆权重
```bash
j --stat
```

# 4 个性操作
## 4.1 iTerm2快速隐藏和显示窗体
打开iTerm2，打开Preferences配置界面，Profiles → Keys →configure Hotkey window，自定义一个快捷键就可以了。
![](https://raw.githubusercontent.com/BaihlUp/Figurebed/master/2023/20240113223549.png)

![](https://raw.githubusercontent.com/BaihlUp/Figurebed/master/2023/20240113223622.png)

## 4.2 iTerm2隐藏用户名和主机名
通常在Shell中默认的我们的用户名和主机名，这两者加在一起会很长，操作的时候很影响观感，我们可以手动去除。
首先使用命令查看当前用户名称：
```bash
whoami
```

```bash
#编辑配置文件
vim ~/.zshrc

#在文件最后增加 DEFAULT_USER="xxxxx" 配置
DEFAULT_USER="xxxxx"

#退出编辑后执行使配置生效
source ~/.zshrc 
```
再次打开终端姓名和主机名就隐藏掉了。

## 4.3 设置 Status bar
iTerm2 提供了不少的 Status bar，开启后我们可以在终端的最上方非常方便的实时查看本机的一些信息。
打开iTerm2，打开Preferences配置界面，Profiles -> session-> 勾选 Status bar enable-> configure Status bar，选择自己想要的展示内容即可。
![](https://raw.githubusercontent.com/BaihlUp/Figurebed/master/2023/20240113223847.png)

![](https://raw.githubusercontent.com/BaihlUp/Figurebed/master/2023/20240113224037.png)

![](https://raw.githubusercontent.com/BaihlUp/Figurebed/master/2023/20240113223910.png)
## 4.4 光标选择

iterm提供了三种光标可供选择：_、|、[]。

打开iTerm2，打开Preferences配置界面，Profiles -> text-> cursor，选择自己想要的光标即可。
![](https://raw.githubusercontent.com/BaihlUp/Figurebed/master/2023/20240113224211.png)

## 4.5 窗口设置
打开iTerm2，打开Preferences配置界面，Profiles -> Window，根据自己的需求设置窗口透明度、背景图片、行列数以及风格等。
![](https://raw.githubusercontent.com/BaihlUp/Figurebed/master/2023/20240113224622.png)

## 4.6 Badge、Title、Icon
打开iTerm2，打开Preferences配置界面，Profiles -> General ，根据自己的需求设置Badge，点击edit按钮调整Badge位置和大小，Title和Icon选项是设置标签页标题和图标的，博主习惯性采用图片中的设置，各位看官可以根据自己的需求灵活设置。

![](https://raw.githubusercontent.com/BaihlUp/Figurebed/master/2023/20240113225516.png)

## 4.7 标签页配色
打开iTerm2，打开Preferences配置界面，Appearence -> General，将 Theme 改为 Minimal

![](https://raw.githubusercontent.com/BaihlUp/Figurebed/master/2023/20240113225913.png)

修改后效果：
![](https://raw.githubusercontent.com/BaihlUp/Figurebed/master/2023/20240113230030.png)

## 4.8 配置SSH快速连接
### 4.8.1 编写脚本方式
编写一个登录脚本，如下：
```bash
#!/usr/bin/expect
set timeout 30
spawn ssh [lindex $argv 0]@[lindex $argv 1] -p [lindex $argv 2]
expect {
        "(yes/no)?"
        {send "yes\n";exp_continue}
        "password:"
        {send "[lindex $argv 3]\n"}
}
interact
```

保存到一个位置，在iterm2中指定。
```bash
[lindex $argv 0]：服务器用户名
[lindex $argv 1]：服务器IP地址
[lindex $argv 2]：端口号
[lindex $argv 3]：服务器密码
```
打开iTerm2，打开Preferences配置界面，Profiles -> general，左下角点击+号，新建profile，参考下面图片在对应位置输入内容即可。
Name:根据需求输入，通常选择标识性较强的内容便于区分，例如服务器的IP地址
Command：这里选择login Shell
Send text at start ：填写格式形如A B C D E这样，每一个部分之间用空格隔开，根据自己实际情况填写,下面是对每一部分内容的解释

```text
A代表咱们上面写的本机保存sh脚本的路径：/Users/iterm/myserver.sh
B代表服务器用户名一般为：root
C代表服务器IP：根据腾讯云服务器对外暴露的公网IP填写
D代表服务器端口号一般远程连接端口为：22
E代表服务器密码：根据自己实际的服务器密码填写
```
![](https://raw.githubusercontent.com/BaihlUp/Figurebed/master/2023/20240113233317.png)
设置好之后打开iTerm2，点击profiles，点击前面自己新增的连接远程服务器的profile的名字


首次连接需要输入一次服务器密码，之后再连接就免密码登陆了。
### 4.8.2 iterm2 触发器方式
点击左上角`Iterm2`任务栏，依次选择`Preferences - Profile`：
![](https://raw.githubusercontent.com/BaihlUp/Figurebed/master/2023/20240113233637.png)

点击左下角的`+`新增一个配置项，在右边的command处输入ssh登录的命令：
```bash
ssh root@x.x.x.x -p xxxxx
```
然后把tab页面切换到`Advanced`，点击`Edit`进入触发器编辑页面：
![](https://raw.githubusercontent.com/BaihlUp/Figurebed/master/2023/20240113233753.png)

新弹框中新增一个触发器，触发器的作用是匹配终端输出的字符串然后执行相应动作。触发字符串是`password:`，Action选择`Send Text`，Parameters填入登录密码，密码最后以`\n`结束表示输完密码后换行：
![](https://raw.githubusercontent.com/BaihlUp/Figurebed/master/2023/20240113233821.png)

配置好后退出，在任务栏的`Profile`中选择创建好的配置就可以自动登录到设备了：
![](https://raw.githubusercontent.com/BaihlUp/Figurebed/master/2023/20240113233841.png)

## 4.9 设置终端历史行数
打开iTerm2，打开Preferences配置界面，Profiles -> Terminal，根须需求进行修改，如果想不限制行数可以勾选Unlimited scrollback：
![](https://raw.githubusercontent.com/BaihlUp/Figurebed/master/2023/20240113230910.png)
