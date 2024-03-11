---
title: OpenResty 性能优化
date: 2023-11-21 16:27:41
categories:
  - 后端&架构
tags:
  - OpenResty
published: true
---
# 1 性能下降10倍的真凶：阻塞函数
OpenResty 之所以可以保持很高的性能，简单来说，是因为它借用了 Nginx 的事件处理和 Lua 的协程机制，所以：
* 在遇到网络 I/O 等需要等待返回才能继续的操作时，就会先调用 Lua 协程的 yield 把自己挂起，然后在 Nginx 中注册回调；
* 在 I/O 操作完成（也可能是超时或者出错）后，由 Nginx 回调 resume，来唤醒 Lua 协程。

这样的流程，保证了 OpenResty 可以一直高效地使用 CPU 资源，来处理所有的请求。
在这个处理流程中，如果没有使用 cosocket 这种非阻塞的方式，而是用阻塞的函数来处理 I/O，那么 LuaJIT 就不会把控制权交给 Nginx 的事件循环。这就会导致，其他的请求要一直排队等待阻塞的事件处理完，才会得到响应。

下面，我再来介绍几个常见的坑，也就是一些经常会被误用的阻塞函数；

## 1.1 执行外部命令
在业务中通常可以执行一些外部的命令和工具，来辅助完成一些操作。
比如杀掉某个进程：
```lua
os.execute("kill -HUP " .. pid)
```

或者拷贝文件、使用OpenSSL生成秘钥等耗时更久的一些操作：
```lua
os.execute(" cp test.exe /tmp ")
os.execute(" openssl genrsa -des3 -out private.pem 2048 ")
```
表面上看， os.execute 是 Lua 的内置函数，而在 Lua 世界中，也确实是用这种方式来调用外部命令的。
但是在 OpenResty 的环境中，os.execute 会阻塞当前请求。所以，如果这个命令的执行时间特别短，那么影响还不是很大。

**有两个解决方案如下：**
- **方案一：如果有 FFI 库可以使用，那么我们就优先使用 FFI 的方式来调用**

