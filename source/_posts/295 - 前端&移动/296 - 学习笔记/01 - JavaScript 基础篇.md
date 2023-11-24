---
title: JavaScript 基础篇
date: 2023-11-24
categories:
  - 前端&移动
tags:
  - 前端
  - JavaScript
published: false
---




# 1 JavaScript 基础知识
## 1.1 JavaScript 数据类型

**值类型(基本类型)**：字符串（String）、数字(Number)、布尔(Boolean)、空（Null）、未定义（Undefined）、Symbol。
**引用数据类型（对象类型）**：对象(Object)、数组(Array)、函数(Function)，还有两个特殊的对象：正则（RegExp）和日期（Date）。
![](https://raw.githubusercontent.com/BaihlUp/Figurebed/master/2023/202311241355190.png)

```js
var str = "hello world"
// 创建数组
var cars=new Array("Saab","Volvo","BMW");
```
JavaScript 只有一种数字类型。数字可以带小数点，也可以不带：

```js
var x1=34.00;      //使用小数点来写
var
x2=34;             //不使用小数点来写
```

- JavaScript 对象

对象由花括号分隔。在括号内部，对象的属性以名称和值对的形式 (name : value) 来定义。属性由逗号分隔：
```js
var person={firstname:"John", lastname:"Doe", id:5566};
```
上面例子中的对象 (person) 有三个属性：firstname、lastname 以及 id。
空格和折行无关紧要。声明可横跨多行：
```js
var person={  
	firstname : "John",  
	lastname  : "Doe",  
	id        :  5566  
};
```

- Undefined 和 Null

Undefined 这个值表示变量不含有值。可以通过将变量的值设置为 null 来清空变量。
```js
cars=null;  
person=null;
```

## 1.2 JavaScript 对象



## 1.3 JavaScript 函数

函数就是包裹在花括号中的代码块，前面使用关键字 function：
```js
function myFunction(_var1_,_var2_)  
{  
	_代码_  
}
```

如果把值赋给尚未声明的变量，该变量将被自动作为window的一个属性：
```js
var var1 = 1; // 不可配置全局属性
var2 = 2; // 没有使用 var 声明，可配置全局属性

console.log(this.var1); // 1
console.log(window.var1); // 1
console.log(window.var2); // 2

delete var1; // false 无法删除
console.log(var1); //1

delete var2; 
console.log(delete var2); // true
console.log(var2); // 已经删除 报错变量未定义
```


## 1.4 JavaScript 事件

|事件|描述|
|---|---|
|onchange|HTML 元素改变|
|onclick|用户点击 HTML 元素|
|onmouseover|鼠标指针移动到指定的元素上时发生|
|onmouseout|用户从一个 HTML 元素上移开鼠标时发生|
|onkeydown|用户按下键盘按键|
|onload|浏览器已完成页面的加载|

事件可以用于处理表单验证，用户输入，用户行为及浏览器动作:

- 页面加载时触发事件
- 页面关闭时触发事件
- 用户点击按钮执行动作
- 验证用户输入内容的合法性
- 等等 ...

可以使用多种方法来执行 JavaScript 事件代码：

- HTML 事件属性可以直接执行 JavaScript 代码
- HTML 事件属性可以调用 JavaScript 函数
- 你可以为 HTML 元素指定自己的事件处理程序
- 你可以阻止事件的发生。
- 等等 ...

```html
<!DOCTYPE html>
<html>
<head> 
<meta charset="utf-8"> 
<title>菜鸟教程(runoob.com)</title> 
</head>
<body>

<p>点击按钮执行 <em>displayDate()</em> 函数.</p>
<button onclick="displayDate()">点这里</button>
<script>
function displayDate(){
	document.getElementById("demo").innerHTML=Date();
}
</script>
<p id="demo"></p>

</body>
</html>
```


## 1.5 JavaScript 字符串

**字符串相关方法：**

|方法|描述|
|:--|:--|
|[charAt()](https://www.runoob.com/jsref/jsref-charat.html)|返回在指定位置的字符。|
|[charCodeAt()](https://www.runoob.com/jsref/jsref-charcodeat.html)|返回在指定的位置的字符的 Unicode 编码。|
|[concat()](https://www.runoob.com/jsref/jsref-concat-string.html)|连接两个或更多字符串，并返回新的字符串。|
|[endsWith()](https://www.runoob.com/jsref/jsref-endswith.html)|判断当前字符串是否是以指定的子字符串结尾的（区分大小写）。|
|[fromCharCode()](https://www.runoob.com/jsref/jsref-fromcharcode.html)|将 Unicode 编码转为字符。|
|[indexOf()](https://www.runoob.com/jsref/jsref-indexof.html)|返回某个指定的字符串值在字符串中首次出现的位置。|
|[includes()](https://www.runoob.com/jsref/jsref-string-includes.html)|查找字符串中是否包含指定的子字符串。|
|[lastIndexOf()](https://www.runoob.com/jsref/jsref-lastindexof.html)|从后向前搜索字符串，并从起始位置（0）开始计算返回字符串最后出现的位置。|
|[match()](https://www.runoob.com/jsref/jsref-match.html)|查找找到一个或多个正则表达式的匹配。|
|[repeat()](https://www.runoob.com/jsref/jsref-repeat.html)|复制字符串指定次数，并将它们连接在一起返回。|
|[replace()](https://www.runoob.com/jsref/jsref-replace.html)|在字符串中查找匹配的子串，并替换与正则表达式匹配的子串。|
|[replaceAll()](https://www.runoob.com/jsref/jsref-replaceall.html)|在字符串中查找匹配的子串，并替换与正则表达式匹配的所有子串。|
|[search()](https://www.runoob.com/jsref/jsref-search.html)|查找与正则表达式相匹配的值。|
|[slice()](https://www.runoob.com/jsref/jsref-slice-string.html)|提取字符串的片断，并在新的字符串中返回被提取的部分。|
|[split()](https://www.runoob.com/jsref/jsref-split.html)|把字符串分割为字符串数组。|
|[startsWith()](https://www.runoob.com/jsref/jsref-startswith.html)|查看字符串是否以指定的子字符串开头。|
|[substr()](https://www.runoob.com/jsref/jsref-substr.html)|从起始索引号提取字符串中指定数目的字符。|
|[substring()](https://www.runoob.com/jsref/jsref-substring.html)|提取字符串中两个指定的索引号之间的字符。|
|[toLowerCase()](https://www.runoob.com/jsref/jsref-tolowercase.html)|把字符串转换为小写。|
|[toUpperCase()](https://www.runoob.com/jsref/jsref-touppercase.html)|把字符串转换为大写。|
|[trim()](https://www.runoob.com/jsref/jsref-trim.html)|去除字符串两边的空白。|
|[toLocaleLowerCase()](https://www.runoob.com/jsref/jsref-tolocalelowercase.html)|根据本地主机的语言环境把字符串转换为小写。|
|[toLocaleUpperCase()](https://www.runoob.com/jsref/jsref-tolocaleuppercase.html)|根据本地主机的语言环境把字符串转换为大写。|
|[valueOf()](https://www.runoob.com/jsref/jsref-valueof-string.html)|返回某个字符串对象的原始值。|
|[toString()](https://www.runoob.com/jsref/jsref-tostring.html)|返回一个字符串。|

- **模板字符串**

```bash
模板字符串使用反引号 `` 作为字符串的定界符分隔的字面量。
模板字面量是用反引号（`）分隔的字面量，允许多行字符串、带嵌入表达式的字符串插值和一种叫带标签的模板的特殊结构。
```

```js
let header = "";  
let tags = ["RUNOOB", "GOOGLE", "TAOBAO"];  
  
let html = `<h2>${header}</h2><ul>`;  
for (const x of tags) {  
  html += `<li>${x}</li>`;  
}  
  
html += `</ul>`;
```


## 1.6 JavaScript 比较

### 1.6.1 比较运算符

|运算符|描述|比较|返回值|实例|
|:--|:--|:--|:--|:--|
|==|等于|x==8|_false_|[实例 »](https://www.runoob.com/try/try.php?filename=tryjs_comparison1)|
|x==5|_true_|[实例 »](https://www.runoob.com/try/try.php?filename=tryjs_comparison2)|
|===|绝对等于（值和类型均相等）|x==="5"|_false_|[实例 »](https://www.runoob.com/try/try.php?filename=tryjs_comparison3)|
|x===5|_true_|[实例 »](https://www.runoob.com/try/try.php?filename=tryjs_comparison4)|
|!=|不等于|x!=8|_true_|[实例 »](https://www.runoob.com/try/try.php?filename=tryjs_comparison5)|
|!==|不绝对等于（值和类型有一个不相等，或两个都不相等）|x!=="5"|_true_|[实例 »](https://www.runoob.com/try/try.php?filename=tryjs_comparison6)|
|!==|不绝对等于（值和类型有一个不相等，或两个都不相等）|x!==5|_false_|[实例 »](https://www.runoob.com/try/try.php?filename=tryjs_comparison7)|
|>|大于|x>8|_false_|[实例 »](https://www.runoob.com/try/try.php?filename=tryjs_comparison8)|
|<|小于|x<8|_true_|[实例 »](https://www.runoob.com/try/try.php?filename=tryjs_comparison9)|
|>=|大于或等于|x>=8|_false_|[实例 »](https://www.runoob.com/try/try.php?filename=tryjs_comparison10)|
|<=|小于或等于|x<=8|_true_|[实例 »](https://www.runoob.com/try/try.php?filename=tryjs_comparison11)|

### 1.6.2 逻辑运算符



|运算符|描述|例子|
|:--|:--|:--|
|&&|and|(x < 10 && y > 1) 为 true|
|\||or|(x==5 \| y==5) 为 false|
|!|not|!(x==y) 为 true|


### 1.6.3 条件运算符

JavaScript 还包含了基于某些条件对变量进行赋值的条件运算符。
_variablename_=(_condition_)?_value1_:_value2_


## 1.7 条件语句

```js
if (condition1)
{
    当条件 1 为 true 时执行的代码
}
else if (condition2)
{
    当条件 2 为 true 时执行的代码
}
else
{
  当条件 1 和 条件 2 都不为 true 时执行的代码
}
```

## 1.8 switch 语句

```js
switch(n)
{
    case 1:
        执行代码块 1
        break;
    case 2:
        执行代码块 2
        break;
    default:
        与 case 1 和 case 2 不同时执行的代码
}
```

工作原理：首先设置表达式 _n_（通常是一个变量）。随后表达式的值会与结构中的每个 case 的值做比较。如果存在匹配，则与该 case 关联的代码块会被执行。请使用 **break** 来阻止代码自动地向下一个 case 运行。

## 1.9 循环

- For循环

```js
for (var i=0;i<cars.length;i++)
{ 
    document.write(cars[i] + "<br>");
}
```

- For/In 循环

JavaScript for/in 语句循环遍历对象的属性：

```js
var person={fname:"Bill",lname:"Gates",age:56}; 
 
for (x in person)  // x 为属性名
{
    txt=txt + person[x];
}
```

- while 循环

js中支持 while循环和 do/while 循环
```js
while (i<5)
{
    x=x + "The number is " + i + "<br>";
    i++;
}
```

- break/continue 语句

 break 语句用于跳出循环。continue 用于跳过循环中的一个迭代。
 支持 带标签，如 `break labelname` 或 `continue labelname`

```js
cars=["BMW","Volvo","Saab","Ford"];
list: 
{
    document.write(cars[0] + "<br>"); 
    document.write(cars[1] + "<br>"); 
    document.write(cars[2] + "<br>"); 
    break list;
    document.write(cars[3] + "<br>"); 
    document.write(cars[4] + "<br>"); 
    document.write(cars[5] + "<br>"); 
}
```
输出：
```bash
BMW  
Volvo  
Saab
```

## 1.10 JavaScript typeof, null, 和 undefined

### 1.10.1 typeof 操作符

```js
typeof "John"                // 返回 string
typeof 3.14                  // 返回 number
typeof false                 // 返回 boolean
typeof [1,2,3,4]             // 返回 object
typeof {name:'John', age:34} // 返回 object
```

### 1.10.2 null 和 undefined

在 JavaScript 中 null 表示 "什么都没有"。null是一个只有一个值的特殊类型。表示一个空对象引用。可以设置为null 来清空对象。
也可以设置为 undefined 来清空对象，**undefined** 是一个没有设置值的变量。

```js
var person;  // 值为 undefined(空), 类型是undefined
```


- **undefined 和 null 的区别**

null 和 undefined 的值相等，但类型不等：
```js
typeof undefined             // undefined  
typeof null                  // object  
null === undefined           // false  
null == undefined            // true
```



# 2 异常处理

## 2.1 try 和 catch

**try** 语句测试代码块的错误。
**catch** 语句处理错误。
**throw** 语句创建自定义错误。
**finally** 语句在 try 和 catch 语句之后，无论是否有触发异常，该语句都会执行。

```js
try {
    ...    //异常的抛出
} catch(e) {
    ...    //异常的捕获与处理
} finally {
    ...    //结束处理
}
```


## 2.2 throw语句

throw 语句允许我们创建自定义错误。语法：`throw exception`

检测输入变量的值。如果值是错误的，会抛出一个异常（错误）。catch 会捕捉到这个错误，并显示一段自定义的错误消息：
```js
function myFunction() {
    var message, x;
    message = document.getElementById("message");
    message.innerHTML = "";
    x = document.getElementById("demo").value;
    try { 
        if(x == "")  throw "值为空";
        if(isNaN(x)) throw "不是数字";
        x = Number(x);
        if(x < 5)    throw "太小";
        if(x > 10)   throw "太大";
    }
    catch(err) {
        message.innerHTML = "错误: " + err;
    }
}
```
# 异步编程