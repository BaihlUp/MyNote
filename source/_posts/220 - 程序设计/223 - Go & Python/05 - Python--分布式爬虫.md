---
title: Python实战--分布式爬虫
date: 2023-10-26 16:27:41
tags:
  - 程序设计
  - Python
---

# 1 参考资料 
## 1.1 预备知识
1. html
2. css
3. xpath
4. 正则表达式

**chromedriver地址**：[https://chromedriver.chromium.org/downloads](https://chromedriver.chromium.org/downloads)
https://www.lfd.uci.edu/~gohlke/pythonlibs/

# 2 基础知识
## 2.1 XPath 选择器
### 2.1.1 XPath 语法
- **测试样本**

```xml
<?xml version="1.0" encoding="UTF-8"?>
 
<bookstore>
 
<book>
  <title lang="eng">Harry Potter</title>
  <price>29.99</price>
</book>
 
<book>
  <title lang="eng">Learning XML</title>
  <price>39.95</price>
</book>
 
</bookstore>
```


- **路径表达式**

|表达式|描述|
|---|---|
|nodename|选取此节点的所有子节点。|
|/|从根节点选取（取子节点）。|
|//|从匹配选择的当前节点选择文档中的节点，而不考虑它们的位置（取子孙节点）。|
|.|选取当前节点。|
|..|选取当前节点的父节点。|
|@|选取属性。|

- **路径表达式以及表达式的结果：**

|路径表达式|结果|
|---|---|
|bookstore|选取 bookstore 元素的所有子节点。|
|/bookstore|选取根元素 bookstore。<br><br>注释：假如路径起始于正斜杠( / )，则此路径始终代表到某元素的绝对路径！|
|bookstore/book|选取属于 bookstore 的子元素的所有 book 元素。|
|//book|选取所有 book 子元素，而不管它们在文档中的位置。|
|bookstore//book|选择属于 bookstore 元素的后代的所有 book 元素，而不管它们位于 bookstore 之下的什么位置。|
|//@lang|选取名为 lang 的所有属性。|

- **谓语**

谓语用来查找某个特定的节点或者包含某个指定的值的节点，谓语被嵌在方括号中。

|路径表达式|结果|
|---|---|
|/bookstore/book[1]|选取属于 bookstore 子元素的第一个 book 元素。|
|/bookstore/book[last()]|选取属于 bookstore 子元素的最后一个 book 元素。|
|/bookstore/book[last()-1]|选取属于 bookstore 子元素的倒数第二个 book 元素。|
|/bookstore/book[position()<3]|选取最前面的两个属于 bookstore 元素的子元素的 book 元素。|
|//title[@lang]|选取所有拥有名为 lang 的属性的 title 元素。|
|//title[@lang='eng']|选取所有 title 元素，且这些元素拥有值为 eng 的 lang 属性。|
|/bookstore/book[price>35.00]|选取 bookstore 元素的所有 book 元素，且其中的 price 元素的值须大于 35.00。|
|/bookstore/book[price>35.00]//title|选取 bookstore 元素中的 book 元素的所有 title 元素，且其中的 price 元素的值须大于 35.00。|

- **选取未知节点**

通过通配符选取未知的XML元素。

|通配符|描述|
|---|---|
|*|匹配任何元素节点。|
|@*|匹配任何属性节点。|
|node()|匹配任何类型的节点。|

**示例：**

|路径表达式|结果|
|---|---|
|/bookstore/*|选取 bookstore 元素的所有子元素。|
|//*|选取文档中的所有元素。|
|//title[@*]|选取所有带有属性的 title 元素。|

- ** 选取若干路径**

通过在路径表达式中使用"|"运算符，您可以选取若干个路径。

|路径表达式|结果|
|---|---|
|//book/title \| //book/price|选取 book 元素的所有 title 和 price 元素。|
|//title \| //price|选取文档中的所有 title 和 price 元素。|
|/bookstore/book/title \| //price|选取属于 bookstore 元素的 book 元素的所有 title 元素，以及文档中所有的 price 元素。|

### 2.1.2 XPath 轴

|轴名称|结果|
|---|---|
|ancestor|选取当前节点的所有先辈（父、祖父等）。|
|ancestor-or-self|选取当前节点的所有先辈（父、祖父等）以及当前节点本身。|
|attribute|选取当前节点的所有属性。|
|child|选取当前节点的所有子元素。|
|descendant|选取当前节点的所有后代元素（子、孙等）。|
|descendant-or-self|选取当前节点的所有后代元素（子、孙等）以及当前节点本身。|
|following|选取文档中当前节点的结束标签之后的所有节点。|
|following-sibling|选取当前节点之后的所有兄弟节点|
|namespace|选取当前节点的所有命名空间节点。|
|parent|选取当前节点的父节点。|
|preceding|选取文档中当前节点的开始标签之前的所有节点。|
|preceding-sibling|选取当前节点之前的所有同级节点。|
|self|选取当前节点。|

### 2.1.3 XPath 运算符
XPath 表达式可返回节点集、字符串、逻辑值以及数字。

|运算符|描述|示例|
|---|---|---|
|**数值运算符**|||
|`+`|加法|`//price + 5`|
|`-`|减法|`//quantity - 2`|
|`*`|乘法|`//price * //quantity`|
|`div`|除法|`//total_price div 2`|
|**比较运算符**|||
|`=`|等于|`//title = 'Introduction to XPath'`|
|`!=`|不等于|`//author != 'John Doe'`|
|`<`|小于|`//price < 20`|
|`<=`|小于等于|`//quantity <= 10`|
|`>`|大于|`//price > 30`|
|`>=`|大于等于|`//quantity >= 100`|
|**逻辑运算符**|||
|`and`|逻辑与|`//price > 20 and //author = 'John Doe'`|
|`or`|逻辑或|`//price < 10 or //quantity > 100`|
|`not`|逻辑非|`not(//price > 30)`|
|**模糊匹配运算符**|||
|`contains()`|判断字符串是否包含|`//title[contains(text(), 'XPath')]`|
|`starts-with()`|判断字符串是否以指定前缀|`//author[starts-with(text(), 'John')]`|
|`ends-with()`|判断字符串是否以指定后缀|`//author[ends-with(text(), 'Doe')]`|
|**多重条件运算符**|||
|多条件组合|使用括号组合多个条件|`//price > 20 and (//author = 'John Doe' or //author = 'Jane Smith')`|

### 2.1.4 示例


```python
titles = root.xpath("//book/title = 'Introduction to Python'")  
print("Titles:", titles)  # True

# 查询所有价格低于30的书的作者  
cheap_books_authors = root.xpath("//book[price+1 < 30]/author/text()")  
print("\nAuthors of Cheap Books:")  
for author in cheap_books_authors:  
    print(author)

lang_attr = root.xpath("//book/title/text() | //book/price/text()")  
print("\nattr: ")  
for attr in lang_attr:  
    print(attr)
```


## 2.2 CSS 选择器

## 2.3 Selenium 使用
一个用于自动化浏览器操作的工具，常用于网站测试和网页数据抓取。Selenium可以基于对应的浏览器driver自动化操作浏览器，比如使用 chromedriver 操作chrome浏览器，需要先下载与本地安装的chrome版本对应的 chromedriver，否则无法使用。
```python
pip install selenium
pip install webdriver-manager  # 自动下载与本地浏览起同版本的driver
```

**使用示例：**

```python
from selenium import webdriver  
from selenium.webdriver.common.by import By  
from selenium .webdriver.chrome.service import Service  
  
if __name__ == "__main__":  
	# 自动下载chromedriver，下载后可以保存到本地，下次直接使用selenium调用
    # from webdriver_manager.chrome import ChromeDriverManager  
    # print(ChromeDriverManager().install())  
    service = Service("chromedriver-mac-arm64/chromedriver")  
    browser = webdriver.Chrome(service=service)  
    browser.get("https://www.zhihu.com/#signin")  # 模拟自动访问zhihu
  
    browser.find_element(By.CSS_SELECTOR, ".view-signin input[name='account']").send_keys("18782902568")  
    browser.find_element(By.CSS_SELECTOR, ".view-signin input[name='password']").send_keys("admin125")
```



# 3 Scrapy 的应用
**Scrapy使用手册：** [https://www.osgeo.cn/scrapy/](https://www.osgeo.cn/scrapy/)
## 3.1 安装配置
- **Ubuntu安装方式**

```bash
sudo apt-get install python-dev python-pip libxml2-dev libxslt1-dev zlib1g-dev libffi-dev libssl-dev
```

- 通过pip安装 Scrapy 框架

```bash
sudo pip install scrapy
```

安装后，在终端执行scrapy，如下：
![](https://raw.githubusercontent.com/BaihlUp/Figurebed/master/2023/20231026094902.png)

## 3.2 Scrapy介绍
### 3.2.1 Scrapy架构
**Scrapy架构图（绿线是数据流向）：**

![](https://raw.githubusercontent.com/BaihlUp/Figurebed/master/2023/20231026093935.png)

![](https://raw.githubusercontent.com/BaihlUp/Figurebed/master/2023/Pasted%20image%2020231024191715.png)

- **Scrapy Engine(引擎)**: 负责Spider、ItemPipeline、Downloader、Scheduler中间的通讯，信号、数据传递等。
- **Scheduler(调度器)**: 它负责接受引擎发送过来的Request请求，并按照一定的方式进行整理排列，入队，当引擎需要时，交还给引擎。
- **Downloader（下载器）**：负责下载Scrapy Engine(引擎)发送的所有Requests请求，并将其获取到的Responses交还给Scrapy Engine(引擎)，由引擎交给Spider来处理，
- **Spider（爬虫）**：它负责处理所有Responses,从中分析提取数据，获取Item字段需要的数据，并将需要跟进的URL提交给引擎，再次进入Scheduler(调度器).
- **Item Pipeline(管道)**：它负责处理Spider中获取到的Item，并进行进行后期处理（详细分析、过滤、存储等）的地方。
- **Downloader Middlewares（下载中间件）**：你可以当作是一个可以自定义扩展下载功能的组件。
- **Spider Middlewares（Spider中间件）**：你可以理解为是一个可以自定扩展和操作引擎和Spider中间通信的功能组件（比如进入Spider的Responses;和从Spider出去的Requests）

### 3.2.2 Scrapy的运行流程
代码写好，程序开始运行...

- 1 引擎：Hi！Spider, 你要处理哪一个网站？
- 2 Spider：老大要我处理xxxx.com。
- 3 引擎：你把第一个需要处理的URL给我吧。
- 4 Spider：给你，第一个URL是xxxxxxx.com。
- 5 引擎：Hi！调度器，我这有request请求你帮我排序入队一下。
- 6 调度器：好的，正在处理你等一下。
- 7 引擎：Hi！调度器，把你处理好的request请求给我。
- 8 调度器：给你，这是我处理好的request
- 9 引擎：Hi！下载器，你按照老大的下载中间件的设置帮我下载一下这个request请求
- 10 下载器：好的！给你，这是下载好的东西。（如果失败：sorry，这个request下载失败了。然后引擎告诉调度器，这个request下载失败了，你记录一下，我们待会儿再下载）
- 11 引擎：Hi！Spider，这是下载好的东西，并且已经按照老大的下载中间件处理过了，你自己处理一下（注意！这儿responses默认是交给def parse()这个函数处理的）
- 12 Spider：（处理完毕数据之后对于需要跟进的URL），Hi！引擎，我这里有两个结果，这个是我需要跟进的URL，还有这个是我获取到的Item数据。
- 13 引擎：Hi ！管道 我这儿有个item你帮我处理一下！调度器！这是需要跟进URL你帮我处理下。然后从第四步开始循环，直到获取完老大需要全部信息。
- 14 管道调度器：好的，现在就做！

> 注意！只有当调度器中不存在任何request了，整个程序才会停止，（也就是说，对于下载失败的URL，Scrapy也会重新下载。）

- **制作Scrapy爬虫一共需要4步：**

1. 新建项目 (scrapy startproject xxx)：新建一个新的爬虫项目
2. 明确目标 （编写items.py）：明确你想要抓取的目标
3. 制作爬虫 （spiders/xxspider.py）：制作爬虫开始爬取网页
4. 存储内容 （pipelines.py）：设计管道存储爬取内容

## 3.3 Scrapy入门
### 3.3.1 新建项目（scrapy startproject）
创建一个新的Scrapy项目：

```bash
scrapy startproject mySpider
```

其中， mySpider 为项目名称，可以看到将会创建一个 mySpider 文件夹，目录结构大致如下，下面来简单介绍一下各个主要文件的作用：

```bash
mySpider/
    scrapy.cfg
    mySpider/
        __init__.py
        items.py
        pipelines.py
        settings.py
        spiders/
            __init__.py
            ...
```
- scrapy.cfg: 项目的配置文件。
- mySpider/: 项目的Python模块，将会从这里引用代码。
- mySpider/items.py: 项目的目标文件。
- mySpider/pipelines.py: 项目的管道文件。
- mySpider/settings.py: 项目的设置文件。
- mySpider/spiders/: 存储爬虫代码目录。

### 3.3.2 明确目标（mySpider/items.py）
打算抓取 **http://www.itcast.cn/channel/teacher.shtml** 网站里的所有讲师的姓名、职称和个人信息。
1. 打开 mySpider 目录下的 items.py。
2. Item 定义结构化数据字段，用来保存爬取到的数据，有点像 Python 中的 dict，但是提供了一些额外的保护减少错误。
3. 可以通过创建一个 scrapy.Item 类， 并且定义类型为 scrapy.Field 的类属性来定义一个 Item（可以理解成类似于 ORM 的映射关系）。

```python
import scrapy

class ItcastItem(scrapy.Item):
   name = scrapy.Field()
   title = scrapy.Field()
   info = scrapy.Field()
```

### 3.3.3 制作爬虫（spiders/itcastSpider.py）
1. 生成爬虫

在当前目录下输入命令，将在mySpider/spider目录下创建一个名为itcast的爬虫，并指定爬取域的范围：
```bash
scrapy genspider itcast "itcast.cn"
```

打开 mySpider/spider目录里的 itcast.py，默认增加了下列代码:

```python
import scrapy

class ItcastSpider(scrapy.Spider):
    name = "itcast"
    allowed_domains = ["itcast.cn"]
    start_urls = (
        'http://www.itcast.cn/',
    )

    def parse(self, response):
        pass
```
- 要建立一个Spider， 你必须用scrapy.Spider类创建一个子类，并确定了三个强制的属性 和 一个方法。
- name = "" ：这个爬虫的识别名称，必须是唯一的，在不同的爬虫必须定义不同的名字。
- allow_domains = [] 是搜索的域名范围，也就是爬虫的约束区域，规定爬虫只爬取这个域名下的网页，不存在的URL会被忽略。
- start_urls = () ：爬取的URL元祖/列表。爬虫从这里开始抓取数据，所以，第一次下载的数据将会从这些urls开始。其他子URL将会从这些起始URL中继承性生成。
- parse(self, response) ：解析的方法，每个初始URL完成下载后将被调用，调用的时候传入从每一个URL传回的Response对象来作为唯一参数，主要作用如下：
- 负责解析返回的网页数据(response.body)，提取结构化数据(生成item) 

生成需要下一页的URL请求。将start_urls的值修改为需要爬取的第一个url：

```bash
start_urls = ("http://www.itcast.cn/channel/teacher.shtml",)
```

修改parse()方法：

```python
def parse(self, response):
    filename = "teacher.html"
    open(filename, 'w').write(response.body)
```
然后运行一下看看，在mySpider目录下执行：

```bash
scrapy crawl itcast
```
`itcast`是`scrapy genspider`命令命名的唯一爬虫名。

2. 取数据

取数据可以通过XPath等方式解析响应的页面，提取需要的数据。之前在 mySpider/items.py 里定义了一个 ItcastItem 类。 这里引入进来：

```bash
from mySpider.items import ItcastItem
```

将得到的数据封装到一个 ItcastItem 对象中，可以保存每个老师的属性：
```python
from mySpider.items import ItcastItem

def parse(self, response):
    #open("teacher.html","wb").write(response.body).close()

    # 存放老师信息的集合
    items = []

    for each in response.xpath("//div[@class='li_txt']"):
        # 将我们得到的数据封装到一个 `ItcastItem` 对象
        item = ItcastItem()
        #extract()方法返回的都是unicode字符串
        name = each.xpath("h3/text()").extract()
        title = each.xpath("h4/text()").extract()
        info = each.xpath("p/text()").extract()

        #xpath返回的是包含一个元素的列表
        item['name'] = name[0]
        item['title'] = title[0]
        item['info'] = info[0]

        items.append(item)

    # 直接返回最后数据
    return items
```
Item中的数据可以通过管道（Pipeline）进行处理。

### 3.3.4 保存数据
scrapy保存信息的最简单的方法主要有四种，-o 输出指定格式的文件，命令如下：

```bash
scrapy crawl itcast -o teachers.json
```

csv 逗号表达式，可用Excel打开：

```bash
scrapy crawl itcast -o teachers.csv
```

xml格式：

```bash
scrapy crawl itcast -o teachers.xml
```

## 3.4 Scrapy各组件
### 3.4.1 Spider
通过Scrapy创建的不同的爬虫项目，都有一个唯一的名字，

### 3.4.2 Items

### 3.4.3 Item Loaders

### 3.4.4 Item Pipeline

### 3.4.5 Feed exports


# CrawlSpider


## 机器学习识别
EasyDL


## 8 Scrapy突破反爬的限制
![](https://raw.githubusercontent.com/BaihlUp/Figurebed/master/2023/Pasted%20image%2020231024200047.png)