上面用 OpenSSL 的命令行来生成秘钥，就可以改为，用 FFI 调用 OpenSSL 的 C 函数的方式来绕过。
杀掉某个进程的示例，你可以使用 lua-resty-signal 这个 OpenResty 自带的库，来非阻塞地解决。代码实现如下，当然，这里的lua-resty-signal ，其实也是用 FFI 去调用系统函数来解决的。
```lua
local resty_signal = require "resty.signal"
local pid = 12345
local ok, err = resty_signal.kill(pid, "KILL")
```
在 LuaJIT 的官方网站上，专门有一个[页面](http://wiki.luajit.org/FFI-Bindings)，里面分门别类地介绍了各种 FFI 的绑定库。当你在处理图片、加解密等 CPU 密集运算的时候，可以先去里面看看，是否有已经封装好的库，可以拿来直接使用。

- **方案二：使用基于 ngx.pipe 的 lua-resty-shell 库**

可以在 shell.run 中运行你自己的命令，它就是一个非阻塞的操作：

```lua
$ resty -e 'local shell = require "resty.shell"
local ok, stdout, stderr, reason, status =
shell.run([[echo "hello, world"]])
ngx.say(stdout) '
```

## 1.2 磁盘 I/O
再来看下处理磁盘I/O的场景。在服务端程序中，读取本地的配置文件是一个很常见的操作，如下边的代码：
```lua
local path = "/conf/apisix.conf"
local file = io.open(path, "rb")
local content = file:read("*a")
file:close()
```
上边的`io.open` 就是一个阻塞的操作，但是也需要考虑实际场景，如果以上操作是在init 和 init worker中调用，那么它其实是一个一次性操作，并没有影响任何终端用户的请求，是完全可以被接受的。
但是如果每一个用户的请求，都会触发磁盘的读写，那就变得不可接受了。这时，你就需要认真地考虑解决方案了。

第一种方式，我们可以使用 lua-io-nginx-module 这个第三方的 C 模块。它为 OpenResty 提供了“非阻塞”的 Lua API，这种方式的原理是，lua-io-nginx-module 利用了 Nginx 的线程池，把磁盘 I/O 操作从主线程转移到另外一个线程中处理，这样，主线程就不会因为磁盘 I/O 操作而被阻塞。
使用这个库时，你需要重新编译 Nginx，因为它是一个 C 模块。它的使用方法如下，和 Lua 的 I/O 库基本是一致的：
```lua
local ngx_io = require "ngx.io"
local path = "/conf/apisix.conf"
local file, err = ngx_io.open(path, "rb")
local data, err = file: read("*a")
file:close()
```

第二种方式，则是尝试架构上的调整。对于这类磁盘 I/O，我们是否可以换种方式，不再读写本地磁盘呢？比如下边的记录日志操作：
```lua
ngx.log(ngx.WARN, "info")
```
调用这个API，即使有缓冲区，大量而频繁的磁盘写入，也会严重地影响性能。
要实现的需求是记录日志，我们可以考虑把日志发送到远端的日志服务器上，这样就可以用 cosocket 来完成非阻塞的网络通信了，也就是把阻塞的磁盘 I/O 丢给日志服务，不要阻塞对外的服务。你可以使用 `lua-resty-logger-socket `，来完成这样的工作：
```lua
local logger = require "resty.logger.socket"
if not logger.initted() then
local ok, err = logger.init{
host = 'xxx',
port = 1234,
flush_limit = 1234,
drop_limit = 5678,
}
local msg = "foo"
local bytes, err = logger.log(msg)
```

> 上面两个方法的本质都是一样的：如果阻塞不可避免，那就不要阻塞主要的工作线程，丢给外部的其他线程或者服务就可以了。

## 1.3 luasocket
luasocket 也是容易被开发者用到的一个 Lua 内置库，经常有人分不清 luasocket 和 OpenResty 提供的 cosocket。luasocket 也可以完成网络通信的功能，但它并没有非阻塞的优势。如果你使用了 luasocket，那么性能也会急剧下降。
前面讲过的cosocket在不少阶段无法使用，这时可以考虑使用luasocket。

[lua-resty-socket](https://github.com/thibaultcha/lua-resty-socket/) 是一个二次封装的开源库，它做到了 luasocket 和 cosocket 的兼容。

# 2 让人又恨又爱的字符串操作
## 2.1 性能优化背后
- **理念一：处理请求要短、平、快**

OpenResty 是一个 Web 服务器，所以经常会同时处理几千、几万甚至几十万的终端请求。想要在整体上达到最高性能，我们就一定要保证单个请求被快速地处理完成，并回收内存等各种资源。

* 这里提到的“短”，是指请求的生命周期要短，不要长时间占用资源而不释放；即使是长连接，也要设定一个时间或者请求次数的阈值，来定期地释放资源。
* 第二个字“平”，则是指在一个 API 中只做一件事情。要把复杂的业务逻辑拆散为多个 API，保持代码的简洁。
* 最后的“快”，是指不要阻塞主线程，不要有大量 CPU 运算。即使是不得不有这样的逻辑，也别忘了咱们上节课介绍的方法，要配合其他的服务去完成。

> 其实，这种架构上的考虑，不仅适合 OpenResty，在其他的开发语言和平台上也都是适用的，希望你能认真理解和思考。

- **理念二：避免产生中间数据**

避免中间的无用数据，可以说是 OpenResty 编程中最为主要的优化理念。这里，我先给你举一个小例子，来讲解下什么是中间的无用数据。我们来看下面这段代码：

```bash
$ resty -e 'local s= "hello"
s = s .. " world"
s = s .. "!"
print(s)
'
```
这段代码，我们对s 这个变量做了多次拼接操作，才得到了hello world! 对结果。但很显然，只有 s 的最终状态，也就是 hello world! 这个状态是有用的。而 s 的初始值和中间的赋值，都属于中间数据，应该尽量少生成。
因为这些临时数据，会带来初始化和 GC 的性能损耗。不要小看这些损耗，如果这出现在循环等热代码中，就会带来非常明显的性能下降了。

## 2.2 字符串是不可变的
**在LUA中，字符串是不可改变的**，在修改字符串的时候，并不是改变原有的字符串，而是会生成一个新的字符串对象，并改变了对字符串的引用。如果原有字符串没有其他的任何引用，就会给Lua的GC给回收掉。
字符串不可变的好处显而易见，那就是节省内存。这样一来，同样内容的字符串在内存中就只有一份了，不同的变量都会指向同一个内存地址。
至于这样设计的缺点，那就是涉及到字符串的新增和 GC 时，每当你新增一个字符串，LuaJIT 都得调用 lj_str_new，去查询这个字符串是否已经存在；没有的话，便需要再创建新的字符串。如果操作很频繁，自然就会对性能有非常大的影响。
我们来看一个具体的例子，类似这个例子中的字符串拼接操作，在很多 OpenResty 的开源项目中都会出现：
```bash
$ resty -e 'local begin = ngx.now()
local s = ""
-- for 循环，使用 .. 进行字符串拼接
for i = 1, 100000 do
	s = s .. "a"
end
ngx.update_time()
print(ngx.now() - begin)
'
```
这段示例代码的作用，是对s 变量做十万次字符串拼接，并把运行时间打印出来。虽然例子有些极端，但却能很好地体现出性能优化前后的差异。未经优化时，这段代码在我的笔记本上跑了 0.4 秒钟，还是比较慢的。那么应该如何优化呢？
使用table做一次封装，去掉所有临时的中间字符串，只保留原始数据和最终结果。

```bash
$ resty -e 'local begin = ngx.now()
local t = {}
-- for 循环，使用数组来保存字符串，每次都计算数组长度
for i = 1, 100000 do
	t[#t + 1] = "a"
end
-- 使用数组的 concat 方法拼接字符串
local s = table.concat(t, "")
ngx.update_time()
print(ngx.now() - begin)
'
```
用 table 依次保存了每一个字符串，下标由 `#t + 1` 来决定，也就是用 table 的当前长度加 1；最后，使用 table.concat 函数，把数组的每一个元素进行拼接，直接得到最终结果。这样自然就跳过了所有的临时字符串，避免了 10 万次 lj_str_new 和 GC。
刚刚是我们对于代码的分析，那么优化的具体效果如何呢？很明显，优化后的代码耗时只有 0.007 秒，也就是说，性能提升了五十多倍。

刚刚这段 0.007 秒的代码，是否就已经足够好了呢？其实不然，它还有继续优化的空间。我们不妨再来修改一行代码，然后来看下效果：
```bash
$ resty -e 'local begin = ngx.now()
local t = {}
-- for 循环，使用数组来保存字符串，自己维护数组的长度
for i = 1, 100000 do
	t[i] = "a"
end
local s = table.concat(t, "")
ngx.update_time()
print(ngx.now() - begin)
'
```
以上通过自己维护下标的操作，避免了计算十万此数组长度的函数调用。

## 2.3 减少其他临时字符串
在OpenResty中有些操作存在着一些更隐蔽的临时字符串的产生，它们就更不容易被发现。
`string.sub` 函数的作用是截取字符串的指定部分。正如我们前面所提到的，Lua 中的字符串是不可变的，那么截取出来的新字符串，就会涉及到 lj_str_new 和后续的 GC 操作。

```bash
resty -e 'print(string.sub("abcd", 1, 1))'
```
上面这段代码的作用，是获取字符串的第一个字符，并打印出来。自然，它不可避免会生成临时字符串。要完成同样的效果，还有别的更好的办法吗？

```bash
resty -e 'print(string.char(string.byte("abcd")))'
```
第二段代码，我们先用 string.byte 获取到第一个字符的数字编码，再用 string.char 把数字转为对应的字符。这个过程中并没有生成任何临时的字符串。因此，使用 string.byte 来完成字符串相关的扫描和分析，是效率最高的。

## 2.4 利用 SDK 对 table 类型的支持
下边是响应体内容输出到客户端的操作：

```bash
$ resty -e 'local begin = ngx.now()
local t = {}
local index = 1
for i = 1, 100000 do
	t[index] = "a"
	index = index + 1
end
local response = table.concat(t, "")
ngx.say(response)
'
```

在 ngx.say、ngx.print 、ngx.log、cosocket:send 等这些可能接受大量字符串的 API 中，它不仅接受 string 作为参数，也同时接受 table 作为参数：

```bash
resty -e 'local begin = ngx.now()
local t = {}
local index = 1
for i = 1, 100000 do
	t[index] = "a"
	index = index + 1
end
ngx.say(t)
'
```

在最后这段代码中，我们省略掉了 local response = table.concat(t, "")， 这个字符串拼接的步骤，直接把 table 传给了 ngx.say。这样，就把字符串拼接的任务，从 Lua 层面转移到了 C 层面，又避免了一次字符串的查找、生成和 GC。对于比较长的字符串而言，这又是一次不小的性能提升。
> OpenResty 的性能优化，很多都是在抠各种细节。所以，你需要对 LuaJIT 和 OpenResty 的 Lua API 了如指掌，才能达到最优的性能。

# 3 性能提升10倍的秘诀：必须用好 table
table 相关的优化，有一个自己的简单原则：**尽量复用，避免不必要的 table 创建。**

## 3.1 预先生成数组
使用LuaJIT 中的 table.new(narray, nhash) 函数预先创建table，这个函数，会预先分配好指定的数组和哈希的空间大小，而不是在插入元素时自增长，这也是它的两个参数 narray 和 nhash 的含义。
使用示例：

```lua
local new_tab = require "table.new"
local t = new_tab(100, 0)
for i = 1, 100 do
    t[i] = i
end
```

另外，因为之前的 OpenResty 并没有完全绑定 LuaJIT，还支持标准 Lua，所以有些旧的代码会做这方面的兼容。如果没有找到 table.new 这个函数，就会模拟出来一个空的函数，来保证调用方的统一。
```lua
local ok, new_tab = pcall(require, "table.new")
if not ok then
    new_tab = function (narr, nrec) return {} end
end
```

## 3.2 自己计算table下标
其实上边性能优化的时候也讲过这种思路，以下操作会经常计算table的长度，会影响性能

```lua
local new_tab = require "table.new"
local t = new_tab(100, 0)
for i = 1, 100 do
    t[#t + 1] = i
end
```
下边是lua-resty-redis官方库的代码：

```lua
local function _gen_req(args)
    local nargs = #args
    local req = new_tab(nargs * 5 + 1, 0)
    req[1] = "*" .. nargs .. "\r\n"
    local nbits = 2
    for i = 1, nargs do
        local arg = args[i]
        req[nbits] = "$"
        req[nbits + 1] = #arg
        req[nbits + 2] = "\r\n"
        req[nbits + 3] = arg
        req[nbits + 4] = "\r\n"
        nbits = nbits + 5
    end
    return req
end
```

这个函数预先生成了数组 req，它的大小由函数的入参来决定，这样就可以保证尽量不浪费空间。
然后，它使用 nbits 这个变量，来自己维护 req 的下标，自然就抛弃了 Lua 内置的 table.insert 函数和获取长度的操作符 `#`。你可以看到，在 for 循环中，nbits + 1 等一些运算，就是直接用下标的方式插入元素；并在最后用 `nbits = nbits + 5` ，让下标保持一个正确的值。
这种的好处很明显，它省略了获取数组大小这个 O(n) 的操作，而是直接用下标访问，时间复杂度也变成了 O(1) 。当然，缺点也一样明显，那就是降低了代码的可读性，并且出错概率大大提高，可以说，这是一把双刃剑。

## 3.3 循环使用单个table
就是通过使用table.clear清空table，然后复用之前的table

```lua
local ok, clear_tab = pcall(require, "table.clear")
if not ok then
    clear_tab = function (tab)
        for k, _ in pairs(tab) do
            tab[k] = nil
        end
    end
end
```

一般来说，我们会把这种循环使用的 table，放在一个模块的 top level 中。这样，在你使用模块中的函数的时候，就可以根据自己的实际情况来决定，到底是直接使用，还是 clear 后再使用。

## 3.4 tablepool池
可以用缓存池的方式来保存多个 table，以便随用随取，官方提供的 lua-tablepool 正是出于这个目的。
下面这段代码，展示了 table 池的基本使用方法。我们可以从指定的池子中获取一个 table，使用完以后再释放回去：

```lua
local tablepool = require "tablepool"
local tablepool_fetch = tablepool.fetch
local tablepool_release = tablepool.release

local pool_name = "some_tag"
local function do_sth()
    local t = tablepool_fetch(pool_name, 10, 0)
    -- -- using t for some purposes
    tablepool_release(pool_name, t)
end
```

第一个是 fetch 方法，它的参数和 table.new 基本一样，只是多了一个 pool_name。如果池子中没有空闲的数组，fetch 方法就会调用 table.new 来新建一个数组。

```lua
tablepool.fetch(pool_name, narr, nrec)
```

第二个是 release 这个把 table 放回池子的函数。在它的参数中，最后的 no_clear ，用来配置是否要调用 table.clear 把数组清空。
```lua
tablepool.release(pool_name, tb, [no_clear])
```


# 4 高性能的关键：shareddict缓存和lru缓存
## 4.1 缓存
一般来说，缓存有两个原则。

- 一是越靠近用户的请求越好。比如，能用本地缓存的就不要发送 HTTP 请求，能用 CDN 缓存的就不要打到源站，能用 OpenResty 缓存的就不要打到数据库。
- 二是尽量使用本进程和本机的缓存解决。因为跨了进程和机器甚至机房，缓存的网络开销就会非常大，这一点在高并发的时候会非常明显。

**OpenResty 中有两个缓存的组件：shared dict 缓存和 lru 缓存。**
- shared dict缓存：只能缓存字符串对象，缓存的数据有且只有一份，每一个 worker 都可以进行访问，所以常用于 worker 之间的数据通信。
- lru缓存：可以缓存所有的 Lua 对象，但只能在单个 worker 进程内访问，有多少个 worker，就会有多少份缓存数据。

**下边的两个表格，可以说明 shared dict 和 lru 缓存的区别：**
![](https://raw.githubusercontent.com/BaihlUp/Figurebed/master/2023/202311171113687.png)
![](https://raw.githubusercontent.com/BaihlUp/Figurebed/master/2023/202311171112644.png)

shared dict 和 lru 缓存，并没有哪一个更好的说法，而是应该根据你的场景来配合使用。

- 如果你没有 worker 之间共享数据的需求，那么 lru 可以缓存数组、函数等复杂的数据类型，并且性能最高，自然是首选。
- 但如果你需要在 worker 之间共享数据，那就可以在 lru 缓存的基础上，加上 shared dict 的缓存，构成两级缓存的架构。

## 4.2 共享字典缓存
使用方法

```shell
$ resty --shdict='dogs 1m' -e 'local dict = ngx.shared.dogs
        dict:set("Tom", 56)
        print(dict:get("Tom"))'
```
你需要事先在 Nginx 的配置文件中，声明一个内存区 dogs，然后在 Lua 代码中才可以使用。如果你在使用的过程中，发现给 dogs 分配的空间不够用，那么是需要先修改 Nginx 配置文件，然后重新加载 Nginx 才能生效的。因为我们并不能在运行时进行扩容和缩容。

- **缓存数据的序列化**

第一个问题，缓存数据的序列化。由于共享字典中只能缓存字符串对象，所以，如果你想要缓存数组，就少不了要在 set 的时候要做一次序列化，在 get 的时候做一次反序列化：

```shell
resty --shdict='dogs 1m' -e 'local dict = ngx.shared.dogs
        dict:set("Tom", require("cjson").encode({a=111}))
        print(require("cjson").decode(dict:get("Tom")).a)'
```
不过，这类序列化和反序列化操作是非常消耗 CPU 资源的。如果每个请求都有那么几次这种操作，那么，在火焰图上你就能很明显地看到它们的消耗。

大部分的序列化都是可以在业务层面进行拆解的。你可以把数组的内容打散，分别用字符串的形式存储在共享字典中。如果还不行的话，那么也可以把数组缓存在 lru 中，用内存空间来换取程序的便捷性和性能。
此外，缓存中的 key 也应该尽量选择短和有意义的，这样不仅可以节省空间，也方便后续的调试。

- **stale数据**

共享字典中还有一个 get_stale 的读取数据的方法，相比 get 方法，多了一个过期数据的返回值：

```
resty --shdict='dogs 1m' -e 'local dict = ngx.shared.dogs
            dict:set("Tom", 56, 0.01)
            ngx.sleep(0.02)
            local val, flags, stale = dict:get_stale("Tom")
            print(val)'
```
在上面的这个示例中，数据只在共享字典中缓存了 0.01 秒，在 set 后的 0.02 秒后，数据就已经超时了。这时候，通过 get 接口就不会获取到数据了，但通过 `get_stale` 还可能获取到过期的数据。这里我之所以用“可能”两个字，是因为过期数据所占用的空间，是有一定几率被回收，再给其他数据使用的，这也就是 LRU 算法。

举个例子，数据源存储在 MySQL 中，我们从 MySQL 中获取到数据后，在 shared dict 中设置了 5 秒超时，那么，当这个数据过期后，我们就会有两个选择：
- 当这个数据不存在时，重新去 MySQL 中再查询一次，把结果放到缓存中；
- 判断 MySQL 的数据是否发生了变化，如果没有变化，就把缓存中过期的数据读取出来，修改它的过期时间，让它继续生效。

很明显，后者是更优化的方案，这样可以尽可能少地去和 MySQL 交互，让终端的请求都从最快的缓存中获取数据。
那怎么判断数据是否发生变化呢，下边以lru缓存为例，看下怎么解决这个问题。

## 4.3 lru缓存
lru 缓存的接口只有 5 个：new、set、get、delete 和 flush_all。和上面问题相关的就只有 get 接口，让我们先来了解下这个接口是如何使用的：

```shell
resty -e 'local lrucache = require "resty.lrucache"
        local cache, err = lrucache.new(200)
        cache:set("dog", 32, 0.01)
        ngx.sleep(0.02)
        local data, stale_data = cache:get("dog")
        print(stale_data)'
```
你可以看到，在 lru 缓存中， get 接口的第二个返回值直接就是 stale_data，而不是像 shared dict 那样分为了 get 和 get_stale 两个不同的 API。这样的接口封装，对于使用过期数据来说显然更加友好。
可以对数据添加版本号的概念，有了版本号后，我们就可以对 lru 缓存做一个简单的二次封装，比如来看下面的伪码，摘自apisix的lrucache.lua

```lua
local function (key, version, create_obj_fun, ...)
    local obj, stale_obj = lru_obj:get(key)
    -- 如果数据没有过期，并且版本没有变化，就直接返回缓存数据
    if obj and obj._cache_ver == version then
      return obj
    end
    -- 如果数据已经过期，但还能获取到，并且版本没有变化，就直接返回缓存中的过期数据
    if stale_obj and stale_obj._cache_ver == version then
        lru_obj:set(key, obj, item_ttl)
        return stale_obj
    end
    -- 如果找不到过期数据，或者版本号有变化，就从数据源获取数据
    local obj, err = create_obj_fun(...)
    obj._cache_ver = version
    lru_obj:set(key, obj, item_ttl)
    return obj, err
end
```
从这段代码中你可以看到，我们通过引入版本号的概念，在版本号没有变化的情况下，充分利用了过期数据来减少对数据源的压力，达到了性能的最优。

除此之外，在上面的方案中，其实还有一个潜在的很大优化点，那就是我们把 key 和版本号做了分离，把版本号作为 value 的一个属性。
我们知道，更常规的做法是把版本号写入 key 中。比如 key 的值是 key_1234，这种做法非常普遍，但在 OpenResty 的环境下，这样其实是存在浪费的。为什么这么说呢？
举个例子你就明白了。假如版本号每分钟变化一次，那么key_1234 过一分钟就变为了 key_1235，一个小时就会重新生成 60 个不同的 key，以及 60 个 value。这也就意味着， Lua GC 需要回收 59 个键值对背后的 Lua 对象。如果你的更新更加频繁，那么对象的新建和 GC 显然会消耗更多的资源。
当然，这些消耗也可以很简单地避免，那就是把版本号从 key 挪到 value 中。这样，一个 key 不管更新地多么频繁，也只有固定的两个 Lua 对象。可以看出，这样的优化技巧非常巧妙，不过，简单巧妙的技巧背后，其实需要你对 OpenResty 的 API 以及缓存的机制都有很深入的了解才可以。

## 4.4 缓存风暴
### 4.4.1 什么是缓存风暴？
下边这个场景：
数据源在 MySQL 数据库中，缓存的数据放在共享字典中，超时时间为 60 秒。在这 60 秒内的时间里，所有的请求都从缓存中获取数据，MySQL 没有任何的压力。但是，一旦到达 60 秒，也就是缓存数据失效的那一刻，如果正好有大量的并发请求进来，在缓存中没有查询到结果，就要触发查询数据源的函数，那么这些请求全部都将去查询 MySQL 数据库，直接造成数据库服务器卡顿，甚至卡死。

下边是一个有缓存风暴隐患的伪代码：

```lua
local value = get_from_cache(key)
if not value then
    value = query_db(sql)
    set_to_cache(value, timeout ＝ 60)
end
return value
```
这段伪代码看上去逻辑都是正常的，你使用单元测试或者端对端测试，都不会触发缓存风暴。只有长时间的压力测试才会发现这个问题，每隔 60 秒的时间，数据库就会出现一次查询的峰值，非常有规律。不过，如果你这里的缓存失效时间设置得比较长，那么缓存风暴问题被发现的几率就会降低。

### 4.4.2 如何避免缓存风暴？
#### 4.4.2.1 主动更新缓存

上边的示例是被动更新缓存，只有在终端请求发现缓存失效时，才会去数据库查询新的数据，如果从被动改为主动，也就可以直接绕开缓存风暴得问题了。

在 OpenResty 中，我们可以这样来实现。首先，使用 ngx.timer.every 来创建一个定时任务，每分钟运行一次，去 MySQL 数据库中获取最新的数据，并放入共享字典中:

```
local function query_db(premature, sql)
    local value = query_db(sql)
    set_to_cache(value, timeout ＝ 60)
end
local ok, err = ngx.timer.every(60, query_db, sql)
```
然后，在终端请求的代码逻辑中，去掉查询 MySQL 的部分，只保留获取共享字典缓存的代码：

```
local value = get_from_cache(key)
return value
```
通过这样两段伪码的操作，缓存风暴的问题就被绕过去了。但这种方式也并非完美，因为这样的每一个缓存都要对应一个周期性的任务（OpenResty 中 timer 是有上限的，不能太多）；而且缓存过期时间和计划任务的周期时间还要对应好，如果这中间出现了什么纰漏，终端就可能一直获取到的都是空数据。

在实际项目中一般都是使用加锁的方式来解决缓存风暴问题。

#### 4.4.2.2 lua-resty-lock

使用 OpenResty 中的 lua-resty-lock 这个库来加锁，lua-resty-lock 是 OpenResty 自带的 resty 库，它底层是基于共享字典，提供非阻塞的 lock API。我们先来看一个简单的示例：

```
resty --shdict='locks 1m' -e 'local resty_lock = require "resty.lock"
            local lock, err = resty_lock:new("locks")
            local elapsed, err = lock:lock("my_key")
            -- query db and update cache
            local ok, err = lock:unlock()
            ngx.say("unlock: ", ok)'
```
因为 lua-resty-lock 是基于共享字典来实现的，所以我们需要事先声明 shdict 的名字和大小；然后，再使用 new 方法来新建 lock 对象。你可以看到，这段代码中，我们只传了第一个参数 shdict 的名字。其实， new 方法还有第二个参数，可以用来指定锁的过期时间、等待锁的超时时间等多个参数。不过这里，我们使用的是默认值，它们就是用来避免死锁等各种异常问题的。
接着，我们就可以调用 lock 方法尝试获取锁。如果成功获取到锁的话，那就可以保证只有一个请求去数据源更新数据；而如果因为锁已经被抢占、超时等导致加锁失败，那就需要从陈旧的缓存中获取数据，返回给终端。这个过程是不是听起来很熟悉？没错，这里就正好用到了我们上节课介绍过的的 get_stale API：

```
local elapsed, err = lock:lock("my_key")
# elapsed 为 nil 表示加锁失败，err 的返回值是 timeout、 locked 中的一个
if not elapsed and err then
    dict:get_stale("my_key")
end
```
如果 lock 成功，那么就可以安全地去查询数据库，并把结果更新到缓存中。最后，我们再调用 unlock 接口，把锁释放掉就可以了。
[lua-resty-lock](https://github.com/openresty/lua-resty-lock#for-cache-locks)
每当遇到一些有趣的实现，我们总是希望能够看看它的源码是如何实现的，这也是开源的好处之一。这里，我们再深入一步，看看 lock 这个接口是如何加锁的，下面便是它的源码：

```
local ok, err = dict:add(key, true, exptime)
if ok then
    cdata.key_id = ref_obj(key)
    self.key = key
    return 0
end
```
在共享字典章节中，我曾经提到过，shared dict 的所有 API 都是原子操作，不用担心出现竞争，所以用 shared dict 来标记锁的状态是个不错的主意。

这里 lock 接口的实现，便使用了 dict:add 接口来尝试设置 key。如果 key 在共享内存中不存在，add 接口就会返回成功，表示加锁成功；其他并发的请求走到 dict:add 这一行的代码逻辑时，就会返回失败，然后根据返回的 err 信息，选择是直接返回，还是多次重试。

#### 4.4.2.3 lua-resty-shcache
在上面 lua-resty-lock 的实现中，你需要自己来处理加锁、解锁、获取过期数据、重试、异常处理等各种问题，还是相当繁琐的。所以，这里我再给你介绍一个简单的封装：lua-resty-shcache。
lua-resty-shcache是 CloudFlare 开源的一个 lua-resty 库，它在共享字典和外部存储之上，做了一层封装；并且额外提供了序列化和反序列化的接口，让你不用去关心上述的各种细节:

```
local shcache = require("shcache")
local my_cache_table = shcache:new(
        ngx.shared.cache_dict
        { external_lookup = lookup,
            encode = cmsgpack.pack,
            decode = cmsgpack.decode,
        },
        { positive_ttl = 10, -- cache good data for 10s
            negative_ttl = 3, -- cache failed lookup for 3s
            name = 'my_cache', -- "named" cache, useful for debug / report
        }
    )
local my_table, from_cache = my_cache_table:load(key)
```

#### 4.4.2.4 Nginx配置指令
即使你没有使用 OpenResty 的 lua-resty 库，你也可以用 Nginx 的配置指令，来实现加锁和获取过期数据——即proxy_cache_lock 和 proxy_cache_use_stale。不过，这里我并不推荐使用 Nginx 指令这种方式，它显然不够灵活，性能也比不上 Lua 代码。


## 4.5 lua-resty-* 封装，让你远离多级缓存之痛
### 4.5.1 lua-resty-memcached-shdict
让我们回到缓存的封装上来。lua-resty-memcached-shdict 是 OpenResty 官方的一个项目，它使用 shared dict 为 memcached 做了一层封装，处理了缓存风暴和过期数据等细节。如果你的缓存数据正好存储在后端的 memcached 中，那么你可以尝试使用这个库。
如果你想在本地测试，需要先把它的[源码](https://github.com/openresty/lua-resty-memcached-shdict)下载到本地 OpenResty 的查找路径下。

这个封装库，其实和我们上节课中提到的解决方案是一样的。它使用 lua-resty-lock 来做到互斥，在缓存失效的情况下，只有一个请求去 memcached 中获取数据，避免缓存风暴。如果没有获取到最新数据，则使用 stale 数据返回给终端。

在这个封装库的文档中，其实也提到了进一步的优化方向：
- 一是使用 lua-resty-lrucache ，来增加 worker 层的缓存，而不仅仅是 server 级别的 shared dict 缓存；
- 二是使用 ngx.timer ，来做异步的缓存更新操作。

### 4.5.2 lua-resty-mlcache
OpenResty 中被普遍使用的缓存封装： lua-resty-mlcache。它使用 shared dict 和 lua-resty-lrucache ，实现了多层缓存机制。我们下面就通过两段代码示例，来看看这个库如何使用：

```lua
local mlcache = require "resty.mlcache"
local cache, err = mlcache.new("cache_name", "cache_dict", {
    lru_size = 500, -- size of the L1 (Lua VM) cache
    ttl = 3600, -- 1h ttl for hits
    neg_ttl = 30, -- 30s ttl for misses
})
if not cache then
    error("failed to create mlcache: " .. err)
end
```
先来看第一段代码。这段代码的开头引入了 mlcache 库，并设置了初始化的参数。我们一般会把这段代码放到 init 阶段，只需要做一次就可以了。

除了缓冲名和字典名这两个必填的参数外，第三个参数是一个字典，里面 12 个选项都是选填的，不填的话就使用默认值。这种方式显然就比 lua-resty-memcached-shdict 要优雅很多。其实，我们自己来设计接口的话，也最好采用 mlcache 这样的做法——让接口尽可能地简单，同时还保留足够的灵活性。

下面再来看第二段代码，这是请求处理时的逻辑代码：

```lua
local function fetch_user(id)
    return db:query_user(id)
end
local id = 123
local user , err = cache:get(id , nil , fetch_user , id)
if err then
    ngx.log(ngx.ERR , "failed to fetch user: ", err)
    return
end
if user then
    print(user.id) -- 123
end
```

你可以看到，这里已经把多层缓存都给隐藏了，你只需要使用 mlcache 的对象去获取缓存，并同时设置好缓存失效后的回调函数就可以了。这背后复杂的逻辑，就可以被完全地隐藏了。
说到这里，你可能好奇，这个库内部究竟是怎么实现的呢？接下来，再让我们来看下这个库的架构和实现。下面这张图，来自 mlcache 的作者 thibault 在 2018 年 OpenResty 大会上演讲的幻灯片：
![](https://raw.githubusercontent.com/BaihlUp/Figurebed/master/2023/202311171110340.png)
从图中你可以看到，mlcache 把数据分为了三层，即 L1、L2 和 L3。

L1 缓存就是 lua-resty-lrucache。每一个 worker 中都有自己独立的一份，有 N 个 worker，就会有 N 份数据，自然也就存在数据冗余。由于在单 worker 内操作 lrucache 不会触发锁，所以它的性能更高，适合作为第一级缓存。
L2 缓存是 shared dict。所有的 worker 共用一份缓存数据，在 L1 缓存没有命中的情况下，就会来查询 L2 缓存。ngx.shared.DICT 提供的 API，使用了自旋锁来保证操作的原子性，所以这里我们并不用担心竞争的问题；
L3 则是在 L2 缓存也没有命中的情况下，需要执行回调函数去外部数据库等数据源查询后，再缓存到 L2 中。在这里，为了避免缓存风暴，它会使用 lua-resty-lock ，来保证只有一个 worker 去数据源获取数据。
整体而言，从请求的角度来看，
- 首先会去查询 worker 内的 L1 缓存，如果 L1 命中就直接返回。如果 L1 没有命中或者缓存失效，就会去查询 worker 间的 L2 缓存。
- 如果 L2 命中就返回，并把结果缓存到 L1 中。
- 如果 L2 也没有命中或者缓存失效，就会调用回调函数，从数据源中查到数据，并写入到 L2 缓存中，这也就是 L3 数据层的功能。

从这个过程你也可以看出，缓存的更新是由终端请求来被动触发的。即使某个请求获取缓存失败了，后续的请求依然可以触发更新的逻辑，以便最大程度地保证缓存的安全性。
不过，虽然 mlcache 已经实现得比较完美了，但在现实使用中，其实还有一个痛点——数据的序列化和反序列化。这个其实并不是 mlcache 的问题，而是我们之前反复提到的 lrucache 和 shared dict 之间的差异造成的。在 lrucache 中，我们可以存储 Lua 的各种数据类型，包括 table；但 shared dict 中，我们只能存储字符串。
L1 也就是 lrucache 缓存，是用户真正接触到的那一层数据，我们自然希望在其中可以缓存各种数据，包括字符串、table、cdata 等。可是，问题在于， L2 中只能存储字符串。那么，当数据从 L2 提升到 L1 的时候，我们就需要做一层转换，也就是从字符串转成我们可以直接给用户的数据类型。
还好，mlcache 已经考虑到了这种情况，并在 new 和 get 接口中，提供了可选的函数 l1_serializer，专门用于处理 L2 提升到 L1 时对数据的处理。我们可以来看下面的示例代码，它是我从测试案例集中摘选出来的：

```lua
local mlcache = require "resty.mlcache"
local cache, err = mlcache.new("my_mlcache", "cache_shm", {
l1_serializer = function(i)
    return i + 2
end,
})
local function callback()
    return 123456
end
local data = assert(cache:get("number", nil, callback))
assert(data == 123458)
```
简单解释一下。在这个案例中，回调函数返回数字 123456；而在 new 中，我们设置的 l1_serializer 函数会在设置 L1 缓存前，把传入的数字加 2，也就是变成 123458。通过这样的序列化函数，数据在 L1 和 L2 之间转换的时候，就可以更加灵活了。

可以说，mlcache 是一个功能很强大的缓存库，而且[文档](https://github.com/thibaultcha/lua-resty-mlcache)也写得非常详尽。今天这节课我只提到了核心的一些原理，更多的使用方法，建议你一定要自己花时间去探索和实践。

# 5 应对突发流量：漏桶和令牌桶的概念
## 5.1 流量控制
我们需要从成本、用户体验、系统稳定性等多个方面来综合考虑。不管使用哪一种算法，都不可避免地会造成正常用户请求变慢甚至被拒绝，牺牲部分的用户体验。所以，流量控制是需要在业务稳定和用户体验之间做平衡的。

举个例子，比如我们假定一个上游服务的设计上限是每分钟处理 1 万条请求。在高峰期的时候，如果入口处没有限流的控制，每分钟堆积的任务达到了 2 万条，那么这个上游服务的处理性能就会下降，可能只有每分钟 5000 条的处理速度，并且持续恶化，最终或许会导致服务不可用。这显然不是我们希望看到的结果。
应对这种突发的流量，我们常用的流量控制算法，便是**漏桶**和**令牌桶**。

## 5.2 漏桶算法
它的目的是让请求的速率保持恒定，把突发的流量变得平滑。
![](https://raw.githubusercontent.com/BaihlUp/Figurebed/master/2023/202311171111956.png)

我们可以把客户端的流量想象成是从水管中流出来的水，水的流速不确定，忽快忽慢；而外层的流量处理模块，就是接水的桶子，并且这个水桶的底部有一个漏水用的洞眼。这其实也就是漏桶算法名字的由来，很明显，这种算法有下面几个好处。

* 第一，不管流入水桶的是涓涓细流还是滔天洪水，都可以保证，水桶中流出来的水速是恒定的。这种稳定的流量对于上游服务是很友好的，这也是流量整形的意义。
* 第二，水桶本身有一定容积，可以积累一定的水来等待流出水桶。这对于终端的请求来说，相当于是如果不能被立即处理，可以排队等待。
* 第三，超过水桶容积的水，不会被水桶接纳，而是会直接流走。这里对应的是，终端的请求如果太多，超过了排队的长度，就直接返回给客户端失败信息。这时候的服务端已经处理不过来了，自然，请求连排队的必要也就没有了。

算法实现，OpenResty自带的[resty.limit.req](https://github.com/openresty/lua-resty-limit-traffic/blob/master/lib/resty/limit/req.lua#L73)为例来看，它就是按照漏桶算法实现的限速模块

```lua
local elapsed = now - tonumber(rec.last)
excess = max(tonumber(rec.excess) - rate * abs(elapsed) / 1000 + 1000,0)
if excess > self.burst then
    return nil, "rejected"
end
-- return the delay in seconds, as well as excess
return excess / rate, excess / 1000
```
简单解释一下这几行代码。其中， elapsed 是当前请求和上一次请求之间的毫秒数，rate 则是我们设定的每秒的速率。因为rate的最小单位是 0.001 s/r，所以在上述实现的代码中，都需要乘以 1000 以便计算。
excess 表示还在排队的请求数量，它为 0 表示水桶是空的，没有请求在排队，而burst 是指整个水桶的容积。如果 excess 已经大于 burst，也就意味着水桶已经满了，这时候再进来的流量就会被直接丢弃；如果 excess 大于 0 、小于 burst，就进入了排队来等待处理，这里最后返回的 excess / rate ，也就是要等待的时间。
这样，在后端服务处理能力不变的情况下，我们就可以通过调节 burst 的大小，来控制突发流量的排队时长了。是直接告诉用户现在请求量太大，稍后再重试，还是让用户多等待一段时间，这就要看你的业务场景了。

## 5.3 令牌桶算法
令牌桶算法与漏桶算法的实现不同，漏桶算法中，我们一般会使用终端 IP 作为 key ，来做限流限速的依据。这样，对于每一个终端用户而言，漏桶算法的出口速率就是固定的。不过，这就会存在一个问题：

> 如果 A 用户的请求频率很高，而其他用户的请求频率很低，即使此时的整体服务压力并不大，但漏桶算法就会把 A 的部分请求变慢或者拒绝掉，虽然这时候服务其实是可以处理的。

漏桶算法关注的是流量的平滑，而令牌桶则可以允许突发流量进入后端服务。令牌桶的原理，是以一个固定的速度向水桶内放入令牌，只要桶没有满就一直往里面放。这样，终端过来的请求都需要先到令牌桶中获取到令牌，才可以被后端处理；如果桶里面没有令牌，那么请求就会被拒绝。

OpenResty 自带的限流限速的库中没有实现令牌桶，所以，这里我用又拍云开源的、基于令牌桶的限速模块 lua-resty-limit-rate 的[代码](https://github.com/upyun/lua-resty-limit-rate)为例，为你做一个简单的介绍：

```lua
local limit_rate = require "resty.limit.rate"
 
-- global 20r/s 6000r/5m
local lim_global = limit_rate.new("my_limit_rate_store", 100, 6000, 2)
 
-- single 2r/s 600r/5m
local lim_single = limit_rate.new("my_limit_rate_store", 500, 600, 1)
 
local t0, err = lim_global:take_available("__global__", 1)
local t1, err = lim_single:take_available(ngx.var.arg_userid, 1)
 
if t0 == 1 then
    return -- global bucket is not hungry
else
    if t1 == 1 then
        return -- single bucket is not hungry
    else
        return ngx.exit(503)
    end
end
```
在这段代码中，我们设置了两个令牌桶：一个是全局的令牌桶，一个是以 b ngx.var.arg_userid 为 key，按照用户来划分的令牌桶。这里用两个令牌桶做了一个组合，主要有这么一个好处：

* 在全局令牌桶还有令牌的情况下，不用去判断用户的令牌桶，如果后端服务能够正常运行，就尽可能多地去服务用户的突发请求；
* 在全局令牌桶没有令牌的情况下，不能无差别地拒绝请求，这时候就需要判断下单个用户的令牌桶，把突发请求比较多的用户请求给拒绝掉。这样一来，就可以保证其他用户的请求不会受到影响。

显然，令牌桶和漏桶相比，更具有弹性，允许出现突发流量传递到后端服务的情况。当然，它们都各有利弊，你可以根据自己的情况来选择使用。

## 5.4 Nginx的限速模块
说完这两个算法，我们最后再来看下，在熟悉的 Nginx 中是如何来实现限流限速的。在 Nginx 中，[limit_req 模块](http://nginx.org/en/docs/http/ngx_http_limit_req_module.html)是最常用的限速模块，下面是一个简单的配置：
```
limit_req_zone $binary_remote_addr zone=one:10m rate=1r/s;
 
server {
    location /search/ {
        limit_req zone=one burst=5;
    }
}
```
这段代码是把终端的 IP 地址作为 key，申请了一块名为 one 的 10M 的内存空间地址，并把速率限制为每秒 1 个请求。
在 server 的 location 中，还引用了 one 这个限速规则，并把 brust 设置为 5。这就表示在超过速率 1r/s 的情况下，同时允许有 5 个请求排队等待被处理，给出了一定的缓存区。要注意，如果没有设置 brust ，超过速率的请求是会被直接拒绝的。
Nginx 的这个模块是基于漏桶来实现的，所以和我们上面介绍过的 OpenResty 中的 resty.limit.req ，本质都是一样的。

## 5.5 动态限流限速
在 OpenResty 中，我们推荐使用 [lua-resty-limit-traffic](https://github.com/openresty/lua-resty-limit-traffic) 来做流量的限制。它里面包含了 limit-req（限制请求速率）、 limit-count（限制请求数） 和 limit-conn （限制并发连接数）这三种不同的限制方式；并且提供了limit.traffic ，可以把这三种方式进行聚合使用。

### 5.5.1 限制请求速率
让我们先来看下 limit-req，它使用的是漏桶算法来限制请求的速率。
示例代码：
```bash
resty --shdict='my_limit_req_store 100m' -e 'local limit_req = require "resty.limit.req"
local lim, err = limit_req.new("my_limit_req_store", 200, 100)
local delay, err = lim:incoming("key", true)
if not delay then
    if err == "rejected" then
        return ngx.exit(503)
    end
    return ngx.exit(500)
end
 
 if delay >= 0.001 then
    ngx.sleep(delay)
end'
```
我们知道，lua-resty-limit-traffic 是使用共享字典来对 key 进行保存和计数的，所以在使用 limit-req 前，我们需要先声明 my_limit_req_store 这个 100m 的空间。这一点对于 limit-conn 和 limit-count 也是类似的，它们都需要自己单独的共享字典空间，以便区分开。

```lua
limit_req.new("my_limit_req_store", 200, 100)
```
上面这行代码，便是其中最关键的一行代码。它的含义，是使用名为 my_limit_req_store 的共享字典来存放统计数据，并把每秒的速率设置为 200。这样，如果超过 200 但小于 300（这个值是 200 + 100 计算得到的） 的话，就需要排队等候；如果超过 300 的话，就会直接拒绝。
在设置完成后，我们就要对终端的请求进行处理了，lim: incoming("key", true) 就是来做这件事情的。incoming这个函数有两个参数，我们需要详细解读一下。
第一个参数，是用户指定的限速的 key。在上面的示例中它是一个字符串常量，这就意味着要对所有终端都统一限速。如果要实现根据不同省份和渠道来限速，其实也很简单，把这两个信息都作为 key 即可，下面是实现这一需求的伪代码：
```lua
local  province = get_ province(ngx.var.binary_remote_addr)
local channel = ngx.req.get_headers()["channel"]
local key = province .. channel
lim:incoming(key, true)
```
当然，你也可以举一反三，自定义 key 的含义以及调用 incoming 的条件，这样你就能收到非常灵活的限流限速效果了。
我们再来看incoming 函数的第二个参数，它是一个布尔值，默认是 false，意味着这个请求不会被记录到共享字典中做统计，这只是一次 演习。如果设置为 true，就会产生实际的效果了。因此，在大多数情况下，你都需要显式地把它设置为 true。
你可能会纳闷儿，为什么会有这个参数的存在呢？我们不妨考虑一下这样的一个场景，你设置了两个不同的 limit-req 实例，针对不同的 key，一个 key 是主机名，另外一个 key 是客户端的 IP 地址。那么，当一个终端请求被处理的时候，会按照先后顺序调用这两个实例的 incoming 方法，就像下面这段伪码表示的一样：
```lua
local limiter_one, err = limit_req.new("my_limit_req_store", 200, 100)
local limiter_two, err = limit_req.new("my_limit_req_store", 20, 10)
limiter_one :incoming(ngx.var.host, true)
limiter_two:incoming(ngx.var.binary_remote_addr, true)
```
如果用户的请求通过了 limiter_one 的阈值检测，但被 limiter_two 的检测拒绝，那么 limiter_one:incoming 这次函数调用就应该被认为是一次 演习，不应该真的去计数。
这样一来，上述的代码逻辑就不够严谨了。我们需要事先对所有的 limiter 做一次演习，如果有 limiter 的阈值被触发，可以 rejected 终端请求，就可以直接返回：
```lua
for i = 1, n do
    local lim = limiters[i]
    local delay, err = lim:incoming(keys[i], i == n)
    if not delay then
        return nil, err
    end
end
```
这其实就是 incoming 函数第二个参数的意义所在。刚刚这段代码就是 limit.traffic 模块最核心的一段代码，专门用作多个限流器的组合所用。

### 5.5.2 限制请求数
limit.count 这个限制请求数的库，它的效果和 GitHub API 的 Rate Limiting 一样，可以限制固定时间窗口内有多少次用户请求。老规矩，我们先来看一段示例代码：

```lua
local limit_count = require "resty.limit.count"
local lim, err = limit_count.new("my_limit_count_store", 5000, 3600)
local key = ngx.req.get_headers()["Authorization"]
local delay, remaining = lim:incoming(key, true)
```
limit.count 和 limit.req 的使用方法是类似的，我们先在 Nginx.conf 中定义一个字典：

```
lua_shared_dict my_limit_count_store 100m;
```
然后 new 一个 limiter 对象，最后用 incoming 函数来判断和处理。
不过，不同的是，limit-count 中的incoming 函数的第二个返回值，代表着还剩余的调用次数，我们可以据此在响应头中增加字段，给终端更好的提示：

```
ngx.header["X-RateLimit-Limit"] = "5000"
ngx.header["X-RateLimit-Remaining"] = remaining
```
### 5.5.3 限制并发连接数
前面所讲的限制请求速率和限制请求数，都是可以直接在 access 这一个阶段内完成的。而限制并发连接数则不同，它不仅需要在 access 阶段判断是否超过阈值，而且需要在 log 阶段调用 leaving 接口：
```
log_by_lua_block {
    local latency = tonumber(ngx.var.request_time) - ctx.limit_conn_delay
    local key = ctx.limit_conn_key
 
    local conn, err = lim:leaving(key, latency)
}
```
不过，这个接口的核心代码其实也很简单，也就是下面这一行代码，实际上就是把连接数减一的操作。如果你没有在 log 阶段做这个清理的动作，那么连接数就会一直上涨，很快就会达到并发的阈值。

```
local conn, err = dict:incr(key, -1)
```
### 5.5.4 限速器组合
到这里，这三种方式我们就分别介绍完了。最后，我们再来看看，怎么把 limit.rate、limit.conn 和 limit.count 组合起来使用。这就需要用到 limit.traffic 中的 combine 函数了：

```lua
local lim1, err = limit_req.new("my_req_store", 300, 200)
local lim2, err = limit_req.new("my_req_store", 200, 100)
local lim3, err = limit_conn.new("my_conn_store", 1000, 1000, 0.5)
local limiters = {lim1, lim2, lim3}
local host = ngx.var.host
local client = ngx.var.binary_remote_addr
local keys = {host, client, client}
local delay, err = limit_traffic.combine(limiters, keys, states)
```
combine 函数的核心代码，在我们上面分析 limit.rate 的时候已经提到了一部分，它主要是借助了演习功能和 uncommit 函数来实现。这样组合以后，你就可以为多个限流器设置不同的阈值和 key，实现更复杂的业务需求了。

> limit.traffic 不仅支持今天所讲的这三种限速器，实际上，只要某个限速器有 incoming 和 uncommit 接口，都可以被 limit.traffic 的 combine 函数管理。

# 6 OpenResty的杀手锏：动态
## 6.1 动态加载代码
在OpenResty中动态加载lua代码
```bash
resty -e 'local s = [[ngx.say("hello world")]]
local func, err = loadstring(s)
func()'
```
只要短短的两三行代码，就可以把一个字符串变为一个 Lua 函数，并运行起来。我们进一步仔细看下这几行代码，我来简单解读一下：

* 首先，我们声明了一个字符串，它的内容是一段合法的 Lua 代码，把 hello world 打印出来；
* 然后，使用 Lua 中的 loadstring 函数，把字符串对象转为函数对象func；
* 最后，在函数名的后面加上括号，把 func 执行起来，打印出 hello world 来。

## 6.2 FaaS（函数即服务）
通过loadstring方式加载函数
```bash
resty -e 'local s = [[
 return function()
    ngx.say("hello world")
end
]]
local  func1 = loadstring(s)
local ret, func = pcall(func1)
func()'
```
更深入一步，我们还可以把 s 这个包含函数的字符串，改成可以由用户指定的形式，并加上执行它的条件，这样其实就是 FaaS 的原型了。

## 6.3 边缘计算
OpenResty 的动态不仅可以用于 FaaS，让脚本语言的动态细化到函数级别，还可以在边缘计算上发挥动态的优势。
得益于 Nginx 和 LuaJIT 良好的多平台支持特性，OpenResty 不仅能运行在 X86 架构下，对于 ARM 的支持也很完善。同时， OpenResty 支持七层和四层的代理，这样一来，常见的各种协议都可以被 OpenResty 解析和代理，这其中也包括了 IoT 中的几种协议。
因为这些优势，我们便可以把 OpenResty 的触角，从 API 网关、WAF、web 服务器等服务端的领域，伸展到物联网设备、CDN 边缘节点、路由器等最靠近用户的边缘节点上去。
以 CDN 的边缘节点为例，OpenResty 的最大使用者 CloudFlare 很早就借助 OpenResty 的动态特性，实现了对于 CDN 边缘节点的动态控制。
CloudFlare 的做法和上面动态加载代码的原理是类似的，大概可以分为下面几个步骤：

* 首先，从键值数据库集群中获取到有变化的代码文件，获取的方式可以是后台 timer 轮询，也可以是用“发布 - 订阅”的模式来监听；
* 然后，用更新的代码文件替换本地磁盘的旧文件，然后使用 loadstring 和 pcall的方式，来更新内存中加载的缓存；

这样，下一个被处理的终端请求，就会走更新后的代码逻辑。
当然，实际的应用要比上面的步骤考虑更多的细节，比如版本的控制和回退、异常的处理、网络的中断、边缘节点的重启等，但整体的流程是不变的。

## 6.4 动态上游
现在，让我们把思绪拉回到 OpenResty 上来，一起来看如何实现动态上游。lua-resty-core 提供了 ngx.balancer 这个库来设置上游，它需要放到 OpenResty 的 balancer 阶段来运行：
```lua
balancer_by_lua_block {
    local balancer = require "ngx.balancer"
    local host = "127.0.0.2"
    local port = 8080
 
    local ok, err = balancer.set_current_peer(host, port)
    if not ok then
        ngx.log(ngx.ERR, "failed to set the current peer: ", err)
        return ngx.exit(500)
    end
}
```
set_current_peer 函数，就是用来设置上游的 IP 地址和端口的。不过要注意，这里并不支持域名，你需要使用 lua-resty-dns 库来为域名和 IP 做一层解析。
不过，ngx.balancer 还比较底层，虽然它有设置上游的能力，但动态上游的实现远非如此简单。所以，在 ngx.balancer 前面还需要两个功能：
- 上游选择算法
- 上游健康检查

OpenResty 官方的 lua-resty-balancer 这个库中，则包含了 resty.chash 和 resty.roundrobin 两类算法来完成第一个功能，并且有 lua-resty-upstream-healthcheck 来尝试完成第二个功能。其中resty-upstream-healthcheck只支持被动的健康检查，可以使用lua-resty-healthcheck 包含主动和被动健康检查。
新兴的微服务 API 网关 APISIX，在 lua-resty-healthcheck 的基础之上，对动态上游做了完整的实现。我们可以参考它的实现，总共只有 200 多行代码，你可以很轻松地把它剥离出来，放到你的自己的项目中使用。

# 7 OpenResty常用第三方库
## 7.1 lua-resty库推荐
首先推荐的是由 Aapo 维护的 awesome-resty [仓库](https://github.com/bungle/awesome-resty)，这个仓库分门别类地整理了和 OpenResty 相关的库，可以说是包罗万象，包括了 Nginx 的 C 模块、lua-resty 库、web 框架、路由库、模板、测试框架等，是你寻找 OpenResty 资源的首选。

还可以去 luarocks、opm 和 GitHub 碰碰运气。有一些开源时间不长的、或者关注不多的库，可能就藏在其中。
还可以去 luarocks、opm 和 GitHub 碰碰运气。有一些开源时间不长的、或者关注不多的库，可能就藏在其中。

## 7.2 lua-var-nginx-module
ngx.var是一个性能损耗比较大的操作，在实际使用时，我们需要用ngx.ctx来做一层缓存。
[lua-var-nginx-module](https://github.com/api7/lua-var-nginx-module)相比ngx.var性能提升了5倍。

它采用的是 FFI 的方式，所以，你需要在编译 OpenResty 的时候，先加上编译选项：
```
./configure --prefix=/opt/openresty \
         --add-module=/path/to/lua-var-nginx-module
```
调用的方法也很简单，只需要一行 fetch 函数的调用就可以了。它的效果完全等价于原有的 ngx.var.remote_addr，来获取到终端的 IP 地址：
```lua
content_by_lua_block {
    local var = require("resty.ngxvar")
    ngx.say(var.fetch("remote_addr"))
}
```
这个模块到底是怎么做到性能大幅度提升的呢？还是那句老话，源码面前无秘密，就让我们来看看 remote_addr 这个变量在其中是如何获取的吧：
```c
ngx_int_t 
ngx_http_lua_var_ffi_remote_addr(ngx_http_request_t *r, ngx_str_t *remote_addr) 
{ 
    remote_addr->len = r->connection->addr_text.len; 
    remote_addr->data = r->connection->addr_text.data; 
 
    return NGX_OK; 
}
```
它的优点很明显，使用 FFI 的方式来直接获取变量，绕过了 ngx.var 原有的查找逻辑；同时，缺点也很明显，那就是要为每一个希望获取的变量，都增加对应的 C 函数和 FFI 调用，这其实是一个体力活。

## 7.3 JSON Schema
[lua-rapidjson](https://github.com/xpol/lua-rapidjson) 。它是对 rapidjson 这个腾讯开源的 JSON 库的封装，以性能见长。这里，我们着重介绍下它和 cjson 的不同之处，也就是支持 JSON Schema。
JSON Schema 是一个通用的标准，借助这个标准，我们就可以精确地描述接口中参数的格式，以及如何校验的问题。下面是一个简单的示例：

```lua
"stringArray": {
    "type": "array",
    "items": { "type": "string" },
    "minItems": 1,
    "uniqueItems": true
}
```
这段 JSON 准确地描述了 stringArray 这个参数的类型是字符串数组，并且数组不能为空，数组元素也不能重复。
而lua-rapidjson，则是可以让我们在 OpenResty 中来使用 JSON Schema，这能给接口的校验带来极大的便利。举个例子，比如对于前面介绍过的 limit count 限流接口，我们就可以用下面的 schema 来描述：
```lua
local schema = {
    type = "object",
    properties = {
        count = {type = "integer", minimum = 0},
        time_window = {type = "integer",  minimum = 0},
        key = {type = "string", enum = {"remote_addr", "server_addr"}},
        rejected_code = {type = "integer", minimum = 200, maximum = 600},
    },
    additionalProperties = false,
    required = {"count", "time_window", "key", "rejected_code"},
}
```
你会发现，这可以带来两个十分明显的收益：
- 对前端来说，前端可以直接复用这个 schema 描述，用于前端页面的开发和参数校验，而不用再去关心后端；
- 而对后端来说，后端直接使用 lua-rapidjson 的 schema 校验函数 SchemaValidator 就能完成接口合法性的判断，更是无须编写多余的代码。

## 7.4 worker 间通信
假设如下一个场景：
> 一个 OpenResty 服务有 24 个 worker 进程，管理员通过 REST HTTP 接口更新了系统的某项配置，这时候只有一个 worker 收到了管理员的更新操作，并把结果写入了数据库，更新了共享字典和自己 worker 内的 lru 缓存。那么，其他 23 个 worker 怎么才能被通知去更新这项配置呢？

显然，多个 worker 之间需要一个通知的机制，才能完成上面的这个任务。在 OpenResty 自身不支持的情况下，我们就只能通过共享字典这个跨 worker 可以访问的空间，来曲线救国了。
[lua-resty-worker-events](https://github.com/Kong/lua-resty-worker-events)便是这个思路的具体实现。它在共享字典中维护了一个版本号，在有新消息需要发布的时候，给这个版本号加一，并把消息内容放到以版本号为 key 的字典中：
```LUA
event_id, err = _dict:incr(KEY_LAST_ID, 1)
success, err = _dict:add(KEY_DATA .. tostring(event_id), json)
```
同时，在后台使用 ngx.timer 创建了一个默认间隔为 1 秒的 polling 循环，来不断地检测版本号是否有变化：

```lua
local event_id, err = get_event_id()
if event_id == _last_event then
    return "done"
end
```
这样，一旦发现有新的事件通知需要处理时，就根据版本号从共享字典中获取消息内容：
```lua
while _last_event < event_id do
    count = count + 1
    _last_event = _last_event + 1
    data, err = _dict:get(KEY_DATA..tostring(_last_event))
end
```
总的来说，虽然 lua-resty-worker-events 会有 1 秒钟的延时，但还是实现了 worker 之间的事件通知机制，瑕不掩瑜。

# 8 答疑
## 8.1 问题一：如何完成Lua模块的动态加载？
示例：
```lua
resty -e 'local s = [[
local ngx = ngx
local _M = {}
function _M.f()
    ngx.say("hello world")
end
return _M
]]
local lua = loadstring(s)
local ret, func = pcall(lua)
func.f()'
```
这里的字符串 s，它的内容就是一个完整的 Lua 模块。所以，在发现这个模块的代码有变化时，你可以用 loadstring 或者 loadfile 来重启加载。这样，其中的函数和变量都会随之更新。
更进一步，你也把可以把获取变化和重新加载，用名为 code_loader 函数做一层包装：

```
local func = code_loader(name)
```
这样一来，代码更新就会变得更为简洁；同时， code_loader 中我们一般会用 lru cache 对 s 做一层缓存，避免每一次都去调用 loadstring。这差不多就是一个完整的实现了。

## 8.2 问题二：LuaJIT 的 NYI 的操作，是否会对性能有很大影响？
Q：loadstring 在 LuaJIT 的 NYI 列表是 never，会不会对性能有很大影响？

A：关于 LuaJIT 的 NYI，我们不用矫枉过正。对于可以 JIT 的操作，自然是 JIT 的方式最好；但对于还不能 JIT 的操作，我们也不是不能使用。

对于性能优化，我们需要用基于统计的科学方法来看待，这也就是火焰图采样的意义。过早优化是万恶之源。对于那些调用次数频繁、消耗 CPU 很高的热代码，我们才有优化的必要。
回到 loadstring 的问题，我们只会在代码发生变化的时候，才会调用它重新加载，和请求多少无关，所以它并不是一个频繁的操作。这个时候，我们就不用担心它对系统整体性能的影响。
结合第二个阻塞的问题，在 OpenResty 中，我们有些时候也会在 init 和 init worker 阶段，去调用阻塞的文件 I/O 操作。这种操作比 NYI 更加影响性能，但因为它只在服务启动的时候执行一次，所以也是可以被我们接受的。
还是那句话，性能优化要从宏观的视角来看待，这是你特别需要注意的一个点。否则，纠结于某一细节，就很有可能优化了半天，却并没有起到很好的效果。

## 8.3 问题三：共享字典的缓存是必须的吗？
Q：在实际的生产应用中，我认为 shared dict 这一层缓存是必须的。貌似大家都只记得 lrucache 的好，数据格式没限制、不需要反序列化、不需要根据 k/v 体积算内存空间、worker 间独立不相互争抢、没有读写锁、性能高云云。
但是，却忘记了它最致命的一个弱点，就是 lrucache 的生命周期是跟着 worker 走的。每当 Nginx reload 时，这部分缓存会全部丢失，这时候，如果没有 shared dict，那 L3 的数据源分分钟被打挂。当然，这是并发比较高的情况下，但是既然用到了缓存，就说明业务体量肯定不会小，也就是刚刚的分析仍然适用。不知道我的这个观点对吗？

A：大部分情况下，确实如你所说，共享字典在 reload 的时候不会丢失，所以它有存在的必要性。但也有一种特例，那就是，如果在 init 阶段或者 init_worker 阶段，就能从 L3 也就是数据源主动获取到所有数据，那么只有 lrucache 也是可以接受的。
举例来说，比如开源 API 网关 [APISIX](https://github.com/apache/apisix) 的数据源在 etcd 中，它只在 init_worker 阶段，从 etcd 中获取数据并缓存在 lrucache 中，后面的缓存更新，都是通过 etcd 的 watch 机制来主动获取的。这样一来，即使 Nginx reload ，也不会有缓存风暴产生。
所以，对待技术的选择，我们可以有倾向，但还是不要一概而论绝对化，因为并没有一个可以适合所有缓存场景的银弹。根据实际场景的需要，构建一个最小化可用的方案，然后逐步地增加，是一个不错的法子。