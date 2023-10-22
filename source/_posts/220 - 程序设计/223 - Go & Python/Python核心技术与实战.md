

# 2 进阶篇
## 2.1 Python对象的比较、拷贝
### 2.1.1 比较
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
对于整型数字来说，以上`a is b`为 True 的结论，只适用于 -5 到 256 范围内的数字。

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

### 2.1.2 深拷贝和浅拷贝
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

## 2.2 值传递 or 引用传递
### 2.2.1 Python变量及其赋值
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

### 2.2.2 Python函数的参数传递
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


## 2.3 装饰器
[[零基础学 Python]]

## 2.4 metaclass
没看懂，需要再研究

## 2.5 迭代器和生成器
### 2.5.1 迭代器
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

### 2.5.2 生成器
#### 2.5.2.1 生成器的使用
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

#### 2.5.2.2 生成器更多用法
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

#### 2.5.2.3 使用生成器实现判断子序列
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

## 2.6 Python 协程
### 2.6.1 asyncio使用
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

### 2.6.2 asyncio创建任务
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

### 2.6.3 asyncio进阶用法
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

### 2.6.4 实战和总结
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

