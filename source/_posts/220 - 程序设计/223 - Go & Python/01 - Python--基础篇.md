---
title: Python编程语言--基础篇
date: 2023-10-16 16:27:41
tags:
  - 程序设计
  - Python
---

# 0 参考资料
![](https://raw.githubusercontent.com/BaihlUp/Figurebed/master/2023/20230919102400.png)

## 0.1 文档
[官方文档](https://docs.python.org/zh-cn/3.11/tutorial/index.html)


# 1 基础类型
## 1.1 数字
交互模式下，上次输出的表达式会赋给变量 \_。把 Python 当作计算器时，用该变量实现下一步计算更简单，例如：
![](https://raw.githubusercontent.com/BaihlUp/Figurebed/master/2023/202309191605430.png)


## 1.2 字符串
定义字符串支持：单引号 ('...') 或双引号 ("...") ，如果要输出原始字符串，在引号前添加 r
![](https://raw.githubusercontent.com/BaihlUp/Figurebed/master/2023/202309191608049.png)
原始字符串有一个限制：一个原始字符串不能以奇数个\字符结束。
 
字符串字面值可以包含多行。 一种实现方式是使用三重引号："""...""" 或 '''...'''。
![](https://raw.githubusercontent.com/BaihlUp/Figurebed/master/2023/202309191615475.png)

### 1.2.1 字符串运算符
![](https://raw.githubusercontent.com/BaihlUp/Figurebed/master/2023/202309201124113.png)

### 1.2.2 字符串相关方法
[字符串的方法](https://docs.python.org/zh-cn/3.11/library/stdtypes.html#string-methods)

## 1.3 列表
列表数据类型支持很多方法，列表对象的所有方法所示如下：

- list.append(_x_)

在列表末尾添加一个元素，相当于 `a[len(a):] = [x]` 。

- list.extend(_iterable_)

用可迭代对象的元素扩展列表。相当于 `a[len(a):] = iterable` 。

- list.insert(_i_, _x_)

在指定位置插入元素。第一个参数是插入元素的索引，因此，`a.insert(0, x)` 在列表开头插入元素， `a.insert(len(a), x)` 等同于 `a.append(x)` 。

- list.remove(_x_)

从列表中删除第一个值为 _x_ 的元素。未找到指定元素时，触发 [`ValueError`](https://docs.python.org/zh-cn/3.11/library/exceptions.html#ValueError "ValueError") 异常。

- list.pop([_i_])

删除列表中指定位置的元素，并返回被删除的元素。未指定位置时，`a.pop()` 删除并返回列表的最后一个元素。（方法签名中 _i_ 两边的方括号表示该参数是可选的，不是要求输入方括号。这种表示法常见于 Python 参考库）。

- list.clear()

删除列表里的所有元素，相当于 `del a[:]` 。

- list.index(_x_[, _start_[, _end_]])

返回列表中第一个值为 _x_ 的元素的零基索引。未找到指定元素时，触发 [`ValueError`](https://docs.python.org/zh-cn/3.11/library/exceptions.html#ValueError "ValueError") 异常。

可选参数 _start_ 和 _end_ 是切片符号，用于将搜索限制为列表的特定子序列。返回的索引是相对于整个序列的开始计算的，而不是 _start_ 参数。

- list.count(_x_)

返回列表中元素 _x_ 出现的次数。

- list.sort(_*_, _key=None_, _reverse=False_)

就地排序列表中的元素（要了解自定义排序参数，详见 [`sorted()`](https://docs.python.org/zh-cn/3.11/library/functions.html#sorted "sorted")）。

- list.reverse()

翻转列表中的元素。

- list.copy()

返回列表的浅拷贝。相当于 `a[:]` 。

**代码示例：**
![](https://raw.githubusercontent.com/BaihlUp/Figurebed/master/2023/202309201045646.png)

### 1.3.1 用列表实现堆栈

通过列表方法可以非常容易地将列表作为栈来使用，最后添加的元素将最先被提取（“后进先出”）。 要向栈顶添加一个条目，请使用 `append()`。 要从栈顶提取一个条目，请使用 `pop()`，无需显式指定索引。 例如:

```python

>>> stack = [3, 4, 5]
>>> stack.append(6)
>>> stack.append(7)
>>> stack
[3, 4, 5, 6, 7]
>>> stack.pop()
7
>>> stack
[3, 4, 5, 6]
>>> stack.pop()
6
>>> stack.pop()
5
>>> stack
[3, 4]
```

### 1.3.2 用列表实现队列
在列表末尾添加和删除元素非常快，但在列表开头插入或移除元素却很慢，实现队列 最好用  [`collections.deque`](https://docs.python.org/zh-cn/3.11/library/collections.html#collections.deque "collections.deque")，可以快速从两端添加或删除元素。例如：
```python
>>> from collections import deque
>>> queue = deque(["Eric", "John", "Michael"])
>>> queue.append("Terry")           # Terry arrives
>>> queue.append("Graham")          # Graham arrives
>>> queue.popleft()                 # The first to arrive now leaves
'Eric'
>>> queue.popleft()                 # The second to arrive now leaves
'John'
>>> queue                           # Remaining queue in order of arrival
deque(['Michael', 'Terry', 'Graham'])
```

### 1.3.3 列表推导式
列表推导式创建列表的方式更简洁。常见的用法为，对序列或可迭代对象中的每个元素应用某种操作，用生成的结果创建新的列表；或用满足特定条件的元素创建子序列。
例如，创建平方值的列表：
```python
>> squares = []
>>> for x in range(10):
...     squares.append(x**2)
...
>>> squares
[0, 1, 4, 9, 16, 25, 36, 49, 64, 81]
```
注意，这段代码创建（或覆盖）变量 `x`，该变量在循环结束后仍然存在。下述方法可以无副作用地计算平方列表：
```python
squares = list(map(lambda x: x**2, range(10)))
```
等价于：
```python
squares = [x**2 for x in range(10)]
```

列表推导式的方括号内包含以下内容：一个表达式，后面为一个 `for` 子句，然后，是零个或多个 `for` 或 `if` 子句。结果是由表达式依据 `for` 和 `if` 子句求值计算而得出一个新列表。 举例来说，以下列表推导式将两个列表中不相等的元素组合起来：
```python
>>> [(x, y) for x in [1,2,3] for y in [3,1,4] if x != y]
[(1, 3), (1, 4), (2, 3), (2, 1), (2, 4), (3, 1), (3, 4)]
```
等价于：
```python
>>> combs = []
>>> for x in [1,2,3]:
...     for y in [3,1,4]:
...         if x != y:
...             combs.append((x, y))
...
>>> combs
[(1, 3), (1, 4), (2, 3), (2, 1), (2, 4), (3, 1), (3, 4)]
```
### 1.3.4 嵌套的列表推导式
列表推导式中的初始表达式可以是任何表达式，甚至可以是另一个列表推导式。
下面这个 3x4 矩阵，由 3 个长度为 4 的列表组成：
```python
>>> matrix = [
...     [1, 2, 3, 4],
...     [5, 6, 7, 8],
...     [9, 10, 11, 12],
... ]
```
下面的列表推导式可以转置行列：
```python
>>> [[row[i] for row in matrix] for i in range(4)]
[[1, 5, 9], [2, 6, 10], [3, 7, 11], [4, 8, 12]]
```
等价于：
```python
>>> transposed = []
>>> for i in range(4):
...     transposed.append([row[i] for row in matrix])
...
>>> transposed
[[1, 5, 9], [2, 6, 10], [3, 7, 11], [4, 8, 12]]
```
也等价于：
```python
>>> transposed = []
>>> for i in range(4):
...     # the following 3 lines implement the nested listcomp
...     transposed_row = []
...     for row in matrix:
...         transposed_row.append(row[i])
...     transposed.append(transposed_row)
...
>>> transposed
[[1, 5, 9], [2, 6, 10], [3, 7, 11], [4, 8, 12]]
```

### 1.3.5 del 语句
有一种方式可以按索引而不是值从列表中移除条目: [`del`](https://docs.python.org/zh-cn/3.11/reference/simple_stmts.html#del) 语句。 这与返回一个值的 `pop()` 方法不同。 `del` 语句也可用于从列表中移除切片或清空整个列表（我们之前通过将切片赋值为一个空列表实现过此操作）。 例如:
```python
>>> a = [-1, 1, 66.25, 333, 333, 1234.5]
>>> del a[0]
>>> a
[1, 66.25, 333, 333, 1234.5]
>>> del a[2:4]
>>> a
[1, 66.25, 1234.5]
>>> del a[:]
>>> a
[]
```
del 也可以用来删除整个变量
```python
>>> del a
```

## 1.4 元组
元组由多个用逗号隔开的值组成，例如：
```python
>>> t = 12345, 54321, 'hello!'
>>> t[0]
12345
>>> t
(12345, 54321, 'hello!')
>>> # Tuples may be nested:
... u = t, (1, 2, 3, 4, 5)
>>> u
((12345, 54321, 'hello!'), (1, 2, 3, 4, 5))
>>> # Tuples are immutable:
... t[0] = 88888
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
TypeError: 'tuple' object does not support item assignment
>>> # but they can contain mutable objects:
... v = ([1, 2, 3], [3, 2, 1])
>>> v
([1, 2, 3], [3, 2, 1])
```

输出时，元组都要由圆括号标注，这样才能正确地解释嵌套元组。输入时，圆括号可有可无，不过经常是必须的（如果元组是更大的表达式的一部分）。不允许为元组中的单个元素赋值，当然，可以创建含列表等可变对象的元组。
虽然，元组与列表很像，但使用场景不同，用途也不同。元组是 [immutable](https://docs.python.org/zh-cn/3.11/glossary.html#term-immutable) （不可变的），一般可包含异质元素序列，通过解包（见本节下文）或索引访问（如果是 [`namedtuples`](https://docs.python.org/zh-cn/3.11/library/collections.html#collections.namedtuple "collections.namedtuple")，可以属性访问）。列表是 [mutable](https://docs.python.org/zh-cn/3.11/glossary.html#term-mutable) （可变的），列表元素一般为同质类型，可迭代访问。

构造 0 个或 1 个元素的元组比较特殊：为了适应这种情况，对句法有一些额外的改变。用一对空圆括号就可以创建空元组；只有一个元素的元组可以通过在这个元素后添加逗号来构建（圆括号里只有一个值的话不够明确）。丑陋，但是有效。例如：
```python
>>> empty = ()
>>> singleton = 'hello',    # <-- note trailing comma
>>> len(empty)
0
>>> len(singleton)
1
>>> singleton
('hello',)
```
语句 `t = 12345, 54321, 'hello!'` 是 _元组打包_ 的例子：值 `12345`, `54321` 和 `'hello!'` 一起被打包进元组。逆操作也可以：
```python
>>> x, y, z = t
```
称之为 _序列解包_ 也是妥妥的，适用于右侧的任何序列。序列解包时，左侧变量与右侧序列元素的数量应相等。注意，多重赋值其实只是元组打包和序列解包的组合。

**元组内置函数：**
![](https://raw.githubusercontent.com/BaihlUp/Figurebed/master/2023/202309201114068.png)

## 1.5 字典
可以把字典理解为 _键值对_ 的集合，但字典的键必须是唯一的。花括号 `{}` 用于创建空字典。另一种初始化字典的方式是，在花括号里输入逗号分隔的键值对，这也是字典的输出方式。
字典的主要用途是通过关键字存储、提取值。用 `del` 可以删除键值对。用已存在的关键字存储值，与该关键字关联的旧值会被取代。通过不存在的键提取值，则会报错。

对字典执行 `list(d)` 操作，返回该字典中所有键的列表，按插入次序排列（如需排序，请使用 `sorted(d)`）。检查字典里是否存在某个键，使用关键字 [`in`](https://docs.python.org/zh-cn/3.11/reference/expressions.html#in)。

**代码示例：**
```python
>>> tel = {'jack': 4098, 'sape': 4139}
>>> tel['guido'] = 4127
>>> tel
{'jack': 4098, 'sape': 4139, 'guido': 4127}
>>> tel['jack']
4098
>>> del tel['sape']
>>> tel['irv'] = 4127
>>> tel
{'jack': 4098, 'guido': 4127, 'irv': 4127}
>>> list(tel)
['jack', 'guido', 'irv']
>>> sorted(tel)
['guido', 'irv', 'jack']
>>> 'guido' in tel
True
>>> 'jack' not in tel
False
```

[`dict()`](https://docs.python.org/zh-cn/3.11/library/stdtypes.html#dict "dict") 构造函数可以直接用键值对序列创建字典：
```python
>>> dict([('sape', 4139), ('guido', 4127), ('jack', 4098)])
{'sape': 4139, 'guido': 4127, 'jack': 4098}
```

字典推导式可以用任意键值表达式创建字典：
```python
>>> {x: x**2 for x in (2, 4, 6)}
{2: 4, 4: 16, 6: 36}
```

关键字是比较简单的字符串时，直接用关键字参数指定键值对更便捷：
```python
>>> dict(sape=4139, guido=4127, jack=4098)
{'sape': 4139, 'guido': 4127, 'jack': 4098}
```

**字典包含的内置函数：**
![](https://raw.githubusercontent.com/BaihlUp/Figurebed/master/2023/202309201112767.png)

**字典包含的内置方法如下：**
![](https://raw.githubusercontent.com/BaihlUp/Figurebed/master/2023/202309201111695.png)


## 1.6 集合
集合是由不重复元素组成的无序容器。基本用法包括成员检测、消除重复元素。集合对象支持合集、交集、差集、对称差分等数学运算。
创建集合用花括号或 [`set()`](https://docs.python.org/zh-cn/3.11/library/stdtypes.html#set "set") 函数。注意，创建空集合只能用 `set()`，不能用 `{}`，`{}` 创建的是空字典。

```python
>>> basket = {'apple', 'orange', 'apple', 'pear', 'orange', 'banana'}
>>> print(basket)                      # show that duplicates have been removed
{'orange', 'banana', 'pear', 'apple'}
>>> 'orange' in basket                 # fast membership testing
True
>>> 'crabgrass' in basket
False

>>> # Demonstrate set operations on unique letters from two words
...
>>> a = set('abracadabra')
>>> b = set('alacazam')
>>> a                                  # unique letters in a
{'a', 'r', 'b', 'c', 'd'}
>>> a - b                              # letters in a but not in b
{'r', 'd', 'b'}
>>> a | b                              # letters in a or b or both
{'a', 'c', 'r', 'd', 'b', 'm', 'z', 'l'}
>>> a & b                              # letters in both a and b
{'a', 'c'}
>>> a ^ b                              # letters in a or b but not both
{'r', 'd', 'b', 'm', 'z', 'l'}
```
集合也支持列表推导式：
```python
>>> a = {x for x in 'abracadabra' if x not in 'abc'}
>>> a
{'r', 'd'}
```

## 1.7 扩展数据类型
### 1.7.1 命名元组
`namedtuple` 是 Python 标准库中的一个数据结构，它用于创建具有字段名的不可变元组。使用前需要导入 `from collections import namedtuple`

```python
from collections import namedtuple
Point = namedtuple('Point', ['x', 'y'])
p1 = Point(11, y=22)

print(p1.x)  # 1
print(p1[1]) # 2
#p1.x = 10 # 报错
print(p1._fields) # 获取字段名的列表  
print(p1._asdict()) # 命名元组转换为字典
```
1. 命名元组是不可变的，这意味着一旦创建，你无法修改它的字段值。如果你尝试这样做，会引发 `AttributeError`。
2. 命名元组还具有一些有用的方法和属性，例如 `._fields` 属性用于获取字段名的列表，以及 `_asdict()` 方法用于将命名元组转换为字典。
3. 命名元组还支持元组的常见操作，例如索引、切片和迭代。


### 1.7.2 双端队列
deque 对象是实现双端队列，比传统的列表多了 `appendleft()` 、`popleft()` 方法
**示例：**
```python
from collections import deque

# 创建双端队列
my_deque = deque()

# 插入元素
my_deque.append(1)       # 将元素1添加到右端
my_deque.appendleft(2)   # 将元素2添加到左端

# 删除元素
right_element = my_deque.pop()       # 删除并返回右端的元素
left_element = my_deque.popleft()   # 删除并返回左端的元素

# 访问元素
element = my_deque[0]  # 访问第一个元素

# 长度和判空
length = len(my_deque)    # 获取双端队列的长度
is_empty = not my_deque  # 判断双端队列是否为空

# 迭代
for element in my_deque:
    print(element)

# 扩展双端队列
my_deque.extend([3, 4, 5])          # 将多个元素添加到右端
my_deque.extendleft([0, -1, -2])    # 将多个元素添加到左端

# 反转双端队列
my_deque.reverse()

# 旋转双端队列
my_deque.rotate(2)  # 向右旋转2步，即右端的元素移到左端

# 转换为列表
my_list = list(my_deque)

# 清空双端队列
my_deque.clear()
```


### 1.7.3 计数器
`Counter` 对象是 Python 中的一个非常有用的工具，它用于统计可迭代对象中元素的数量，通常用于处理计数问题。

```python
from collections import Counter  
  
# 创建 Counter 对象  
my_list = [1, 2, 2, 3, 3, 3, 4, 4, 4, 4]  
my_counter = Counter(my_list)  
  
# 获取元素的计数  
print(my_counter[2]) # 输出: 2  
print(my_counter[5]) # 输出: 0  
  
# 元素计数的方法  
  
# 使用 elements() 方法获取每个元素的重复次数  
elements = my_counter.elements()  
for element in elements:  
print(element)  
  
# 使用 most_common() 方法获取计数最高的元素和其计数，可以指定返回的元素数量  
most_common_elements = my_counter.most_common(2)  
print(most_common_elements) # 输出: [(4, 4), (3, 3)]  
  
# 更新计数  
other_counter = Counter([2, 3, 3, 3, 5])  
my_counter.update(other_counter)  
print(my_counter) # Counter({3: 6, 4: 4, 2: 3, 1: 1, 5: 1})  
  
# 删除元素  
del my_counter[4]  
print(my_counter) # Counter({3: 6, 2: 3, 1: 1, 5: 1})  
  
# 计数的加法和减法  
counter1 = Counter([1, 2, 3, 4, 5])  
counter2 = Counter([3, 4, 5, 6, 7])  
  
result_add = counter1 + counter2  
print(result_add) # Counter({3: 2, 4: 2, 5: 2, 1: 1, 2: 1, 6: 1, 7: 1})  
  
result_subtract = counter1 - counter2  
print(result_subtract) # Counter({1: 1, 2: 1})  
  
# 清空 Countermy_counter.clear()  
print(my_counter) # Counter()  
  
# 更多用法  
  
# 统计字符串中字符的出现次数  
my_string = "hello, world!"  
char_count = Counter(my_string)  
print(char_count) # Counter({'l': 3, 'o': 2, 'h': 1, 'e': 1, ',': 1, ' ': 1, 'w': 1, 'r': 1, 'd': 1, '!': 1})  
  
# 统计单词的出现次数  
my_text = "This is a sample text. This text is just a sample."  
word_count = Counter(my_text.split())  
print(word_count) # Counter({'This': 2, 'is': 2, 'a': 2, 'sample': 1, 'text.': 1, 'text': 1, 'just': 1, 'sample.': 1})  
  
# 合并多个 Counter 对象  
counter1 = Counter([1, 2, 2, 3, 3])  
counter2 = Counter([2, 3, 3, 4, 4])  
merged_counter = counter1 + counter2  
print(merged_counter) # Counter({3: 4, 2: 3, 4: 2, 1: 1})
```
### 1.7.4 字典和列表子类化
- UserDict 类用于字典对象的二次开发
- UserList 类用于列表对象的二次开发
- 当普通字典、列表无法满足需求时，可以通过继承 UserDict 和 UserList 实现增强功能的字典和列表

**增强字典的功能：**

```python
from collections import UserDict

class MyDict(UserDict):
    def __setitem__(self, key, value):
        if key in self.data.keys():
            print(f"字典里已经有key {key} ,不能覆盖")
        else:
            self.data[str(key)] = value
 
dict1 = MyDict()
dict1['a'] = 1
dict1['a'] = 2   # 字典里已经有key a ,不能覆盖
```
以上程序通过重新实现 字典的 `__setitem__` 方法，实现了在赋值时的检查，如果key已存在，则不覆盖。

更多容器数据类型可以查看文档：[https://docs.python.org/zh-cn/3.11/library/collections.html](https://docs.python.org/zh-cn/3.11/library/collections.html)

### 1.7.5 魔术方法
每种类型都有魔术方法，可以重写魔术方法实现一些自己定制的功能。
查看类型的魔术方法 `dir(类型/实例)`

- **如下查看 类的 魔术方法**

```python
class MyClass:
    pass

myclass = MyClass()
dir(myclass)
```
输出：
```shell
['__class__',
 '__delattr__',
 '__dict__',
 '__dir__',
 '__doc__',
 '__eq__',
 '__format__',
 '__ge__',
 '__getattribute__',
 '__getstate__',
 '__gt__',
 '__hash__',
 '__init__',
 '__init_subclass__',
 '__le__',
 '__lt__',
 ...
 ]
```


# 2  判断和循环
## 2.1 if 语句

```python
li = 10
if li == 1:
    print("1")
elif li == 10:
    print("10")
else :
    print("else")
```

## 2.2 while 循环

```python
number = 1
while number <= 3:
    print(f"number is {number}")
    # number_temp = number + 1
    # number = number_temp
    number += 1
print(number)
```

- 支持break、continue

```python
number = 10
while number >= 1:
    number -= 1
    if number == 5:
        # break
        continue
    print(f"number is {number}")
```

比较运算符 `in` 和 `not in` 用于执行确定一个值是否存在（或不存在）于某个容器中的成员检测。 运算符 `is` 和 `is not` 用于比较两个对象是否是同一个对象。 所有比较运算符的优先级都一样，且低于任何数值运算符。

比较操作支持链式操作。例如，`a < b == c` 校验 `a` 是否小于 `b`，且 `b` 是否等于 `c`。

比较操作可以用布尔运算符 `and` 和 `or` 组合，并且，比较操作（或其他布尔运算）的结果都可以用 `not` 取反。这些操作符的优先级低于比较操作符；`not` 的优先级最高， `or` 的优先级最低，因此，`A and not B or C` 等价于 `(A and (not B)) or C`。与其他运算符操作一样，此处也可以用圆括号表示想要的组合。

布尔运算符 `and` 和 `or` 是所谓的 _短路_ 运算符：其参数从左至右求值，一旦可以确定结果，求值就会停止。例如，如果 `A` 和 `C` 为真，`B` 为假，那么 `A and B and C` 不会对 `C` 求值。用作普通值而不是布尔值时，短路运算符的返回值通常是最后一个求了值的参数。

还可以把比较运算或其它布尔表达式的结果赋值给变量，例如：
```python
>>> string1, string2, string3 = '', 'Trondheim', 'Hammer Dance'
>>> non_null = string1 or string2 or string3
>>> non_null
'Trondheim'
```

## 2.3 for 循环

```python
movie1 = { "name":"Friends", "language":"En", "Sessions":10, "Other name":"Six of One" }

for title in movie1.keys():
    print(title)
```
输出：
```
name
language
Sessions
Other name
```

```python
for i in enumerate(movie1.items()):
    print(i)
```
输出：
```shell
(0, ('name', 'Friends'))
(1, ('language', 'En'))
(2, ('Sessions', 10))
(3, ('Other name', 'Six of One'))
```
enumerate() 函数用于将一个可遍历的数据对象(如列表、元组或字符串)组合为一个索引序列。一般用在for循环中。

按指定顺序循环序列，可以用 [`sorted()`](https://docs.python.org/zh-cn/3.11/library/functions.html#sorted "sorted") 函数，在不改动原序列的基础上，返回一个重新的序列：
```python
>>> basket = ['apple', 'orange', 'apple', 'pear', 'orange', 'banana']
>>> for i in sorted(basket):
...     print(i)
...
apple
apple
banana
orange
orange
pear
```
使用 [`set()`](https://docs.python.org/zh-cn/3.11/library/stdtypes.html#set "set") 去除序列中的重复元素。使用 [`sorted()`](https://docs.python.org/zh-cn/3.11/library/functions.html#sorted "sorted") 加 [`set()`](https://docs.python.org/zh-cn/3.11/library/stdtypes.html#set "set") 则按排序后的顺序，循环遍历序列中的唯一元素：
```python
>>> basket = ['apple', 'orange', 'apple', 'pear', 'orange', 'banana']
>>> for f in sorted(set(basket)):
...     print(f)
...
apple
banana
orange
pear
```

## 2.4 range() 函数
内置 range() 函数用于生成等差数列
生成的序列绝不会包括给定的终止值；`range(10)` 生成 10 个值——长度为 10 的序列的所有合法索引。range 可以不从 0 开始，且可以按给定的步长递增（即使是负数步长）：
```python
>>> list(range(5, 10))
[5, 6, 7, 8, 9]
>>> list(range(0, 10, 3))
[0, 3, 6, 9]
>>> list(range(-10, -100, -30))
[-10, -40, -70]
```
要按索引迭代序列，可以组合使用 [`range()`](https://docs.python.org/zh-cn/3.11/library/stdtypes.html#range "range") 和 [`len()`](https://docs.python.org/zh-cn/3.11/library/functions.html#len "len")：
```python
>>> a = ['Mary', 'had', 'a', 'little', 'lamb']
>>> for i in range(len(a)):
>>>	print(i, a[i])
0 Mary
1 had
2 a
3 little
4 lamb
```

## 2.5 pass 语句
pass 语句不执行任何动作。

## 2.6 match 语句

```python
http_response_status = 500
match http_response_status:
        case 400:
            print("Bad request" )
        case 404:
            print( "Not found")
        case 418:
            print( "I'm a teapot")
        case _:
            print( "Something's wrong with the internet")
```
输出：
```
Something's wrong with the internet
```
`_`  表示匹配所有

- match语句在类中的使用

```python

class Point:
    def __init__(self, x, y):
        self.x = x
        self.y = y


def where_is(point):
    match point:
        case Point(x=0, y=0):
            print("Origin")
        case Point(x=0, y=y):
            print(f"Y={y}")
        case Point(x=x, y=0):
            print(f"X={x}")
        case Point():
            print("Somewhere else")
        case _:
            print("Not a point")


point = Point(0, 1)
where_is(point)
```
输出：
```shell
Y=1
```

## 2.7 Python列表推导式

```python
list3 = [ i*i for i in (1, 2, 3, 4) if i > 2]
print(list3)
```
输出：
```shell
[9, 16]
```
以上的程序相当于下边的写法：
```python
for i in (1, 2, 3, 4):
    if i > 2:
        list3.append( i*i )
print(list3)
```

## 2.8 其他
- **越界错误**

```python
list1 = [ 'a', 'b', 'c', 'd' ]
for i in range(0, len(list1)):
    list1.pop(i)
print(list1)
```
执行后报错：
```shell
IndexError                                Traceback (most recent call last)
Cell In [10], line 3
      1 list1 = [ 'a', 'b', 'c', 'd' ]
      2 for i in range(0, 3):
----> 3     list1.pop(i)
      5 print(list1)

IndexError: pop index out of range
```

- **序列和其他类型的比较**
序列对象可以与相同序列类型的其他对象比较。这种比较使用 _字典式_ 顺序：首先，比较前两个对应元素，如果不相等，则可确定比较结果；如果相等，则比较之后的两个元素，以此类推，直到其中一个序列结束。如果要比较的两个元素本身是相同类型的序列，则递归地执行字典式顺序比较。如果两个序列中所有的对应元素都相等，则两个序列相等。如果一个序列是另一个的初始子序列，则较短的序列可被视为较小（较少）的序列。 对于字符串来说，字典式顺序使用 Unicode 码位序号排序单个字符。下面列出了一些比较相同类型序列的例子：
```python
(1, 2, 3)              < (1, 2, 4)
[1, 2, 3]              < [1, 2, 4]
'ABC' < 'C' < 'Pascal' < 'Python'
(1, 2, 3, 4)           < (1, 2, 4)
(1, 2)                 < (1, 2, -1)
(1, 2, 3)             == (1.0, 2.0, 3.0)
(1, 2, ('aa', 'ab'))   < (1, 2, ('abc', 'a'), 4)
```
> 注意，当比较不同类型的对象时，只要待比较的对象提供了合适的比较方法，就可以使用 `<` 和 `>` 进行比较。例如，混合的数字类型通过数字值进行比较，所以，0 等于 0.0，等等。如果没有提供合适的比较方法，解释器不会随便给出一个比较结果，而是引发 [`TypeError`](https://docs.python.org/zh-cn/3.11/library/exceptions.html#TypeError "TypeError") 异常。


# 3 输入和输出
## 3.1 输入

使用 input() 函数可以直接从终端接收输入，如果是命令行参数推荐如下两个库：
1. argparse -- 用户友好的命令行选项解释器，[参考文档](https://docs.python.org/zh-cn/3.10/library/argparse.html)
程序示例：
```python
import argparse

# 执行方式  python3 6-1arg-1.py -number 100
parser = argparse.ArgumentParser(description="这个程序用来演示参数处理")
# -number 表示 number 参数是可选参数
parser.add_argument( "-number", help="输入一个数字")
# number 表示必选参数，如果执行时不输入参数，则会报错
# parser.add_argument( "number", help="输入一个数字")
# parser.add_argument("-number", type=int, default=10) 强制转换参数类型和设置默认值
args = parser.parse_args()
print(f"你输入的number参数是 {args.number} ")
```
因为是可选参数，当执行时未输出参数，执行结果如下：
```shell
你输入的number参数是 None
```

示例：
```python
import argparse
# 执行方式  python3 6-1arg-1.py -number 100
parser = argparse.ArgumentParser(description="这个程序用来演示参数处理")
# -number 表示 number 参数是可选参数
parser.add_argument( "-number1", type=int,  default=10, help="输入一个数字")
parser.add_argument( "-number2", type=int,  default=20, help="输入一个数字")
args = parser.parse_args()
print(f"你输入的两数之和是 {args.number1+args.number2} ")
```

1. getopt -- C风格命令行选项解析器：[参考文档](https://docs.python.org/zh-cn/3.11/library/getopt.html)

## 3.2 输出
1. 百分号 %
```python
""%s is %s than %s" %("Beautiful", "better", "ugly")
```

%s 格式化字符串
%d 格式化整数
%f 浮点数

```python
>>> "%-5.3d" %(123.456)
'123  ' # 3后面有两个占位符
```

1. format() 函数

```python
print('{0} and {1}'.format('spam', 'eggs'))
print('{1} and {0}'.format('spam', 'eggs'))
```
花括号中的数字表示传递给 [`str.format()`](https://docs.python.org/zh-cn/3.11/library/stdtypes.html#str.format "str.format") 方法的对象所在的位置。

### 3.1.1 F-strings
F-strings 是Python 3.6 新增的字符串格式化功能

- F-strings 的{}中可以实现数字计算、字符串连接、函数执行等计算任务
```python
f"{ 1+2 }"
f"{ 'a'+'b' }"
f"{ print('c') }
```

- 嵌入的内容可解释执行
- F-strings宽度和精度：`f"{对象:宽度.精度类型}"`

```python
number = 123.456  
print(f"{number:10}") #   123.456 前面有三个占位符
print(f"{number:010}")  # 000123.456
print(f"{number:4f}") # 123.456000 指定类型后，默认保存小数点后6位
print(f"{number:.2f}") # 123.46 保存两位小数
```

**示例：**
```python
year = 2016
event = 'Referendum'
f'Results of the {year} {event}'
```
还有一些修饰符可以在格式化前转换值。 `'!a'` 应用 [`ascii()`](https://docs.python.org/zh-cn/3.11/library/functions.html#ascii "ascii") ，`'!s'` 应用 [`str()`](https://docs.python.org/zh-cn/3.11/library/stdtypes.html#str "str")，`'!r'` 应用 [`repr()`](https://docs.python.org/zh-cn/3.11/library/functions.html#repr "repr")：
```python
animals = 'eels'
print(f'My hovercraft is full of {animals}.')
print(f'My hovercraft is full of {animals!r}.')
```

# 4 文件操作
## 4.1 打开文件

```python
open(file, mode='r', buffering=-1, encoding=None, errors=None, newline=None, closefd=True, opener=None)
```

`file_handler = open("/tmp/afile", mode="r")` 常见的 mode 参数值如下：
![](https://raw.githubusercontent.com/BaihlUp/Figurebed/master/2023/202309201019205.png)

## 4.2 文件编码

涉及文件编码的要指定 encoding，默认是 UTF-8
```python
f = open('demo_GBK.txt', mode='r', encoding='GBK')  
data = f.readlines()  
print(data)  
f.close()
```

## 4.3 文件读取
读取文件有如下几个函数：
![](https://raw.githubusercontent.com/BaihlUp/Figurebed/master/2023/202309201023039.png)

## 4.4 文件写入
不同的 open() 函数打开模式，执行写入时，写入的位置和结果不同：
![](https://raw.githubusercontent.com/BaihlUp/Figurebed/master/2023/202309201026742.png)

配合不同函数可以实现指定位置的文件写入：
- tell() 函数 -- 返回文件指针位置
- seek(偏移量，[起始位置]) -- 移动指针
- close() -- 关闭文件

## 4.5 文件关闭
文件关闭除了使用 close() 函数，也可以使用 with 语句进行简化。使用 with 语句打开的文件，在离开 with 语句块作用域会自动关闭。
```python
files_name = ["demo1.txt", "demo2.txt", "demo3.txt"]  
files_data = []  
  
for f_name in files_name:  
	with open(f_name) as f:  
		files_data.append(f.read())  
  
with open("demo4.txt", mode="w") as f:  
	for data in files_data:  
		f.write(data)
```
以上实现 三个文件的内容合并。

# 5 函数
## 5.0 函数定义
```python
def foo():  
	print("foo")  
	print("is a function")  
  
print("aaaaa")
```

- **匿名函数**

```python
add_1 = lambda x:x+1
print(add_1(100))
```
相当于：
```python
def add_1(x):
	return x+1
print(add_1(100))
```

## 5.1 函数参数
函数定义中可能包含多个形参，因此函数调用中也可能包含多个实参。向函数传递实参的方式很多：可使用位置实参 ，这要求实参的顺序与形参的顺序相同；也可使用关键字实参 ，其中每个实参都由变量名和值组成；还可使用列表和字典。下面依次介绍这些方式。

### 5.1.1 位置实参
调用函数时，Python必须将函数调用中的每个实参都关联到函数定义中的一个形参。为此，最简单的关联方式是基于实参的顺序。这种关联方式称为位置实参 。
```python
def describe_pet(animal_type, pet_name):
      """显示宠物的信息。"""
      print(f"\nI have a {animal_type}.")
      print(f"My {animal_type}'s name is {pet_name.title()}.")

describe_pet('hamster', 'harry')

```
输出：
```shell
I have a hamster. 
My hamster's name is Harry.
```
> 位置实参的顺序很重要

### 5.1.2 关键字实参
关键字实参 是传递给函数的名称值对。因为直接在实参中将名称和值关联起来，所以向函数传递实参时不会混淆。
```python
def describe_pet(animal_type, pet_name):
    """显示宠物的信息。"""
    print(f"\nI have a {animal_type}.")
    print(f"My {animal_type}'s name is {pet_name.title()}.")

describe_pet(animal_type='hamster', pet_name='harry')
```
函数describe_pet() 还和之前一样，但调用这个函数时，向Python明确地指出了各个实参对应的形参。看到这个函数调用时，Python知道应该将实参'hamster' 和'harry' 分别赋给形参animal_type 和pet_name 。输出正确无误，指出有一只名为Harry的仓鼠。

关键字实参的顺序无关紧要，因为Python知道各个值该赋给哪个形参。下面两个函数调用是等效的：
```python
describe_pet(animal_type='hamster', pet_name='harry')
describe_pet(pet_name='harry', animal_type='hamster')
```

> 位置实参需要位于关键字参数前边

### 5.1.3 等效的函数调用
鉴于可混合使用位置实参、关键字实参和默认值，通常有多种等效的函数调用方式。请看下面对函数describe_pet() 的定义，其中给一个形参提供了默认值：
```python
def describe_pet(pet_name, animal_type='dog'):
```
基于这种定义，在任何情况下都必须给pet_name 提供实参。指定该实参时可采用位置方式，也可采用关键字方式。如果要描述的动物不是小狗，还必须在函数调用中给animal_type 提供实参。同样，指定该实参时可以采用位置方式，也可采用关键字方式。
下面对这个函数的所有调用都可行：
```python
# 一条名为Willie的小狗。
describe_pet('willie')
describe_pet(pet_name='willie')

# 一只名为Harry的仓鼠。
describe_pet('harry', 'hamster')
describe_pet(pet_name='harry', animal_type='hamster')
describe_pet(animal_type='hamster', pet_name='harry')
```
这些函数调用的输出与前面的示例相同。

### 5.1.4 不定长参数

```python
def address_book(name, *telphone, alias_name=None, **custom):  
	print(f"name: {name}, tel: {telphone}, aname: {alias_name}, custom:{custom}")
```
```shell
* 接收位置参数
* * 接收关键字参数
```

位置形参需要在关键子形参前边。
```python
address_book("wilson")
```
输出：
```shell
name: wilson, tel: (), aname: None, custom:{}
```

```python
address_book("wilson",1234,4567,5678)
```
输出：
```shell
name: wilson, tel: (1234, 4567, 5678), aname: None, custom:{}
```

```python
address_book("wilson",1234,4567,5678,home="Guangdong")
```
输出：
```shell
name: wilson, tel: (1234, 4567, 5678), aname: None, custom:{'home': 'Guangdong'}
```

```python
address_book("wilson",1234,4567,5678,alias_name='w',aaa="bbb", home="Guangdong")
```
输出：
```shell
name: wilson, tel: (1234, 4567, 5678), aname: w, custom:{'aaa': 'bbb', 'home': 'Guangdong'}
```

### 5.1.5 函数参数传递列表
如果函数参数传递的是列表，如果在函数中修改列表，传递的实参也会相应变化。
为了禁止函数修改列表，可以使用如下方式，传递列表的副本而非原件。
```python
function_name(list_name_[:])
```

## 5.2 函数文档
```python
def foo():
	"""这个函数的用途、用法、注意事项等"""
	pass
```

查看文档的方法：`foo.__doc__`
![](https://raw.githubusercontent.com/BaihlUp/Figurebed/master/2023/202309201617053.png)

1. 函数内部有很多内置的属性，这些属性是在将一个函数定义为函数对象后，自动产生的
2. 使用dir() 函数查看其内部定义的属性和方法

## 5.3 返回值
函数可返回任何类型的值，包括列表和字典等较复杂的数据结构。
- return 变量
- return 变量[， 变量，变量]
- return 字面值
- return 函数调用
- return ...


## 5.4 高阶函数
### 5.4.1 函数对象和函数调用
```python
def foo():
	print("foo 函数被调用")
```
函数调用：`foo()`
函数对象：
```python
var = foo
var()
```

### 5.4.2 map函数
```python
map(函数，可迭代对象)
```
- 函数：函数对象、lambda表达式
- 可迭代对象：列表、元组、range等
- 将可迭代对象的每个元素作为函数参数执行

```python
def add(number):  
	return ( number + number )

# range(5) 0 1 2 3 4  
# map(add, range(5) 0 2 4 6 7  
for i in map(add, range(5) ):  
	print( i )
```
输出：
```python
0
2
4
6
8
```

以上程序可以简写成 如下：
```python
print( list( map( lambda x: x+x, range(5) ) ) )
```
输出：
```shell
[0, 2, 4, 6, 8]
```

### 5.4.3 filter函数
```python
filter(函数，可迭代对象)
```
- 过滤掉可迭代对象中返回值不为True的元素
- filter函数返回一个可迭代对象

```python
for i in filter(lambda x:x>0, (-2, -1, 1, 2)):  
	print(i)
```
输出：
```shell
1
2
```


### 5.4.4 reduce函数
```python
reduce(函数，可迭代对象)
```
- 对可迭代对象进行累积

```python
from functools import reduce  
reduce(lambda x, y: x+y, [1, 2, 3, 4, 5])
```
输出：
```shell
15
```

### 5.4.5 闭包
函数的返回值也可以是函数对象（闭包）
```python
def func_closure():
    def get_message(message):
        print('Got a message: {}'.format(message))
    return get_message
 
send_message = func_closure()
send_message('hello world')
 
# 输出
Got a message: hello world
```




# 7 模块与标准库
## 7.1 模块使用
模块是存放函数的且以 `.py` 结尾的文件。
模块的导入方式：
1. 导入整个模块：`import os`
2. 导入模块的特定方法：`from os import chdir` 或 `from os import chdir, getcwd`
3. 多个模块放在一个文件夹中，改文件夹称作包
4. 包的导入与模块的导入相同： `import 包` 或 `from 包 import 模块`
5. 为导入的模块指定别名：`import numpy as np`
6. 不建议使用 `*` 导入模块的所有函数：`from os import *`

**模块导入和使用：**
```python
import os 
os.getcwd()

from os import getcwd
getcwd()
```


- **在导入模块时，模块就会执行**

test.py
```python 
var1 = 100  
  
def func1():  
	print("func1 is called")  
  
  
class Class1(object):  
	def __init__(self):  
		print("Class1 is instance")  
  
print(__name__)
```
```python
import test.py
```
输出：
```shell
test
```
被别的模块导入时执行的，`__name__` 是模块名字，如果是直接执行的 `__name__ == __main__` 
```python
var1 = 100  
  
def func1():  
	print("func1 is called")  
  
class Class1(object):  
	def __init__(self):  
		print("Class1 is instance")  
  
if __name__ == "__main__":  
	func1()
```
通过以上方式，则在导入模块时不执行 `func1`，如果是直接执行，则会执行`func1`


## 7.2 标准库
标准库的常见组件：
- 内置函数、类型、异常
- 文本处理
- 数字
- 文件和目录
- 通用操作系统
- 并发执行
- 网络和进程间通信
- 互联网协议

参考文档：[https://docs.python.org/zh-cn/3.11/library/index.html](https://docs.python.org/zh-cn/3.11/library/index.html)

## 7.3 安装和使用第三方模块
安装第三方模块使用 pip 命令：`pip3.10 install 第三方模块名称` 或 `python3.10 -m pip install 第三方模块名称`

默认安装的第三方模块可以供所有的项目使用，但我们自己开发的项目可能只依赖部分模块，或者两个不同的项目依赖同一个模块的不同版本。这时候可以使用虚拟环境。

创建虚拟环境：`python -m venv myvenv`
- venv 虚拟环境模块
- myvenv 保存虚拟环境的文件夹

将当前安装的包及其版本保存到文件中：`pip3.10 freeze > requirements.txt`
虚拟环境文件夹里包含了所需的工具，进入虚拟环境：`source myvenv/bin/activate`
在虚拟环境中导入指定的包：`pip3.10 install -r requirements.txt`
离开虚拟环境：`deactivate`

- **加速第三方模块安装**

临时加速：`pip install -i https://pypi.tuna.tsinghua.edu.cn/simple package_name`
永久加速：
```bash
cat ~/pip.conf

[global]
index-url = http://mirrors.aliyun.com/pypi/simple
[install]
trusted-host=mirrors.aliyun.com
```

- **pip命令使用**

```shell
pip freeze > requirements.txt # 导出当前环境的包
pip install -r requriements.txt # 导入指定的包
```

- **pipreqs工具**

这个工具的好处是可以通过对项目目录的扫描，自动发现使用了那些类库，自动生成依赖清单。

```bash
pipreqs . --encoding=utf8 --force

```
1. “.” 指的是将导出依赖包的文件放在当前目录下
2. “--encoding=utf8” 指的是存放文件的编码为utf-8,否则会报错
3. “--force” --force 强制执行，当 生成目录下的requirements.txt存在时强子覆盖

**pipreqs常见参数：**

```bash
选项：  
--use-local仅使用本地包信息而不是查询PyPI  
--pypi-server <url>使用自定义PyPi服务器  
--proxy <url>使用Proxy，参数将传递给请求库。你也可以设置  
--debug打印调试信息  
--ignore <dirs> ...忽略额外的目录  
--encoding <charset>使用编码参数打开文件  
--savepath <file>保存给定文件中的需求列表  
--print输出标准输出中的需求列表  
--force覆盖现有的requirements.txt  
--diff <file>将requirements.txt中的模块与项目导入进行比较。  
--clean <file>通过删除未在项目中导入的模块来清理requirements.txt。
```


# 9 异常
## 9.1 异常分类
- **异常基类**

BaseException：所有内置异常的基类
Exception：所有内置的非系统推出类异常都派生自此类，所有用户自定义异常也应当派生自此类。

异常分类：[https://docs.python.org/zh-cn/3.11/library/exceptions.html#base-classes](https://docs.python.org/zh-cn/3.11/library/exceptions.html#base-classes)

![](https://raw.githubusercontent.com/BaihlUp/Figurebed/master/2023/202309211044954.png)

## 9.2 异常处理
- **try-except**

```python
try:
	可能产生异常的代码
except 异常:
	捕获指定的异常后运行的代码
```
示例：
```python
num = 1
num2 = 0
try:
    num / num2

except Exception as e:
    print("上一段程序有异常")
    print(e)
```
输出：
```shell
上一段程序有异常
division by zero
```

- **else代码块**

```python
try:
	可能产生异常的代码
except 异常:
	捕获指定的异常后运行的代码
else:
	try 部分的代码没有抛出异常，执行此部分代码
```

- **finally 代码块**

```python
try:
	可能产生异常的代码
except 异常:
	捕获指定的异常后运行的代码
finally:
	无论是否抛出异常，该部分代码均会执行
```

- **捕获多个异常**

```python
try:
	可能产生异常的语句
except 异常1:
	处理方式一
except 异常2:
	处理方式二
except ...
```
> 捕获多个异常，也可以使用嵌套的方法

## 9.3 自定义异常

使用 `raise` 抛出异常：
```python
class NameError(Exception):
    def __init__(self, message):
        self.message = message
    
    @property
    def msg(self):
        return f"名字不允许使用 {self.message}"


name = "jerry"
try:
    if name == "jerry":
        raise NameError(name)

except NameError as ne:
    print(ne.msg)
```
输出：
```shell
名字不允许使用 jerry
```


# 10 Python和C++混合编程
1. 使用ctypes库加载 C++ 编写的动态链接库：[https://docs.python.org/zh-cn/3.10/library/ctypes.html](https://docs.python.org/zh-cn/3.10/library/ctypes.html)
2. 使用pybind将C++编译为Python库：[https://github.com/pybind/python_example](https://github.com/pybind/python_example)
3. 使用Pythran库将Python直接转换为C++代码：[https://pypi.org/project/pythran/](https://pypi.org/project/pythran/)

