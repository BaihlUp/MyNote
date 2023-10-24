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



# 3 Scrapy 的使用
## 3.1 安装配置


## 3.2 Scrapy介绍

1. scrapy shell 命令进行调试

```python
# Obey robots.txt rules  
ROBOTSTXT_OBEY = False
```
创建爬虫：
```python
scrapy genspider jobbole new.cnblogs.com
```


## 机器学习识别
EasyDL


## 8 Scrapy突破反爬的限制
![[Pasted image 20231024200047.png]]

网站数据动态填充

![[Pasted image 20231024190906.png]]

![[Pasted image 20231024191715.png]]