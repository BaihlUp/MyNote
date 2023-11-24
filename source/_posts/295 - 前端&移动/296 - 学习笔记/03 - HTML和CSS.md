---
title: HTML和CSS
date: 2023-11-24
categories:
  - 前端&移动
tags:
  - 前端
  - HTML
  - CSS
published: false
---
# 1 HTML 知识

编辑器工具：VS Code 可以安装 Live Preview 插件来实时预览编写的代码

## 1.1 HTML简介

- HTML 指的是超文本标记语言: **H**yper**T**ext **M**arkup **L**anguage
- HTML 不是一种编程语言，而是一种**标记**语言
- 标记语言是一套**标记标签** (markup tag)
- HTML 使用标记标签来**描述**网页
- HTML 文档包含了HTML **标签**及**文本**内容
- HTML文档也叫做 **web 页面**

- HTML基本框架

![](https://raw.githubusercontent.com/BaihlUp/Figurebed/master/2023/202311240948381.png)


```
1. <!DOCTYPE html> 声明为 HTML5 文档
2. <html> 元素是 HTML 页面的根元素
3. <head> 元素包含了文档的元（meta）数据，如 <meta charset="utf-8"> 定义网页编码格式为 **utf-8**。
4. <title> 元素描述了文档的标题
5. <body> 元素包含了可见的页面内容
6. <h1> 元素定义一个大标题
7. <p> 元素定义一个段落
```


目前在大部分浏览器中，直接输出中文会出现中文乱码的情况，这时候我们就需要在头部将字符声明为 UTF-8 或 GBK。

```html
<!DOCTYPE html>
<html>
<head>
<meta charset="UTF-8">
<title>
页面标题</title>
</head>
<body>
 
<h1>我的第一个标题</h1>
 
<p>我的第一个段落。</p>
 
</body>
</html>
```

## 1.2 HTML元素和属性
**HTML元素和标签：**

[https://www.runoob.com/tags/html-reference.html](https://www.runoob.com/tags/html-reference.html)



**HTML相关属性：**

|  属性 | 描述  |
|---|---|
|class|为html元素定义一个或多个类名（classname）(类名从样式文件引入)|
|id|定义元素的唯一id|
|style|规定元素的行内样式（inline style）|
|title|描述了元素的额外信息 (作为工具条使用)|
|[accesskey](https://www.runoob.com/tags/att-global-accesskey.html "HTML Global accesskey 属性")|设置访问元素的键盘快捷键。|
|[class](https://www.runoob.com/tags/att-global-class.html "HTML Global class 属性")|规定元素的类名（classname）|
|[contenteditable](https://www.runoob.com/tags/att-global-contenteditable.html "HTML Global contenteditable 属性")New|规定是否可编辑元素的内容。|
|[contextmenu](https://www.runoob.com/tags/att-global-contextmenu.html "HTML contextmenu 属性")New|指定一个元素的上下文菜单。当用户右击该元素，出现上下文菜单|
|[data-*](https://www.runoob.com/tags/att-global-data.html)New|用于存储页面的自定义数据|
|[dir](https://www.runoob.com/tags/att-global-dir.html "HTML dir 属性")|设置元素中内容的文本方向。|
|[draggable](https://www.runoob.com/tags/att-global-draggable.html "HTML draggable 属性")New|指定某个元素是否可以拖动|
|[dropzone](https://www.runoob.com/tags/att-global-dropzone.html "HTML dropzone 属性")New|指定是否将数据复制，移动，或链接，或删除|
|[hidden](https://www.runoob.com/tags/att-global-hidden.html "HTML hidden 属性")New|hidden 属性规定对元素进行隐藏。|
|[id](https://www.runoob.com/tags/att-global-id.html "HTML id 属性")|规定元素的唯一 id|
|[lang](https://www.runoob.com/tags/att-global-lang.html "HTML lang 属性")|设置元素中内容的语言代码。|
|[spellcheck](https://www.runoob.com/tags/att-global-spellcheck.html "HTML spellcheck 属性")New|检测元素是否拼写错误|
|[style](https://www.runoob.com/tags/att-global-style.html "HTML style 属性")|规定元素的行内样式（inline style）|
|[tabindex](https://www.runoob.com/tags/att-global-tabindex.html "HTML tabindex 属性")|设置元素的 Tab 键控制次序。|
|[title](https://www.runoob.com/tags/att-global-title.html "HTML title 属性")|规定元素的额外信息（可在工具提示中显示）|
|[translate](https://www.runoob.com/tags/att-global-translate.html "HTML translate 属性")New|指定是否一个元素的值在页面载入时是否需要翻译|

## 1.3 HTML链接

以下是 HTML 中创建链接的基本语法和属性：`code><a>` 元素：创建链接的主要HTML元素是`<a>`（锚）元素。`<a>`元素具有以下属性：

- `href`：指定链接目标的URL，这是链接的最重要属性。可以是另一个网页的URL、文件的URL或其他资源的URL。
- `target`（可选）：指定链接如何在浏览器中打开。常见的值包括 `_blank`（在新标签或窗口中打开链接）和 `_self`（在当前标签或窗口中打开链接）。
- `title`（可选）：提供链接的额外信息，通常在鼠标悬停在链接上时显示为工具提示。
- `rel`（可选）：指定与链接目标的关系，如 nofollow、noopener 等。

**文本链接：** 最常见的链接类型是文本链接，它使用 <a> 元素将一段文本转化为可点击的链接，例如：

```html
<a href="https://www.example.com">访问示例网站</a>
```
**图像链接：** 可以使用图像作为链接。在这种情况下，<a> 元素包围着 <img> 元素。例如：

```html
<a href="https://www.example.com">
  <img src="example.jpg" alt="示例图片">
</a>
```

**锚点链接：**除了链接到其他网页外，您还可以在同一页面内创建内部链接，这称为锚点链接。要创建锚点链接，需要在目标位置使用 <a> 元素定义一个标记，并使用#符号引用该标记。例如：
```html
<a href="#section2">跳转到第二部分</a>
<!-- 在页面中的某个位置 -->
<a name="section2"></a>
```

**下载链接：**如果您希望链接用于下载文件而不是导航到另一个网页，可以使用 download 属性。例如：
```html
<a href="document.pdf" download>下载文档</a>
```

- target属性

使用 target 属性，你可以定义被链接的文档在何处显示。下面的这行会在新窗口打开文档：
```html
<a href="https://www.runoob.com/" target="_blank" rel="noopener noreferrer">访问菜鸟教程!</a>
```

- id 属性

id 属性可用于创建一个 HTML 文档书签。


## 1.4 HTML 头部

```
head> 元素包含了所有的头部标签元素。在 <head>元素中你可以插入脚本（scripts）, 样式文件（CSS），及各种meta信息。

可以添加在头部区域的元素标签为: <title>, <style>, <meta>, <link>, <script>, <noscript> 和 <base>。

<title> 标签定义了不同文档的标题。
<base> 标签描述了基本的链接地址/链接目标，该标签作为HTML文档中所有的链接标签的默认链接
<link> 标签定义了文档与外部资源之间的关系。
<style> 标签定义了HTML文档的样式文件引用地址，在<style> 元素中你也可以直接添加样式来渲染 HTML 文档:
meta 标签描述了一些基本的元数据。<meta> 标签提供了元数据.元数据也不显示在页面上，但会被浏览器解析。META 元素通常用于指定网页的描述，关键词，文件的最后修改时间，作者，和其他元数据。元数据可以使用于浏览器（如何显示内容或重新加载页面），搜索引擎（关键词），或其他Web服务。<meta> 一般放置于 <head> 区域
<script>标签用于加载脚本文件，如： JavaScript。
```

```html
<head>
<base href="http://www.runoob.com/images/" target="_blank">
</head>
```
```html
<head>
<style type="text/css">
body {
    background-color:yellow;
}
p {
    color:blue
}
</style>
</head>
```

## 1.5 HTML CSS
```
- 内联样式- 在HTML元素中使用"style" **属性**
- 内部样式表 -在HTML文档头部 <head> 区域使用<style> **元素** 来包含CSS
- 外部引用 - 使用外部 CSS **文件**
```

- 内联样式

背景色属性（background-color）定义一个元素的背景颜色：
```html
<body style="background-color:yellow;">
<h2 style="background-color:red;">这是一个标题</h2>
<p style="background-color:green;">这是一个段落。</p>
</body>
```

- 内部样式表

当单个文件需要特别样式时，就可以使用内部样式表。你可以在`<head>` 部分通过 `<style>`标签定义内部样式表:
```html
<head>
<style type="text/css">
body {background-color:yellow;}
p {color:blue;}
</style>
</head>
```

- 外部样式表

当样式需要被应用到很多页面的时候，外部样式表将是理想的选择。使用外部样式表，你就可以通过更改一个文件来改变整个站点的外观。
```html
<head>
<link rel="stylesheet" type="text/css" href="mystyle.css">
</head>
```

- HTML 样式标签

|标签|描述|
|:--|:--|
|[`<style>`](https://www.runoob.com/tags/tag-style.html)|定义文本样式|
|[`<link>`](https://www.runoob.com/tags/tag-link.html)|定义资源引用地址|

## 1.6 HTML 列表

|标签|描述|
|:--|:--|
|[`<table>`](https://www.runoob.com/tags/tag-table.html)|定义表格|
|[`<th>`](https://www.runoob.com/tags/tag-th.html)|定义表格的表头|
|[`<tr>`](https://www.runoob.com/tags/tag-tr.html)|定义表格的行|
|[`<td>`](https://www.runoob.com/tags/tag-td.html)|定义表格单元|
|[`<caption>`](https://www.runoob.com/tags/tag-caption.html)|定义表格标题|
|[`<colgroup>`](https://www.runoob.com/tags/tag-colgroup.html)|定义表格列的组|
|[`<col>`](https://www.runoob.com/tags/tag-col.html)|定义用于表格列的属性|
|[`<thead>`](https://www.runoob.com/tags/tag-thead.html)|定义表格的页眉|
|[`<tbody>`](https://www.runoob.com/tags/tag-tbody.html)|定义表格的主体|
|[`<tfoot>`](https://www.runoob.com/tags/tag-tfoot.html)|定义表格的页脚|

## 1.7 HTML 列表
|标签|描述|
|:--|:--|
|[`<ol>`](https://www.runoob.com/tags/tag-ol.html)|定义有序列表|
|[`<ul>`](https://www.runoob.com/tags/tag-ul.html)|定义无序列表|
|[`<li>`](https://www.runoob.com/tags/tag-li.html)|定义列表项|
|[`<dl>`](https://www.runoob.com/tags/tag-dl.html)|定义列表|
|[`<dt>`](https://www.runoob.com/tags/tag-dt.html)|自定义列表项目|
|[`<dd>`](https://www.runoob.com/tags/tag-dd.html)|定义自定列表项的描述|


## 1.8 HTML区块
|标签|描述|
|:--|:--|
|[`<div>`](https://www.runoob.com/tags/tag-div.html)|定义了文档的区域，块级 (block-level)|
|[`<span>`](https://www.runoob.com/tags/tag-span.html)|用来组合文档中的行内元素， 内联元素(inline)|

## 1.9 HTML 布局
|标签|描述|
|:--|:--|
|---|---|
|[`<div>`](https://www.runoob.com/tags/tag-div.html)|定义文档区块，块级(block-level)|
|[`<span>`](https://www.runoob.com/tags/tag-span.html)|定义 span，用来组合文档中的行内元素。|

**实例：**

```html
<!DOCTYPE html>
<html>
<head> 
<meta charset="utf-8"> 
<title>菜鸟教程(runoob.com)</title> 
</head>
<body>
 
<div id="container" style="width:500px">
 
<div id="header" style="background-color:#FFA500;">
<h1 style="margin-bottom:0;">主要的网页标题</h1></div>
 
<div id="menu" style="background-color:#FFD700;height:200px;width:100px;float:left;">
<b>菜单</b><br>
HTML<br>
CSS<br>
JavaScript</div>
 
<div id="content" style="background-color:#EEEEEE;height:200px;width:400px;float:left;">
内容在这里</div>
 
<div id="footer" style="background-color:#FFA500;clear:both;text-align:center;">
版权 © runoob.com</div>
 
</div>
 
</body>
</html>
```
上边的HTML代码会产生如下结果：
![](https://raw.githubusercontent.com/BaihlUp/Figurebed/master/2023/202311241056410.png)


## 1.10 HTML 表单
HTML 表单用于收集用户的输入信息。
HTML 表单表示文档中的一个区域，此区域包含交互控件，将用户收集到的信息发送到 Web 服务器。
HTML 表单通常包含各种输入字段、复选框、单选按钮、下拉列表等元素。
以下是一个简单的HTML表单的例子：

- `<form>` 元素用于创建表单，`action` 属性定义了表单数据提交的目标 URL，`method` 属性定义了提交数据的 HTTP 方法（这里使用的是 "post"）。
- `<label>` 元素用于为表单元素添加标签，提高可访问性。
- `<input>` 元素是最常用的表单元素之一，它可以创建文本输入框、密码框、单选按钮、复选框等。`type` 属性定义了输入框的类型，`id` 属性用于关联 `<label>` 元素，`name` 属性用于标识表单字段。
- `<select>` 元素用于创建下拉列表，而 `<option>` 元素用于定义下拉列表中的选项。

```html
<form action="/" method="post">
    <!-- 文本输入框 -->
    <label for="name">用户名:</label>
    <input type="text" id="name" name="name" required>

    <br>

    <!-- 密码输入框 -->
    <label for="password">密码:</label>
    <input type="password" id="password" name="password" required>

    <br>

    <!-- 单选按钮 -->
    <label>性别:</label>
    <input type="radio" id="male" name="gender" value="male" checked>
    <label for="male">男</label>
    <input type="radio" id="female" name="gender" value="female">
    <label for="female">女</label>

    <br>

    <!-- 复选框 -->
    <input type="checkbox" id="subscribe" name="subscribe" checked>
    <label for="subscribe">订阅推送信息</label>

    <br>

    <!-- 下拉列表 -->
    <label for="country">国家:</label>
    <select id="country" name="country">
        <option value="cn">CN</option>
        <option value="usa">USA</option>
        <option value="uk">UK</option>
    </select>

    <br>

    <!-- 提交按钮 -->
    <input type="submit" value="提交">
</form>
```

## 1.11 HTML 脚本
`<script>` 标签用于定义客户端脚本
`<noscript>` 标签应对不支持或禁用脚本的浏览器

**HTML 中插入脚本：**

```html
<!DOCTYPE html>
<html>
<head> 
<meta charset="utf-8"> 
<title>菜鸟教程(runoob.com)</title> 
</head>
<body>

<h1>我的第一个 JavaScript </h1>

<p id="demo">
JavaScript 可以触发事件，就像按钮点击。</p>

<script>
function myFunction()
{
	document.getElementById("demo").innerHTML="Hello JavaScript!";
}
</script>

<button type="button" onclick="myFunction()">点我</button>

</body>
</html>
```

## 1.12 HTML 速查表

[https://www.runoob.com/html/html-quicklist.html](https://www.runoob.com/html/html-quicklist.html)


# 2 CSS 
## 2.1 简介
CSS 规则由两个主要的部分构成：选择器，以及一条或多条声明：
![](https://raw.githubusercontent.com/BaihlUp/Figurebed/master/2023/202311241132217.png)

选择器通常是您需要改变样式的 HTML 元素。
每条声明由一个属性和一个值组成。
属性（property）是您希望设置的样式属性（style attribute）。每个属性有一个值。属性和值被冒号分开。

## 2.2 Id 和 Class 选择器
### 2.2.1 id 选择器
HTML元素以id属性来设置id选择器,CSS 中 id 选择器以 "#" 来定义。
样式规则应用于元素属性 `id="para1"`:
```
#para1
{
	text-align:center;
	color:red;
} 
```

### 2.2.2 class 选择器

- 在以下实例中, 所有的 p 元素使用 `class="center"` 让该元素的文本居中:

```css
p.center {text-align:center;}
```

- **多个 class 选择器使用空格分开**

```html
<!DOCTYPE html>
<html>
<head>
<meta charset="utf-8"> 
<title>菜鸟教程(runoob.com)</title> 
<style>
.center {
	text-align:center;
}
.color {
	color:#ff0000;
}
</style>
</head>

<body>
<h1 class="center">标题居中</h1>
<p class="center color">段落居中，颜色为红色。</p> 
</body>
</html>
```