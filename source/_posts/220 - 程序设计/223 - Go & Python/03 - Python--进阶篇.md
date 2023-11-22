---
title: Python编程语言--进阶篇
date: 2023-10-20 16:27:41
categories:
  - Go & Python
tags:
  - 程序设计
  - Python
---

# 0 参考资料
- [Python核心技术与实战 -- Geek](https://time.geekbang.org/column/intro/100026901)

# 1 Python对象的比较、拷贝
## 1.1 比较
**`is` 操作符和 `==`操作符：**

在 Python 中，每个对象的身份标识，都能通过函数 id(object) 获得。因此，`'is'`操作符，相当于比较对象之间的 ID 是否相等，
```python
a = 10
b = 10
 
a == b
True
 
id(a)
4427562448
 
id(b)
4427562448
 
a is b
True
```
对于整型数字来说，以上`a is b`为 True 的结论，适用于 -5 到 256 范围内的数字。

```python
a = 257
b = 257
 
a == b
True
 
id(a)
4473417552
 
id(b)
4473417584
 
a is b
False
```
Python 内部会对 -5 到 256 的整型维持一个数组，起到一个缓存的作用。这样，每次你试图创建一个 -5 到 256 范围内的整型数字时，Python 都会从这个数组中返回相对应的引用，而不是重新开辟一块新的内存空间。
使用`'=='`的次数会比`'is'`多得多，因为我们一般更关心两个变量的值，而不是它们内部的存储地址。但是，当我们比较一个变量与一个单例（singleton）时，通常会使用`'is'`。一个典型的例子，就是检查一个变量是否为 None：

```python
if a is None:
      ...
 
if a is not None:
      ...
```
比较操作符`'is'`的速度效率，通常要优于`'=='`。因为`'is'`操作符不能被重载，这样，Python 就不需要去寻找，程序中是否有其他地方重载了比较操作符，并去调用。执行比较操作符`'is'`，就仅仅是比较两个变量的 ID 而已。

但是`'=='`操作符却不同，执行`a == b`相当于是去执行`a.__eq__(b)`，而 Python 大部分的数据类型都会去重载`__eq__`这个函数，其内部的处理通常会复杂一些。比如，对于列表，`__eq__`函数会去遍历列表中的元素，比较它们的顺序和值是否相等。

## 1.2 深拷贝和浅拷贝
- **浅拷贝**

常见的浅拷贝的方法，是使用数据类型本身的构造器，比如下面两个例子：

```python
l1 = [1, 2, 3]
l2 = list(l1)
 
l2
[1, 2, 3]
 
l1 == l2
True
 
l1 is l2
False
 
s1 = set([1, 2, 3])
s2 = set(s1)
 
s2
{1, 2, 3}
 
s1 == s2
True
 
s1 is s2
False
```
l2 就是 l1 的浅拷贝，s2 是 s1 的浅拷贝。对于可变的序列，还可以通过切片操作符`':'`完成浅拷贝，比如下面这个列表的例子：
```python
l1 = [1, 2, 3]
l2 = l1[:]
 
l1 == l2
True
 
l1 is l2
False
```
Python 中也提供了相对应的函数 copy.copy()，适用于任何数据类型：
```python
import copy
l1 = [1, 2, 3]
l2 = copy.copy(l1)
```
需要注意的是，对于元组，使用 tuple() 或者切片操作符`':'`不会创建一份浅拷贝，相反，它会返回一个指向相同元组的引用：
```python
t1 = (1, 2, 3)
t2 = tuple(t1)
 
t1 == t2
True
 
t1 is t2
True
```

浅拷贝，是指重新分配一块内存，创建一个新的对象，里面的元素是原对象中子对象的引用。如下：
```python
l1 = [1, 2, 3, [4, 5]]  
l2 = list(l1)  
print(id(l1), id(l2))  # 4372080128 4371268224  
print(id(l1[3]), id(l2[3])) # 4302757312 4302757312  元素引用，地址一样
print(l1 is l2)  # False
```
如果原对象中的元素不可变，那倒无所谓；但如果元素可变，浅拷贝通常会带来一些副作用，尤其需要注意。我们来看下面的例子：
```python
l1 = [[1, 2], (30, 40)]
l2 = list(l1)
l1.append(100)
l1[0].append(3)
 
l1
[[1, 2, 3], (30, 40), 100]
 
l2
[[1, 2, 3], (30, 40)]
 
l1[1] += (50, 60)
l1
[[1, 2, 3], (30, 40, 50, 60), 100]
 
l2
[[1, 2, 3], (30, 40)]
```
因为浅拷贝里的元素是对原对象元素的引用，因此 l2 中的元素和 l1 指向同一个列表和元组对象。
`l1.append(100)`，表示对 l1 的列表新增元素 100。这个操作不会对 l2 产生任何影响，因为 l2 和 l1 作为整体是两个不同的对象，并不共享内存地址。操作过后 l2 不变，l1 会发生改变。
`l1[0].append(3)`，这里表示对 l1 中的第一个列表新增元素 3。因为 l2 是 l1 的浅拷贝，l2 中的第一个元素和 l1 中的第一个元素，共同指向同一个列表，因此 l2 中的第一个列表也会相对应的新增元素 3。

- **深拷贝**

深度拷贝，是指重新分配一块内存，创建一个新的对象，并且将原对象中的元素，以递归的方式，通过创建新的子对象拷贝到新对象中。因此，新对象和原对象没有任何关联。
Python 中以 copy.deepcopy() 来实现对象的深度拷贝：

```python
import copy
l1 = [[1, 2], (30, 40)]
l2 = copy.deepcopy(l1)
l1.append(100)
l1[0].append(3)
 
l1
[[1, 2, 3], (30, 40), 100]
 
l2 
[[1, 2], (30, 40)]
```

可以看到，无论 l1 如何变化，l2 都不变。因为此时的 l1 和 l2 完全独立，没有任何联系。
深度拷贝也不是完美的，往往也会带来一系列问题。如果被拷贝对象中存在指向自身的引用，那么程序很容易陷入无限循环。

```python
import copy
x = [1]
x.append(x) # 列表 x 中有指向自身的引用，因此 x 是一个无限嵌套的列表。
 
x
[1, [...]]
 
y = copy.deepcopy(x)  
y
[1, [...]]

print(x == y) # 报错：list比较会进行遍历，导致无限循环
```
深度拷贝函数 deepcopy 中会维护一个字典，记录已经拷贝的对象与其 ID。拷贝过程中，如果字典里已经存储了将要拷贝的对象，则会从字典直接返回，不会导致无限循环。如下是deepcopy源码：

```python
def deepcopy(x, memo=None, _nil=[]):
    """Deep copy operation on arbitrary Python objects.
    	
	See the module's __doc__ string for more info.
	"""
	
    if memo is None:
        memo = {}
    d = id(x) # 查询被拷贝对象 x 的 id
	y = memo.get(d, _nil) # 查询字典里是否已经存储了该对象
	if y is not _nil:
	    return y # 如果字典里已经存储了将要拷贝的对象，则直接返回
        ...    
```

# 2 值传递 or 引用传递
## 2.1 Python变量及其赋值
Python代码示例：
```python
a = 1
b = a
a = a + 1
```
前两行会让a、b同时指向 1 这个对象。
![](https://raw.githubusercontent.com/BaihlUp/Figurebed/master/2023/20231022095756.png)

最后一行，a的值变成2，会重新创建一个新的值为2的对象，让a指向它。b的值不变。
![](https://raw.githubusercontent.com/BaihlUp/Figurebed/master/2023/20231022095921.png)
通过这个例子你可以看到，这里的 a 和 b，开始只是两个指向同一个对象的变量而已，或者你也可以把它们想象成同一个对象的两个名字。

下边看一个列表的例子：

```python
l1 = [1, 2, 3]
l2 = l1
l1.append(4)
l1
[1, 2, 3, 4]
l2
[1, 2, 3, 4]
```
首先让列表 l1 和 l2 同时指向了 [1, 2, 3] 这个对象。
![](https://raw.githubusercontent.com/BaihlUp/Figurebed/master/2023/20231022100035.png)
由于列表是可变的，所以 l1.append(4) 不会创建新的列表，只是在原列表的末尾插入了元素 4，变成 [1, 2, 3, 4]。
![](https://raw.githubusercontent.com/BaihlUp/Figurebed/master/2023/20231022100058.png)
需要注意的是，Python 里的变量可以被删除，但是对象无法被删除。比如下面的代码：

```python
l = [1, 2, 3]
del l
```
del l 删除了 l 这个变量，从此以后你无法访问 l，但是对象 [1, 2, 3] 仍然存在。Python 程序运行时，其自带的垃圾回收系统会跟踪每个对象的引用。如果 [1, 2, 3] 除了 l 外，还在其他地方被引用，那就不会被回收，反之则会被回收。
> 使用 `l2 = l1` 会创建两个变量指向同一个列表，而使用 `l2 = list(l1)` 会创建一个新的包含相同元素的列表。

**总结：**

- 变量的赋值，只是表示让变量指向了某个对象，并不表示拷贝对象给变量；而一个对象，可以被多个变量所指向。
- 可变对象（列表，字典，集合等等）的改变，会影响所有指向该对象的变量。
- 对于不可变对象（字符串，整型，元祖等等），所有指向该对象的变量的值总是一样的，也不会改变。但是通过某些操作（+= 等等）更新不可变对象的值时，会返回一个新的对象。
- 变量可以被删除，但是对象无法被删除。

## 2.2 Python函数的参数传递
Python 的参数传递是**赋值传递** （pass by assignment），或者叫作对象的**引用传递**（pass by object reference）。Python 里所有的数据类型都是对象，所以参数传递时，只是让新变量与原变量指向相同的对象而已，并不存在值传递或是引用传递一说。
```python
def my_func2(b):
	b = 2
	return b
 
a = 1
a = my_func2(a)
a
2
```
函数my_func2传递a时，a，b都指向1，执行`b=2`后，b指向了一个新的对象2，a不变，如果要改变a，可以通过返回值赋值方式。（Python中无法通过引用方式改变参数值）

当可变对象当作参数传入函数里的时候，改变可变对象的值，就会影响所有指向它的变量。比如下面的例子：

```python
def my_func3(l2):
	l2.append(4)
 
l1 = [1, 2, 3]
my_func3(l1)
l1
[1, 2, 3, 4]
```
这里 l1 和 l2 先是同时指向值为 [1, 2, 3] 的列表。不过，由于列表可变，执行 append() 函数，对其末尾加入新元素 4 时，变量 l1 和 l2 的值也都随之改变了。

```python
def my_func4(l2):
	l2 = l2 + [4]  # 创建了新的对象赋值给l2
 
l1 = [1, 2, 3]
my_func4(l1)
l1
[1, 2, 3]
```
为什么 l1 仍然是 [1, 2, 3]，而不是 [1, 2, 3, 4] 呢？
要注意，这里 l2 = l2 + [4]，表示创建了一个“末尾加入元素 4“的新列表，并让 l2 指向这个新的对象。这个过程与 l1 无关，因此 l1 的值不变。当然，同样的，如果要改变 l1 的值，我们就得让上述函数返回一个新列表，再赋予 l1 即可：
```python
def my_func5(l2):
	l2 = l2 + [4]
	return l2
 
l1 = [1, 2, 3]
l1 = my_func5(l1)
l1
[1, 2, 3, 4]
```
my_func3() 和 my_func5() 的用法，两者虽然写法不同，但实现的功能一致。不过，在实际工作应用中，往往倾向于类似 my_func5() 的写法，添加返回语句。

**总结：**

- 如果对象是可变的，当其改变时，所有指向这个对象的变量都会改变。
- 如果对象不可变，简单的赋值只能改变其中一个变量的值，其余变量则不受影响。

>如果你想通过一个函数来改变某个变量的值，通常有两种方法。一种是直接将可变数据类型（比如列表，字典，集合）当作参数传入，直接在其上修改；第二种则是创建一个新变量，来保存修改后的值，然后将其返回给原变量。在实际工作中，我们更倾向于使用后者，因为其表达清晰明了，不易出错。

**示例程序：**

```python
def func(d):  
    d['a'] = 10  
    d['b'] = 20  
    d = {'a': 1, 'b': 2}  # 重新定义了新的字典，d为函数func函数内的局部变量
  
if __name__ == "__main__":  
    d = {}  
    func(d)  
    print(d)
```
输出：

```bash
{'a': 10, 'b': 20}
```


# 3 装饰器
## 3.1 函数装饰器
函数的基本用法包括：
1. 函数参数传递变量
2. 函数当作参数传递
3. 函数中嵌套函数
4. 函数的返回值可以是函数对象（闭包）

```python
def my_decorator(func):
    def wrapper():
        print('wrapper of decorator')
        func()
    return wrapper
 
def greet():
    print('hello world')
 
greet = my_decorator(greet)
greet()
 
# 输出
wrapper of decorator
hello world
```

这段代码中，变量 greet 指向了内部函数 wrapper()，而内部函数 wrapper() 中又会调用原函数 greet()，因此，最后调用 greet() 时，就会先打印`'wrapper of decorator'`，然后输出`'hello world'`。
这里的函数 my_decorator() 就是一个装饰器，它把真正需要执行的函数 greet() 包裹在其中，并且改变了它的行为，但是原函数 greet() 不变。

事实上，上述代码在 Python 中有更简单、更优雅的表示：
```python
def my_decorator(func):
    def wrapper():
        print('wrapper of decorator')
        func()
    return wrapper
 
@my_decorator
def greet():
    print('hello world')
 
greet()
```
这里的`@`，我们称之为语法糖，`@my_decorator`就相当于前面的`greet=my_decorator(greet)`语句，只不过更加简洁。因此，如果你的程序中有其它函数需要做类似的装饰，你只需在它们的上方加上`@decorator`就可以了，这样就大大提高了函数的重复利用和程序的可读性。

- **带有参数的装饰器**

`*args`和`**kwargs`，表示接受任意数量和类型的参数：
```python
def my_decorator(func):
    def wrapper(*args, **kwargs):
        print('wrapper of decorator')
        func(*args, **kwargs)
    return wrapper
```

- **带有自定义参数的装饰器**

```python
def repeat(num):
    def my_decorator(func):
        def wrapper(*args, **kwargs):
            for i in range(num):
                print('wrapper of decorator')
                func(*args, **kwargs)
        return wrapper
    return my_decorator
 
 
@repeat(4)  # 先执行 repeat(4) 函数，返回的是一个装饰器
def greet(message):
    print(message)
 
greet('hello world')
 
# 输出：
wrapper of decorator
hello world
wrapper of decorator
hello world
wrapper of decorator
hello world
wrapper of decorator
hello world
```

- **保留原函数**

```python
greet.__name__
## 输出
'wrapper'
 
help(greet)
# 输出
Help on function wrapper in module __main__:
 
wrapper(*args, **kwargs)
```
greet() 函数被装饰以后，它的元信息变了。元信息告诉我们“它不再是以前的那个 greet() 函数，而是被 wrapper() 函数取代了”。

通常使用内置的装饰器`@functools.wrap`，它会帮助保留原函数的元信息（也就是将原函数的元信息，拷贝到对应的装饰器函数里）。
```python
import functools
 
def my_decorator(func):
    @functools.wraps(func)
    def wrapper(*args, **kwargs):
        print('wrapper of decorator')
        func(*args, **kwargs)
    return wrapper
    
@my_decorator
def greet(message):
    print(message)
 
greet.__name__
 
# 输出
'greet'
```

## 3.2 类装饰器
- 类的装饰器是类方法的装饰器的缩写
- 可以通过装饰器改变方法的调用方式和行为

### 3.2.1  `__call__` 模式方法
类装饰器主要依赖于函数`__call_()`，每当你调用一个类的示例时，函数`__call__()`就会被执行一次。
```python
class Count:
    def __init__(self, func):
        self.func = func
        self.num_calls = 0
	# 类默认没有 __call__ 模式方法，当增加 __call__ 方法后则可以调用类
    def __call__(self, *args, **kwargs):
        self.num_calls += 1
        print('num of calls is: {}'.format(self.num_calls))
        return self.func(*args, **kwargs)
 
@Count
def example():
    print("hello world")
 
example()
 
# 输出
num of calls is: 1
hello world
 
example()
 
# 输出
num of calls is: 2
hello world
 
...
```
定义了类 Count，初始化时传入原函数 func()，而`__call__()`函数表示让变量 num_calls 自增 1，然后打印，并且调用原函数。因此，在我们第一次调用函数 example() 时，num_calls 的值是 1，而在第二次调用时，它的值变成了 2。

- **在函数中**

```python
def func1():  
	pass  
dir(func1)
```
输出：
```shell
['__annotations__',
 '__builtins__',
 '__call__',  # 类中没有此方法
 '__class__',
 '__closure__',
 '__code__',
 '__defaults__',
 '__delattr__',
 '__dict__',
 '__dir__',
 '__doc__',
 '__eq__',
 ...
```

- **在类中**

```python
class Class1:
    pass

cls = Class1()
cls()   # 报错
```
增加 `__call__` :
```python
class Class1:
    def __call__(self, *args, **kwargs):
        print("class is run")

cls = Class1()
cls()
```
输出：
```shell
class is run
```

### 3.2.2 classmethod 装饰器
classmethod 修饰的方法定义为类的方法，用于类直接调用。
```python
class Klass1:
      @classmethod
      def funcs(cls): 
           print("Class method")
        
Klass1.funcs()
```
输出：
```shell
Class method
```

**用途：** `classmethod` 用于定义类方法，类方法与类相关联，而不是与类的实例相关联。它们可以访问类级别的属性和方法，但不能直接访问实例级别的属性和方法。通常用于实现与类相关的功能，而不需要创建类的实例。
**调用方式：** 类方法的第一个参数通常被命名为 `cls`，它表示类本身，可以使用它来访问类的属性和调用其他类方法。类方法可以通过类名或类的实例调用。

```python
class MyClass:
    class_variable = 0

    def __init__(self, value):
        self.value = value

    @classmethod
    def increment_class_variable(cls):
        cls.class_variable += 1

# 使用类方法
MyClass.increment_class_variable()
print(MyClass.class_variable)  # 1
```

### 3.2.3 staticmethod 装饰器
- **用途：** `staticmethod` 用于定义静态方法，静态方法与类相关联，但不依赖于类的实例。它们不能访问类级别的属性或实例级别的属性。通常用于实现与类相关但不需要访问实例状态的功能。
- **调用方式：** 静态方法没有特殊的参数，可以通过类名或类的实例调用。

```python
class MathUtils:
    @staticmethod
    def add(x, y):
        return x + y
	def func(self):
		return self.add(1, 2) # 实例方法可以直接调用静态方法和类方法

# 使用静态方法
result = MathUtils.add(5, 3)  # 8
```

### 3.2.4 `classmethod` 和 `staticmethod`的区别

- 主要区别在于能否访问类属性和实例属性。类方法可以访问类属性，但不能访问实例属性，而静态方法既不能访问类属性也不能访问实例属性。
    
- 使用 `classmethod` 主要是为了在方法内部操作类级别的属性或实现与类相关的逻辑，而使用 `staticmethod` 主要是为了封装与类相关但与实例无关的功能。
    
- 如果你需要在方法内部访问或修改类级别的属性，或者需要与类相关的操作，使用 `classmethod`。如果方法不依赖于类或实例的状态，使用 `staticmethod`。

### 3.2.5 property 修饰器
`property` 修饰器是一种用于创建属性的特殊装饰器，它允许你定义一个方法，这个方法可以像访问属性一样被调用，而不需要使用函数调用的方式。这样可以隐藏属性的内部实现细节，同时可以提供更多的控制和验证。

1. **创建只读属性**

```python
class Circle:
    def __init__(self, radius):
        self._radius = radius

    @property
    def radius(self):
        return self._radius

# 创建 Circle 类的实例
circle = Circle(5)
# 访问只读属性
print(circle.radius)
# 尝试修改属性会引发 AttributeError
# circle.radius = 10  # 这会引发 AttributeError
```

2. **创建可读写属性**

提供一个与属性同名的setter方法，用户设置属性的值。
```python
class Circle:
    def __init__(self, radius):
        self._radius = radius

    @property
    def radius(self):
        return self._radius

    @radius.setter
    def radius(self, value):
        if value < 0:
            raise ValueError("Radius cannot be negative")
        self._radius = value

# 创建 Circle 类的实例
circle = Circle(5)
# 访问可读写属性
print(circle.radius)
# 设置属性的值
circle.radius = 10
print(circle.radius)
```
定义`@property`和`@radius.setter`是配对出现，不可直接定义`@radius.setter`。

3. **创建可删除属性**

```python
class Circle:
    def __init__(self, radius):
        self._radius = radius

    @property
    def radius(self):
        return self._radius

    @radius.setter
    def radius(self, value):
        if value < 0:
            raise ValueError("Radius cannot be negative")
        self._radius = value

    @radius.deleter
    def radius(self):
        print("Deleting radius")
        del self._radius

# 创建 Circle 类的实例
circle = Circle(5)
# 删除属性
del circle.radius
```
在执行 del 属性时，会自动执行 deleter 属性方法。可删除属性可以做一些实例收尾操作，比如连接数据库，在清理数据库连接时。

> 通过使用 `property`可以隐藏内部实现细节，封装属性的访问和修改，从而提供更多的控制和验证。在定义时常用下划线前缀（例如`_radius`）来表示属性是受保护的。



## 3.3 装饰器的嵌套

```python
@decorator1
@decorator2
@decorator3
def func():
    ...
```

执行顺序从里到外，上边的代码等价如下：
```python
decorator1(decorator2(decorator3(func)))

```

**示例：**
```python
import functools
 
def my_decorator1(func):
    @functools.wraps(func)
    def wrapper(*args, **kwargs):
        print('execute decorator1')
        func(*args, **kwargs)
    return wrapper
 
 
def my_decorator2(func):
    @functools.wraps(func)
    def wrapper(*args, **kwargs):
        print('execute decorator2')
        func(*args, **kwargs)
    return wrapper
 
 
@my_decorator1
@my_decorator2
def greet(message):
    print(message)
 
 
greet('hello world')
 
# 输出
execute decorator1
execute decorator2
hello world
```

## 3.4 装饰器的应用实例
### 3.4.1 身份认证
在某函数执行前做身份认证，如果未认证则抛出异常。
```python
import functools
 
def authenticate(func):
    @functools.wraps(func)
    def wrapper(*args, **kwargs):
        request = args[0]
        if check_user_logged_in(request): # 如果用户处于登录状态
            return func(*args, **kwargs) # 执行函数 post_comment() 
        else:
            raise Exception('Authentication failed')
    return wrapper
    
@authenticate
def post_comment(request, ...)
    ...
 
```

### 3.4.2 日志记录
统计日志记录某函数的执行时间：
```python
import time
import functools
 
def log_execution_time(func):
    @functools.wraps(func)
    def wrapper(*args, **kwargs):
        start = time.perf_counter()
        res = func(*args, **kwargs)
        end = time.perf_counter()
        print('{} took {} ms'.format(func.__name__, (end - start) * 1000))
        return res
    return wrapper
    
@log_execution_time
def calculate_similarity(items):
    ...
```
装饰器 log_execution_time 记录某个函数的运行时间，并返回其执行结果。如果你想计算任何函数的执行时间，在这个函数上方加上`@log_execution_time`即可。

### 3.4.3 输入合理性检查
```python
import functools
 
def validation_check(input):
    @functools.wraps(func)
    def wrapper(*args, **kwargs): 
        ... # 检查输入是否合法
    
@validation_check
def neural_network_training(param1, param2, ...):
    ...
```

### 3.4.4 缓存
LRU cache，在 Python 中的表示形式是`@lru_cache`。`@lru_cache`会缓存进程中的函数参数和结果，当缓存满了以后，会删除 least recenly used 的数据。
使用缓存装饰器，来包裹这些检查函数，避免其被反复调用，进而提高程序运行效率，比如写成下面这样：
```python
from functools import lru_cache
@lru_cache
def check(param1, param2, ...) # 检查用户设备类型，版本号等等
    ...
```

通过使用 `lru_cache` 提升斐波那契数列计算时间
![](https://raw.githubusercontent.com/BaihlUp/Figurebed/master/2023/202309221129781.png)

# 4 metaclass
## 4.1 type类
所有的 Python 的用户定义类，都是 type 这个类的实例，在Python中type这个类就是造物的上帝，可以通过如下代码查看：

```python
# Python 3 和 Python 2 类似
class MyClass:
  pass
 
instance = MyClass()
 
type(instance)
# 输出
<class '__main__.C'>
 
type(MyClass)
# 输出
<class 'type'>
```

- **用户自定义类，只不过是 type 类的`__call__`运算符重载**

当我们定义一个类的语句结束时，真正发生的情况，是 Python 调用 type 的`__call__`运算符。

```python
class MyClass:
  data = 1
```
Python真正执行的是下面的代码：
```python
class = type(classname, superclasses, attributedict)
```

这里等号右边的`type(classname, superclasses, attributedict)`，就是 type 的`__call__`运算符重载，它会进一步调用：
```python
type.__new__(typeclass, classname, superclasses, attributedict)
type.__init__(class, classname, superclasses, attributedict)
```

通过type定义MyClass类：
```python
class MyClass:
  data = 1
  
instance = MyClass()
MyClass, instance
# 输出
(__main__.MyClass, <__main__.MyClass instance at 0x7fe4f0b00ab8>)
instance.data
# 输出
1
 
MyClass = type('MyClass', (), {'data': 1})
instance = MyClass()
MyClass, instance
# 输出
(__main__.MyClass, <__main__.MyClass at 0x7fe4f0aea5d0>)
 
instance.data
# 输出
1
```

通过上面可以看到，正常的 MyClass 定义，和手工去调用 type 运算符的结果是完全一样的。

**示例：**

```python
class_body = """
def greeting(self):
    print('Hello customer')
    
def jump(self):
    print('jump')
"""
class_dict = {}
exec(class_body, globals(), class_dict)

# type的第三个参数是具体类的属性，是一个字典
Customer = type("Customer", (object,), class_dict)

c = Customer()
c.greeting()
c.jump()
```
输出：

```shell
Hello customer
jump
```
以上代码，通过type类实现了一个`Customer`类，通过字符串内容定义了类的属性，使用此方法，可以实现动态定义类。

## 4.2 metaclass的使用
把一个类型 MyClass 的 metaclass 设置成 MyMeta，MyClass 就不再由原生的 type 创建，而是会调用 MyMeta 的`__call__`运算符重载。

**自定义metaclass类创建类：**

```python
class Human(type):  
    @staticmethod  
    def __new__(mcs, *args, **kwargs):  
        class_ = super().__new__(mcs, *args)  
        # class_.freedom = True  
        if kwargs:  
            for name, value in kwargs.items():  
                setattr(class_, name, value)  
        return class_  
  
  
class Student(object, metaclass=Human, country="China", freedom=True):  
    pass  
  
  
print(Student.country)  # China
print(Student.freedom)  # True
```


# 5 迭代器和生成器
## 5.1 迭代器
可迭代对象，通过 iter() 函数返回一个迭代器，再通过 next() 函数就可以实现遍历。for in 语句将这个过程隐式化。

```python
def is_iterable(param):
    try: 
        iter(param) 
        return True
    except TypeError:
        return False
 
params = [
    1234,
    '1234',
    [1, 2, 3, 4],
    set([1, 2, 3, 4]),
    {1:1, 2:2, 3:3, 4:4},
    (1, 2, 3, 4)
]
    
for param in params:
    print('{} is iterable? {}'.format(param, is_iterable(param)))
 
########## 输出 ##########
 
1234 is iterable? False
1234 is iterable? True
[1, 2, 3, 4] is iterable? True
{1, 2, 3, 4} is iterable? True
{1: 1, 2: 2, 3: 3, 4: 4} is iterable? True
(1, 2, 3, 4) is iterable? True
```

列表转换成迭代器：
```python
l1 = [1,2,3,4]  
l2 = iter(l1)  
print(l1)   # [1, 2, 3, 4]
print(l2)   # <list_iterator object at 0x104887010>
```

## 5.2 生成器
### 5.2.1 生成器的使用
**生成器是懒人版本的迭代器**，在迭代器中，如果我们想要枚举它的元素，这些元素需要事先生成，但是生成器，是在调用 `next()`函数的时候，才会生成下一个变量，并不需要在内存中同时保存太多值。

```python
import os  
import psutil  
  
  
# 显示当前 python 程序占用的内存大小  
def show_memory_info(hint):  
    pid = os.getpid()  
    p = psutil.Process(pid)  
  
    info = p.memory_full_info()  
    memory = info.uss / 1024. / 1024  
    print('{} memory used: {} MB'.format(hint, memory))  
  
  
def test_iterator():  
    show_memory_info('initing iterator')  
    list_1 = [i for i in range(100000000)]  
    show_memory_info('after iterator initiated')  
    print(sum(list_1))  
    show_memory_info('after sum called')  
  
  
def test_generator():  
    show_memory_info('initing generator')  
    list_2 = (i for i in range(100000000))  
    show_memory_info('after generator initiated')  
    print(sum(list_2))  
    show_memory_info('after sum called')  
  
test_iterator()  
print("\n")  
test_generator()
```

输出：
```shell
initing iterator memory used: 9.34375 MB
after iterator initiated memory used: 1146.734375 MB
4999999950000000
after sum called memory used: 3452.4375 MB

initing generator memory used: 4.578125 MB
after generator initiated memory used: 4.609375 MB
4999999950000000
after sum called memory used: 4.609375 MB
```
使用生成器时，不需要在内存中同时保存对元素求和，我们只需要知道每个元素在相加的那一刻是多少就行了，用完就可以扔掉了。
相对迭代器，使用生成器省掉更多的内存。

### 5.2.2 生成器更多用法
含有yield的函数，会返回一个生成器，每次执行到yield，会把对应的值返回出去，并且函数暂停，等待下次被唤醒。
```python
def generator(k):  
    i = 1  
    while True:  
        yield i ** k  
        i += 1  
  
gen_1 = generator(1)  
gen_3 = generator(3)  
print(gen_1)  
print(gen_3)  
  
def get_sum(n):  
    sum_1, sum_3 = 0, 0  
    for i in range(n):  
        next_1 = next(gen_1)  
        next_3 = next(gen_3)  
        print('next_1 = {}, next_3 = {}'.format(next_1, next_3))  
        sum_1 += next_1  
        sum_3 += next_3  
    print(sum_1, sum_3)  
  
get_sum(8)
```
generator() 这个函数，返回了一个生成器，执行到yield时程序会从这里暂停，每次 next(gen) 函数被调用的时候，暂停的程序就又复活了，从 yield 这里向下继续执行。通过yield实现了返回了一个k次幂的生成器。

下边看一个使用生成器，确认target元素在列表的位置：
```python
def index_generator(L, target):
    for i, num in enumerate(L):
        if num == target:
            yield i
 
print(list(index_generator([1, 6, 2, 4, 5, 2, 8, 6, 3, 2], 2)))
```
以上 index_generator 返回了一个生成器，然后使用 list 转换为列表，默认会执行 遍历生成器的元素。

### 5.2.3 使用生成器实现判断子序列
给定两个序列，判定第一个是不是第二个的子序列：序列就是列表，子序列则指的是，一个列表的元素在第二个列表中都按顺序出现，但是并不必挨在一起。举个例子，[1, 3, 5] 是 [1, 2, 3, 4, 5] 的子序列，[1, 4, 3] 则不是。

```python
def is_subsequence(a, b):
    b = iter(b)
    return all(i in b for i in a)
 
print(is_subsequence([1, 3, 5], [1, 2, 3, 4, 5]))
print(is_subsequence([1, 4, 3], [1, 2, 3, 4, 5]))
```
代码注解：
1. is_subsequence 函数先把 b 转换为迭代器
2. 通过 `(i for i in a)` 会生成一个生成器
3. 通过 `(i in b)` 判断 i 是否在 列表 b 中，如果在 b 中则返回 True，若不在则返回 Flase。最后返回一个含有多个 bool值的列表。
4. 最后的 all() 函数用来判断一个迭代器的元素是否全部为 True，如果是则返回 True，否则就返回 False。

具体解释下 `(i in b)` 大概等价于下面的代码：

```python
while True:
    val = next(b)
    if val == i:
        yield True
```
利用生成器的特性，next() 函数运行的时候，**保存了当前的指针**，如下示例：
```python
b = (i for i in range(5))
 
print(2 in b)
print(4 in b)
print(3 in b)
 
########## 输出 ##########
 
True
True
False 
```

# 6 并发编程--Python 协程
## 6.1 asyncio使用
协程事件循环：
![](https://raw.githubusercontent.com/BaihlUp/Figurebed/master/2023/%E6%88%AA%E5%B1%8F2023-10-22%2018.15.17.png)

在Python 3.7 基于 asyncio 和 async / await 的方法使用协程。
```python
import asyncio
 
async def crawl_page(url):
    print('crawling {}'.format(url))
    sleep_time = int(url.split('_')[-1])
    await asyncio.sleep(sleep_time)
    print('OK {}'.format(url))
 
async def main(urls):
    for url in urls:
        await crawl_page(url)
 
%time asyncio.run(main(['url_1', 'url_2', 'url_3', 'url_4']))
 
########## 输出 ##########
 
crawling url_1
OK url_1
crawling url_2
OK url_2
crawling url_3
OK url_3
crawling url_4
OK url_4
Wall time: 10 s
```
在函数前通过 async 定义了协程函数，使用 await 把协程函数加入事件循环，并等待协程函数完成，使用 asyncio.run 执行协程函数。
如果你 `print(crawl_page(''))`，便会输出`<coroutine object crawl_page at 0x000002BEDF141148>`，提示你这是一个 Python 的协程对象，而并不会真正执行这个函数。

## 6.2 asyncio创建任务
以上方式在main中直接使用 await 调用协程函数，会让main函数阻塞，程序执行完的总时间是所有协程函数执行时间之和，下边使用asyncio创建任务执行：

```python
import asyncio
import time


async def call_api(name: str, delay: float):
    print(f"{name} - step 1")
    await asyncio.sleep(delay)
    print(f"{name} - step 2")


async def main():
    time_1 = time.perf_counter()
    print("start A coroutine")
    task_1 = asyncio.create_task(call_api("A", 2))

    print("start B coroutine")
    task_2 = asyncio.create_task(call_api("B", 5))

    await task_1
    print("task 1 completed")
    await task_2
    print("task 2 completed")

    time_2 = time.perf_counter()
    print(f"Spent {time_2 - time_1}")


asyncio.run(main())
```
输出：

```shell
start A coroutine
start B coroutine
A - step 1
B - step 1
A - step 2
task 1 completed
B - step 2
task 2 completed
Spent 5.002599277999252
```

通过`asyncio.create_task`只是把协程函数放到队列中，直接返回，然后由时间循环进行调度执行，使用 await 等待任务执行完毕，以上方式执行总时间是所有协程函数最长的那个。

## 6.3 asyncio进阶用法
把正在执行的任务取消、判断任务是否完成、为任务设置超时时间等。

```python
import asyncio
from asyncio.exceptions import TimeoutError


async def play_music(music: str):
    print(f"Start playing {music}")
    await asyncio.sleep(3)
    print(f"Finished playing {music}")

    return music


async def call_api():
    print("calling api.....")
    raise Exception("Error calling")


async def my_cancel():
    task = asyncio.create_task(play_music("A"))

    await asyncio.sleep(3)

    if not task.done():
        task.cancel()  # 取消任务


async def my_cancel_with_timeout():
    task = asyncio.create_task(play_music("B"))

    try:
        await asyncio.wait_for(task, timeout=2)  # 任务超时会抛出异常后自动取消
    except TimeoutError:
        print("timeout")


async def my_timeout():
    task = asyncio.create_task(play_music("B"))

    try:
        await asyncio.wait_for(asyncio.shield(task), timeout=2)  # 任务超时抛出异常，但不取消任务
    except TimeoutError:
        print("timeout")
        await task   # 任务超时，正常执行完退出


async def my_gather():
    results = await asyncio.gather(play_music("A"), play_music("B"))  # 等待多个协程执行完成，并获取协程执行结果
    print(results)


async def my_gather_with_exception():
    results = await asyncio.gather(play_music("A"), play_music("B"), call_api(),
                                   return_exceptions=True)
    print(results) # 设置 return_exceptions 获取异常协程的结果，如果不设置，则任意一个协程抛出异常，则程序异常退出

if __name__ == "__main__":
    asyncio.run(my_gather_with_exception())
```
1. `task.cancel()` 取消任务
2. `task.done()` 任务完成为True，否则为 Flase
3. `asyncio.wait_for` 为任务设置超时时间
4. `asyncio.gather` 获取多个协程执行结果，若有一个协程异常则退出，设置`return_exceptions=True` 则不退出，正常获取协程异常结果

## 6.4 实战和总结
通过协程实现获取豆瓣数据：

```python
import asyncio
import aiohttp
 
from bs4 import BeautifulSoup
 
async def fetch_content(url):
    async with aiohttp.ClientSession(
        headers=header, connector=aiohttp.TCPConnector(ssl=False)
    ) as session:
        async with session.get(url) as response:
            return await response.text()
 
async def main():
    url = "https://movie.douban.com/cinema/later/beijing/"
    init_page = await fetch_content(url)
    init_soup = BeautifulSoup(init_page, 'lxml')
 
    movie_names, urls_to_fetch, movie_dates = [], [], []
 
    all_movies = init_soup.find('div', id="showing-soon")
    for each_movie in all_movies.find_all('div', class_="item"):
        all_a_tag = each_movie.find_all('a')
        all_li_tag = each_movie.find_all('li')
 
        movie_names.append(all_a_tag[1].text)
        urls_to_fetch.append(all_a_tag[1]['href'])
        movie_dates.append(all_li_tag[0].text)
 
    tasks = [fetch_content(url) for url in urls_to_fetch]
    pages = await asyncio.gather(*tasks)
 
    for movie_name, movie_date, page in zip(movie_names, movie_dates, pages):
        soup_item = BeautifulSoup(page, 'lxml')
        img_tag = soup_item.find('img')
 
        print('{} {} {}'.format(movie_name, movie_date, img_tag['src']))
 
%time asyncio.run(main())
 
########## 输出 ##########
 
阿拉丁 05 月 24 日 https://img3.doubanio.com/view/photo/s_ratio_poster/public/p2553992741.jpg
龙珠超：布罗利 05 月 24 日 https://img3.doubanio.com/view/photo/s_ratio_poster/public/p2557371503.jpg
五月天人生无限公司 05 月 24 日 https://img3.doubanio.com/view/photo/s_ratio_poster/public/p2554324453.jpg
... ...
直播攻略 06 月 04 日 https://img3.doubanio.com/view/photo/s_ratio_poster/public/p2555957974.jpg
Wall time: 4.98 s
```

**总结：**

- 协程和多线程的区别，主要在于两点，一是协程为单线程；二是协程由用户决定，在哪些地方交出控制权，切换到下一个任务。
- 协程的写法更加简洁清晰，把 async / await 语法和 create_task 结合来用，对于中小级别的并发需求已经毫无压力。
- 写协程程序的时候，你的脑海中要有清晰的事件循环概念，知道程序在什么时候需要暂停、等待 I/O，什么时候需要一并执行到底。

# 7 并发编程--多线程
## 7.1 线程
使用 `Thread` 创建线程，示例如下：
```python
from threading import Thread  
  
def task(count: int):  
    for n in range(count):  
        print(n)  
  
  
thread1 = Thread(target=task, args=(10,))  
thread2 = Thread(target=task, args=(20,))  

thread1.daemon = True  # 创建守护线程
thread2.daemon = True 

thread1.start()  
thread2.start()  
  
thread1.join()  # 等待线程结束
thread2.join()  
  
print("Main threads is end")
```
以上默认创建的是 非守护线程，执行`thread1.daemon = True`设置成守护线程， 主线程需要使用 join 等待守护线程结束，否则主程序结束后，线程可能未执行完。

- 守护线程会在主线程结束时候自动结束
- 主线程需要等到所有非守护线程结束才能结束（默认创建的为非守护线程）
- 守护线程一般用于执行后台任务和服务，如日志记录、监控、定时任务等

## 7.2 线程安全队列
queue模块中的Queue类提供了线程安全队列功能：
1. queue.put(item, block=False)  非阻塞写入数据到队列
2. queue.put(item, timeout=3)    阻塞超时3s
3. queue.get(block=False)
4. queue.get(timeout=10)
5. queue.qsize()
6. queue.empty()
7. queue.full()

通过继承 Thread 类创建线程，实现生产者和消费者：
```python
from threading import Thread
from queue import Queue  

class MsgProducer(Thread):  
    def __init__(self, name: str, count: int, queue: Queue):  
        super().__init__()  
  
        self.name = name  
        self.count = count  
        self.queue = queue  
  
    def run(self) -> None:  
        for n in range(self.count):  
            msg = f"{self.name} - {n}"  
            self.queue.put(msg, block=True)  
  
  
class MsgConsumer(Thread):  
    def __init__(self, name: str, queue: Queue):  
        super().__init__()  
  
        self.name = name  
        self.queue = queue  
        self.daemon = True  
  
    def run(self) -> None:  
        while True:  
            msg = self.queue.get(block=True)  
            print(f"{self.name} - {msg}\n", end='')  # 取消print默认带的换行符
  
  
queue = Queue(3)  
threads = list()  
threads.append(MsgProducer("PA", 10, queue))  
threads.append(MsgProducer("PB", 10, queue))  
threads.append(MsgProducer("PC", 10, queue))  
  
threads.append(MsgConsumer("CA", queue))  
threads.append(MsgConsumer("CB", queue))  
  
for t in threads:  
    t.start()
```

以上消费者线程设置为了守护线程，会等到所有生产者线程和主线程结束后，自动结束。run函数表示线程要执行的逻辑，主线程中创建了3个生产者线程和2个消费者线程。

## 7.3 线程锁
使用Lock让线程顺序执行：

```python
from threading import Thread, Lock, Condition  

task_lock = Lock()  
def task(name: str):  
    global task_lock  
    for n in range(2):  
        task_lock.acquire()  # 获取锁
        print(f"{name} - round {n} - step 1\n", end='')  
        print(f"{name} - round {n} - step 2\n", end='')  
        print(f"{name} - round {n} - step 3\n", end='')  
        task_lock.release()  # 释放锁
  
  
t1 = Thread(target=task, args=("A",))  
t2 = Thread(target=task, args=("B",))  
t3 = Thread(target=task, args=("C",))  
  
t1.start()  
t2.start()  
t3.start()
```
以上三个线程将依次执行task函数。

基于 list 自定义实现一个安全队列：

```python
from threading import Thread, Lock, Condition

class SafeQueue:  
    def __init__(self, size: int):  
        self.__item_list = list()  
        self.size = size  
        self.__item_lock = Condition()  
  
    def put(self, item):  
        with self.__item_lock:  # 使用with语句加锁，多线程可以并发访问 
            while len(self.__item_list) >= self.size:  
                self.__item_lock.wait()  # 队列满，阻塞等待
  
            self.__item_list.insert(0, item)  
            self.__item_lock.notify_all()  
  
    def get(self):  
        with self.__item_lock:  
            while len(self.__item_list) == 0:  
                self.__item_lock.wait()  # 队列空，阻塞等待
  
            result = self.__item_list.pop()  
            self.__item_lock.notify_all()  # 通知所有wait的线程
  
            return result
```


## 7.4 线程池
线程的创建和销毁相对比较昂贵，频繁的创建和销毁线程不利于高性能。

```python
import time  
from concurrent.futures import ThreadPoolExecutor

def task(name: str):  
    print(f"{name} - step 1\n", end='')  
    time.sleep(1)  
    print(f"{name} - step 2\n", end='')  
  
    return f"{name} complete"  

# 创建一个ThreadPoolExecutor，设置max_workers指定线程池的大小
with ThreadPoolExecutor(max_workers=4) as executor:
	# 提交任务给线程池
    result_1 = executor.submit(task, 'A')  
    result_2 = executor.submit(task, 'B')  

    # 获取任务的执行结果
    print(result_1.result())  
    print(result_2.result())  
  
with ThreadPoolExecutor() as executor:  
	# 批量提交多个任务
    results = executor.map(task, ['C', 'D'])  
  
    for r in results:  
        print(r)
```
`ThreadPoolExecutor` 类是一个用于管理线程池的工具，可用于异步执行函数或方法。
1. `executor.submit()` 方法提交任务给线程池，后边的参数是给任务传递的参数，可以多个
2. 创建`ThreadPoolExecutor` 类时，根据 `max_workers` 参数来控制线程池中的线程数量，默认根据CPU数量设置线程数
3. executor.map函数批量提交多个任务到线程池

**使用submit提交多个任务，示例：**

```python
import concurrent.futures
import requests
import time

def download_one(url):
    resp = requests.get(url)
    print('Read {} from {}'.format(len(resp.content), url))
 
def download_all(sites):
    with concurrent.futures.ThreadPoolExecutor(max_workers=5) as executor:
        to_do = []
        for site in sites:
            future = executor.submit(download_one, site)
            to_do.append(future)
            
        for future in concurrent.futures.as_completed(to_do):
            future.result()
```
以上使用 `concurrent.futures.as_completed` 函数处理多个并发任务的结果。它的作用是返回一个生成器，该生成器在任务完成时生成任务的 Future 对象，而不是按照它们完成的顺序。这使你可以处理任何任务的结果，而不必等待它们按照提交的顺序完成。

## 7.5 多线程还是Asyncio
**多线程和Asyncio的区别：**
多线程：
- 多线程是使用标准的线程和锁机制来实现并发的方式。
- 在多线程中，每个线程都是一个独立的执行单元，可以并发执行不同的任务。
- 多线程由于 Python 的 GIL（全局解释器锁）的限制，在同一时刻只能执行一个线程

Asyncio（协程）：
- asyncio 使用单线程和事件循环来管理异步协程任务。
- 在 asyncio 中，多个协程可以在同一线程中并发执行，但在某一时刻只有一个协程在执行，而不会涉及线程切换。
- asyncio 适用于 I/O 密集型任务，如网络通信和文件操作。由于避免了线程切换的开销，它通常比多线程更高效。

Asyncio（协程）可以通过编程控制协程切换，本质是一个线程在异步执行都个协程任务，避免了线程协换的开销，比多线程更加高效。

**多线程和Asyncio的选择：**

- 如果是 I/O bound，并且 I/O 操作很慢，需要很多任务 / 线程协同实现，那么使用 Asyncio 更合适。
- 如果是 I/O bound，但是 I/O 操作很快，只需要有限数量的任务 / 线程，那么使用多线程就可以了。
- 如果是 CPU bound，则需要使用多进程来提高程序运行效率，使用多线程是无效的。


# 8  GIL（全局解释器锁）
## 8.1 什么是GIL
GIL，是最流行的 Python 解释器 CPython 中的一个技术术语。它的意思是全局解释器锁，本质上是类似操作系统的 Mutex。每一个 Python 线程，在 CPython 解释器中执行时，都会先锁住自己的线程，阻止别的线程执行。CPython会轮流执行Python线程，这样一来，用户看到的就是“伪并行”——Python 线程在交错执行，来模拟真正并行的线程。

- 为什么需要GIL？

CPython 使用引用计数来管理内存，所有 Python 脚本中创建的实例，都会有一个引用计数，来记录有多少个指针指向它。当引用计数只有 0 时，则会自动释放内存。

```python
>>> import sys
>>> a = []
>>> b = a
>>> sys.getrefcount(a)
3
```
a 的引用计数是 3，因为有 a、b 和作为参数传递的 getrefcount 这三个地方，都引用了一个空列表。
这样一来，如果有两个 Python 线程同时引用了 a，就会造成引用计数的 race condition，引用计数可能最终只增加 1，这样就会造成内存被污染。因为第一个线程结束时，会把引用计数减少 1，这时可能达到条件释放内存，当第二个线程再试图访问 a 时，就找不到有效的内存了。

CPython 引进 GIL 其实主要就是这么两个原因：
- 一是设计者为了规避类似于内存管理这样的复杂的竞争风险问题（race condition）；
- 二是因为 CPython 大量使用 C 语言库，但大部分 C 语言库都不是原生线程安全的（线程安全会降低性能和增加复杂度）。

## 8.2 GIL如何工作的
下面这张图，就是一个 GIL 在 Python 程序的工作示例。其中，Thread 1、2、3 轮流执行，每一个线程在开始执行时，都会锁住 GIL，以阻止别的线程执行；同样的，每一个线程执行完一段后，会释放 GIL，以允许别的线程开始利用资源。
![](https://raw.githubusercontent.com/BaihlUp/Figurebed/master/2023/20231024092005.png)

CPython 中还有一个check_interval机制，CPython 解释器会去轮询检查线程 GIL 的锁住情况。每隔一段时间，Python 解释器就会强制当前线程去释放 GIL，这样别的线程才能有执行的机会。

- Python的线程安全

有了GIL并不代表Python就不需要考虑线程安全了，因为有check interval这种抢占机制。

## 8.3 如何绕过GIL
Python 的 GIL，是通过 CPython 的解释器加的限制。如果你的代码并不需要 CPython 解释器来执行，就不再受 GIL 的限制。
事实上，很多高性能应用场景都已经有大量的 C 实现的 Python 库，例如 NumPy 的矩阵运算，就都是通过 C 来实现的，并不受 GIL 影响。
**绕过 GIL 的大致思路有这么两种：**
1. 绕过 CPython，使用 JPython（Java 实现的 Python 解释器）等别的实现；
3. 把关键性能代码，放到别的语言（一般是 C++）中实现。

# 9 并发编程--多进程
## 9.1 多进程
基于`multiprocessing`包，`multiprocessing.Process` 类：
- `Process` 类用于创建新的进程。
- 使用 `target` 参数指定要在新进程中运行的函数。
- 使用 `args` 参数传递给目标函数的参数。
- 通过调用 `start()` 方法启动新进程。
- 通过调用 `join()` 方法等待新进程执行完成。

```python
import multiprocessing  
import time  
  
  
def task(name: str, count: int):  
    print(f"{name} - start\n", end='')  
    result = 0  
    for n in range(count):  
        result += n + 1  
    time.sleep(1)  
    print(f"{name} - end with {result}")  
  
  
def start_process_1():  
    process = multiprocessing.Process(target=task, args=["A", 100])  
  
    process.start()  
  
    process.join()  
  
    print("Main process over")  
  
  
def start_process_2():  
    args_list = [("A", 100), ("B", 99), ("C", 98)]  
    processes = [multiprocessing.Process(target=task, args=[name, count]) for name, count in args_list]  
  
    for p in processes:  
        p.start()  
  
    for p in processes:  
        p.join()  
  
  
if __name__ == "__main__":  
    start_process_2()
```
在使用多进程时需要 把代码放在 `__name__ == "__main__"` 中。

## 9.2 进程池
Python中实现多进程的包与多线程的类似，都可以使用futures包，示例如下：

```python
import concurrent.futures

def worker_function(x):
    return x * x

if __name__ == "__main__":
    numbers = [1, 2, 3, 4, 5]

    with concurrent.futures.ProcessPoolExecutor() as executor:
        results = list(executor.map(worker_function, numbers))

    print(results)
```


# 10 Python垃圾回收机制
- 计数引用
- 循环引用

调试内存泄漏的工具：objgraph

# 11 上下文管理器和With语句
## 11.1 With语句的使用
- **使用with语句自动关闭文件**

```python
for x in range(10000000):
    with open('test.txt', 'w') as f:
        f.write('hello')
```

通过使用以上的With语句方式，不再需要写关闭文件的操作。

- **自动释放锁**

```python
some_lock = threading.Lock()
with somelock:
    ...
```

## 11.2 上下文管理器的实现
### 11.2.1 基于类的上下文管理器
当我们用类来创建上下文管理器时，必须保证这个类包括方法`”__enter__()”`和方法`“__exit__()”`。其中，方法`“__enter__()”`返回需要被管理的资源，方法`“__exit__()”`里通常会存在一些释放、清理资源的操作，比如这个例子中的关闭文件等等。

```python
class FileManager:
    def __init__(self, name, mode):
        print('calling __init__ method')
        self.name = name
        self.mode = mode 
        self.file = None
        
    def __enter__(self):
        print('calling __enter__ method')
        self.file = open(self.name, self.mode)
        return self.file  # 赋值给with语句中as后边的变量
 
 
    def __exit__(self, exc_type, exc_val, exc_tb):
        print('calling __exit__ method')
        if self.file:
            self.file.close()
            
with FileManager('test.txt', 'w') as f:
    print('ready to write to file')
    f.write('hello world')
    
## 输出
calling __init__ method
calling __enter__ method
ready to write to file
calling __exit__ method
```

以上with语句执行逻辑：
1. 方法`“__init__()”`被调用，程序初始化对象 FileManager，使得文件名（name）是`"test.txt"`，文件模式 (mode) 是`'w'`；
2. with语句自动调用方法`“__enter__()”`，文件`“test.txt”`以写入的模式被打开，并且返回 FileManager 对象赋予变量 f；
3. 字符串`“hello world”`被写入文件`“test.txt”`；
4. 方法`“__exit__()”`被调用，负责关闭之前打开的文件流。

方法`“__exit__()”`中的参数`“exc_type, exc_val, exc_tb”`，分别表示 exception_type、exception_value 和 traceback。当执行含有上下文管理器的 with 语句时，如果有异常抛出，异常的信息就会包含在这三个变量中，传入方法`“__exit__()”`。

```python
class Foo:
    def __init__(self):
        print('__init__ called')        
 
    def __enter__(self):
        print('__enter__ called')
        return self
    
    def __exit__(self, exc_type, exc_value, exc_tb):
        print('__exit__ called')
        if exc_type:
            print(f'exc_type: {exc_type}')
            print(f'exc_value: {exc_value}')
            print(f'exc_traceback: {exc_tb}')
            print('exception handled')
        return True
    
with Foo() as obj:
    raise Exception('exception raised').with_traceback(None)
 
# 输出
__init__ called
__enter__ called
__exit__ called
exc_type: <class 'Exception'>
exc_value: exception raised
exc_traceback: <traceback object at 0x1046036c8>
exception handled
```

在 with 语句中手动抛出了异常“exception raised”，你可以看到，`“__exit__()”`方法中异常，被顺利捕捉并进行了处理。不过需要注意的是，如果方法`“__exit__()”`没有返回 True，异常仍然会被抛出。因此，如果你确定异常已经被处理了，请在`“__exit__()”`的最后，加上`“return True”`这条语句。

- **示例代码：（用上下文管理器，实现数据库连接）**

```python
class DBConnectionManager: 
    def __init__(self, hostname, port): 
        self.hostname = hostname 
        self.port = port 
        self.connection = None
  
    def __enter__(self): 
        self.connection = DBClient(self.hostname, self.port) 
        return self
  
    def __exit__(self, exc_type, exc_val, exc_tb): 
        self.connection.close() 
  
with DBConnectionManager('localhost', '8080') as db_client: 
```

实现了 DBconnectionManager 这个类，那么在程序每次连接数据库时，只需要简单地调用 with 语句即可，并不需要关心数据库的关闭、异常等等，显然大大提高了开发的效率。


### 11.2.2 基于生成器的上下文管理器
可以使用装饰器 contextlib.contextmanager，来定义自己所需的基于生成器的上下文管理器，用以支持 with 语句。
```python
from contextlib import contextmanager
 
@contextmanager
def file_manager(name, mode):
    try:
        f = open(name, mode)
        yield f
    finally:
        f.close()
        
with file_manager('test.txt', 'w') as f:
    f.write('hello world')
```
函数 file_manager() 是一个生成器，当执行 with 语句时，便会打开文件，并返回文件对象 f；当 with 语句执行完后，finally block 中的关闭文件操作便会执行。
使用基于生成器的上下文管理器时，我们不再用定义`“__enter__()”`和`“__exit__()”`方法，但请务必加上装饰器 @contextmanager。

**总结：**

- 基于类的上下文管理器更加 flexible，适用于大型的系统开发；
- 而基于生成器的上下文管理器更加方便、简洁，适用于中小型程序。