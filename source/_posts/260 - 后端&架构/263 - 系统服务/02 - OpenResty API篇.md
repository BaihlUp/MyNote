---
title: OpenResty API篇
date: 2023-11-21 16:27:41
categories:
  - 后端&架构
tags:
  - OpenResty
published: false
---

# 2 OpenResty开发平台
### 2.1.1 协程和事件驱动
在 OpenResty 层面，Lua 的协程会与 NGINX 的事件机制相互配合。如果 Lua 代码中出现类似查询 MySQL 数据库这样的 I/O 操作，就会先调用 Lua 协程的 yield 把自己挂起，然后在 NGINX 中注册回调；在 I/O 操作完成（也可能是超时或者出错）后，再由 NGINX 回调 resume 来唤醒 Lua 协程。这样就完成了 Lua 协程和 NGINX 事件驱动的配合，避免在 Lua 代码中写回调。
我们可以来看下面这张图，描述了这整个流程。其中，`lua_yield` 和 `lua_resume` 都属于 Lua 提供的 `lua_CFunction`。
![](https://i.loli.net/2020/09/14/sdrcA42uWpTaQ9x.png)

下面是 `ngx.sleep` 的一段源码，可以帮你更清晰理解这一点。 这段代码位于 `ngx_http_lua_sleep.c` 中，你可以在 `lua-nginx-module` 项目的 [src](https://github.com/openresty/lua-nginx-module/tree/master/src) 目录中找到它。
在`ngx_http_lua_sleep.c` 中，我们可以看到 sleep 函数的具体实现。你需要先通过 C 函数 `ngx_http_lua_ngx_sleep`，来注册 `ngx.sleep` 这个 Lua API：

```c
void
ngx_http_lua_inject_sleep_api(lua_State *L)
{
     lua_pushcfunction(L, ngx_http_lua_ngx_sleep);
     lua_setfield(L, -2, "sleep");
}
```
下面便是 sleep 的主函数，这里我只摘取了几行主要的代码：

```c
static int ngx_http_lua_ngx_sleep(lua_State *L)
{
    coctx->sleep.handler = ngx_http_lua_sleep_handler;
    ngx_add_timer(&coctx->sleep, (ngx_msec_t) delay);
    return lua_yield(L, 0);
}
```
- 这里先增加了 `ngx_http_lua_sleep_handler` 这个回调函数；
- 然后调用 `ngx_add_timer` 这个 NGINX 提供的接口，向 NGINX 的事件循环中增加一个定时器；
- 最后使用 `lua_yield` 把 Lua 协程挂起，把控制权交给 NGINX 的事件循环。

当 sleep 操作完成后， `ngx_http_lua_sleep_handler` 这个回调函数就被触发了。它里面调用了 `ngx_http_lua_sleep_resume`, 并最终使用 `lua_resume` 唤醒了 Lua 协程。更具体的调用过程，可以自己去代码里面检索。

### 2.1.2 OpenResty 的阶段
- `set_by_lua`，用于设置变量；
- `rewrite_by_lua`，改写URI，可用于实现跳转/重定向；
- `access_by_lua`，用于处理访问控制或限速；
- `content_by_lua`，用于生成返回内容；
- `header_filter_by_lua`，用于应答头过滤处理；
- `body_filter_by_lua`，用于应答体过滤处理；
- `log_by_lua`，用于日志记录。

在编写模块时，不同的API在使用前应该弄清楚这个API的context，如果使用不当会报错。如下示例：
以`ngx.sleep` 为例，通过查阅文档，我知道它只能用于下面列出的上下文中，并不包括 log 阶段：

```conf
context: rewrite_by_lua*, access_by_lua*, content_by_lua*, ngx.timer.*, ssl_certificate_by_lua*, ssl_session_fetch_by_lua*_
```
而如果你不知道这一点，在它不支持的 log 阶段使用 sleep 的话：

```lua
location / {
    log_by_lua_block {
        ngx.sleep(1)
     }
}
```
在 NGINX 的错误日志中，就会出现 error 级别的提示：

```log
[error] 62666#0: *6 failed to run log_by_lua*: log_by_lua(nginx.conf:14):2: API disabled in the context of log_by_lua*
stack traceback:
    [C]: in function 'sleep'
```

## 2.2 文档和测试案例
在学习 OpenResty 时一定要多看文档和测试用例，在模块的 `t` 目录下有模块的完整用例集，可以方便测试和学习。
下边简单以`shdict get API` 为例：
[文档链接](https://github.com/openresty/lua-nginx-module/#ngxshareddictget) 为对照。下边是一个get方法：

```lua
 http {
     lua_shared_dict dogs 10m;
     server {
         location /demo {
             content_by_lua_block {
                 local dogs = ngx.shared.dogs
        dogs:set("Jim", 8)
        local v = dogs:get("Jim")
                 ngx.say(v)
             }
         }
     }
 }
```
简单说明一下，在 Lua 代码中使用 shared dict 之前，我们需要在 nginx.conf 中用 `lua_shared_dict` 指令增加一块内存空间，它的名字是 dogs，大小为 10M。修改完 nginx.conf 后，你还需要重启进程，用浏览器或者 curl 访问才能看到结果。
下边是shdict get API 的一个测试用例：
```perl
=== TEST 1: string key, int value
 --- http_config
     lua_shared_dict dogs 1m;
 --- config
     location = /test {
         init_by_lua '
             local dogs = ngx.shared.dogs
             local val = dogs:get("foo")
             ngx.say(val)
         ';
     }
 --- request
 GET /test
 --- response_body
 32
 --- no_error_log
 [error]
 --- ONLY
```

# 3 OpenResty 处理终端请求和响应
## 3.1 API 分类
**OpenResty 的 API 主要分为下面几个大类：**
- 处理请求和响应；
- SSL 相关；
- shared dict；
- cosocket；
- 处理四层流量；
- process 和 worker；
- 获取 NGINX 变量和配置；
- 字符串、时间、编解码等通用功能。

OpenResty 的 API 不仅仅存在于 nginx-lua-module 项目中，也存在于 `lua-resty-core` 项目中，比如 `ngx.ssl`、`ngx.base64`、`ngx.errlog`、`ngx.process`、`ngx.re.split`、`ngx.resp.add_header`、`ngx.balancer`、`ngx.semaphore`、`ngx.ocsp` 这些 API 。

而对于不在 nginx-lua-module 项目中的 API，你需要单独 require 才能使用。

## 3.2 请求处理
### 3.2.1 请求行
OpenResty 中的 `ngx.var.*` 这个 API可以取出响应的数据，如下：
- `ngx.var.scheme` 返回协议名字，是“http”或“https”
- `ngx.var.request_method`  代表请求方法

但是在OpenResty 提供了有相应的ngx.req开头的 API，为什么还要有ngx.var 呢。

- 首先是对性能的考虑。ngx.var 的效率不高，不建议反复读取；
- 也有对程序友好的考虑，ngx.var 返回的是字符串，而非 Lua 对象，遇到获取 args 这种可能返回多个值的情况，就不好处理了；
- 另外是对灵活性的考虑，绝大部分的 ngx.var 是只读的，只有很少数的变量是可写的，比如 $args 和 limit_rate，可很多时候，我们会有修改 method、URI 和 args 的需求。

可以打开 [API列表](https://github.com/openresty/lua-nginx-module/#nginx-api-for-lua) 具体查看。

- `ngx.req.get_method`  返回：字符串格式的方法名
- `ngx.req.set_method(ngx.HTTP_POST)` 参数是数字常量

```bash
[root@localhost]#resty -e 'print(ngx.HTTP_POST)'
8
```
- `ngx.req.set_uri` 和 `ngx.req.set_uri_args` 这两个 API，可以用来改写 uri 和 args。

### 3.2.2 请求头
`ngx.req.get_headers` 来解析和获取请求头，返回值的类型则是 table：

```lua
local h, err = ngx.req.get_headers()
 
  if err == "truncated" then
      -- one can choose to ignore or reject the current request here
  end
 
  for k, v in pairs(h) do
      ...
  end
```
这里默认返回前 100 个 header，如果请求头超过了 100 个，就会返回 truncated 的错误信息，由开发者自己决定如何处理。
 - 改写和删除请求头：
```
ngx.req.set_header("Content-Type", "text/css")
ngx.req.clear_header("Content-Type")
```

> 需要注意的是，OpenResty 并没有提供获取某一个指定请求头的 API，也就是没有 `ngx.req.header['host']` 这种形式。如果你有这样的需求，那就需要借助 NGINX 的变量 $http_xxx 来实现了，那么在 OpenResty 中，就是 `ngx.var.http_xxx` 这样的获取方式。

### 3.2.3 请求体
出于性能考虑，OpenResty 不会主动读取请求体的内容，除非在 nginx.conf 中强制开启了 `lua_need_request_body` 指令。此外，对于比较大的请求体，OpenResty 会把内容保存在磁盘的临时文件中，所以读取请求体的完整流程是下面这样的：

```lua
ngx.req.read_body()
local data = ngx.req.get_body_data()
if not data then
    local tmp_file = ngx.req.get_body_file()
     -- io.open(tmp_file)
     -- ...
 end
```

## 3.3 处理响应
### 3.3.1 响应状态
在默认情况下，返回的 HTTP 状态码是 200，也就是 OpenResty 中内置的常量 ngx.HTTP_OK。
OpenResty 的 HTTP 状态码中，有一个特别的常量：ngx.OK。当 ngx.exit(ngx.OK) 时，请求会退出当前处理阶段，进入下一个阶段，而不是直接返回给客户端。

```lua
ngx.exit(ngx.HTTP_BAD_REQUEST)
```
改写状态码：

```lua
ngx.status = ngx.HTTP_FORBIDDEN
```
更多的状态码常量，查看[文档](https://github.com/openresty/lua-nginx-module/#http-status-constants)

### 3.3.2 响应头
响应头，其实，你有两种方法来设置它。第一种是最简单的：
```lua
ngx.header.content_type = 'text/plain'
ngx.header["X-My-Header"] = 'blah blah'
ngx.header["X-My-Header"] = nil -- 删除
```
这里的 ngx.header 保存了响应头的信息，可以读取、修改和删除。
第二种设置响应头的方法是 `ngx_resp.add_header` ，来自 lua-resty-core 仓库，它可以增加一个头信息，用下面的方法来调用：

```lua
local ngx_resp = require "ngx.resp"
ngx_resp.add_header("Foo", "bar")
```
与第一种方法的不同之处在于，add header 不会覆盖已经存在的同名字段。

### 3.3.3 响应体
可以使用 `ngx.say` 和 `ngx.print` 来输出响应体：

```lua
ngx.say('hello, world')
```
这两个 API 的功能是一致的，唯一的不同在于， `ngx.say` 会在最后多一个换行符。

## 3.4 常量
### 3.4.1 状态码
- ngx.HTTP OK：200，请求已成功处理
- ngx.HTTP MOVED TEMPORARILY：302，重定向跳转
- ngx.HTTP BAD REQUEST：400，客户端请求错误
- ngx.HTTP UNAUTHORIZED：401，未认证
- ngx.HTTP FORBIDDEN：禁止访问 403
- ngx.HTTP NOT FOUND：404，资源未找到
- ngx.HTTP INTERNAL SERVER ERROR：500，服务器内部错误
- ngx.HTTP BAD GATEWAY：502，网关错误，反向代理后端无效响应
- ngX.HTTP SERVICE UNAVAILABLE：503，服务器暂不可用
- ngx.HTTP GATEWAY TIMEOUT：504，网关超时，反向代理时后端超时

在编写代码时不使用这些常量，直接用200、404这样的数字字面值也是可以的，两者完全等价，OpenResty对此没有强制要求。

### 3.4.2 请求方法
- ngx.HTTP GET：读操作，获取数据
- ngx.HTTP HEAD：读操作，获取元数据
- ngx.HTTP POST：写操作，提交数据
- ngx.HTTP PUT：写操作，更新数据
- ngX.HTTP DELETE：写操作，删除数据
- ngx.HTTP PATCH：写操作，局部更新数据

要注意的是这些常量并不是字符串，而是数字。


# 4 OpenResty 中数据共享的方式
OpenResty中的共享内存字典 shared dict，是最重要的数据结构。它不仅支持数据的存放和读取，还支持原子计数和队列操作。
基于 shared dict，你可以实现多个 worker 之间的缓存和通信，以及限流限速、流量统计等功能。你可以把 shared dict 当作简单的 Redis 来使用，只不过 shared dict 中的数据不能持久化，所以存放在其中的数据，一定要考虑到丢失的情况。
除了shared dict 还有其他几种数据共享的方式。

## 4.1 Nginx 中的变量
它可以在 Nginx C 模块之间共享数据，自然的，也可以在 C 模块和 OpenResty 提供的 `lua-nginx-module` 之间共享数据，比如下面这段代码：

```lua
location /foo {
     set $my_var ''; # this line is required to create $my_var at config time
     content_by_lua_block {
         ngx.var.my_var = 123;
         ...
     }
 }
```
不过，使用 Nginx 变量这种方式来共享数据是比较慢的，因为它涉及到 hash 查找和内存分配。同时，这种方法有其局限性，只能用来存储字符串，不能支持复杂的 Lua 类型。
## 4.2 ngx.ctx
可以在同一个请求的不同阶段之间共享数据。它其实就是一个普通的 Lua 的 table，所以速度很快，还可以存储各种 Lua 的对象。它的生命周期是请求级别的，当一个请求结束的时候，ngx.ctx 也会跟着被销毁掉。

下面是一个典型的使用场景，我们用 ngx.ctx 来缓存 Nginx 变量 这种昂贵的调用，并在不同阶段都可以使用到它：

```lua
location /test {
     rewrite_by_lua_block {
         ngx.ctx.host = ngx.var.host
     }
     access_by_lua_block {
        if (ngx.ctx.host == 'openresty.org') then
            ngx.ctx.host = 'test.com'
        end
     }
     content_by_lua_block {
         ngx.say(ngx.ctx.host)
     }
 }
```
如果你使用 curl 访问的话：

```bash
curl -i 127.0.0.1:8080/test -H 'host:openresty.org'
```
则响应 `test.com`。
需要注意的是 ngx.ctx 的生命周期是请求级别的，所以并不能在模块级别进行缓存。
如下使用就是错误的：
```lua
local ngx_ctx = ngx.ctx
 
local function bar()
    ngx_ctx.host =  'test.com'
end
```
应该在函数级别进行调用和缓存：

```lua
local ngx = ngx
 
local function bar()
	local ngx_ctx = ngx.ctx
    ngx_ctx.host =  'test.com'
end
```

## 4.3 模块级别的变量
可以在同一个 worker 内的所有请求之间共享数据。
下边先解释下模块级别的变量是什么？

```lua
-- mydata.lua
 local _M = {}
 
 local data = {
     dog = 3,
     cat = 4,
     pig = 5,
 }
 
 function _M.get_age(name)
     return data[name]
 end
 
 return _M
```
在 nginx.conf 的配置如下：

```lua
location /lua {
     content_by_lua_block {
         local mydata = require "mydata"
         ngx.say(mydata.get_age("dog"))
     }
 }
```
在这个示例中，mydata 就是一个模块，它只会被 worker 进程加载一次，之后，这个 worker 处理的所有请求，都会共享 mydata 模块的代码和数据。
自然，mydata 模块中的 data 这个变量，就是 模块级别的变量。

需要特别注意的是，一般我们只用这种方式来保存只读的数据。

## 4.4 shared dict
这些数据可以在多个 worker 之间共享。
这种方法是基于红黑树实现的，性能很好，但也有自己的局限性——你必须事先在 Nginx 的配置文件中，声明共享内存的大小，并且这不能在运行期更改：
```lua
lua_shared_dict dogs 10m;
```
shared dict 同样只能缓存字符串类型的数据，不支持复杂的 Lua 数据类型。详细API查看[文档](https://github.com/openresty/lua-nginx-module#ngxshareddict)
> 前面三种数据共享的范围都是在请求级别，或者单个 worker 级别。所以，在当前的 OpenResty 的实现中，只有 shared dict 可以完成 worker 间的数据共享，并借此实现 worker 之间的通信，这也是它存在的价值。

shared dict 在实现多个进程共享时，并不需要加锁，因为其中的接口已经实现了原子操作。

 - 读写类

首先来看字典读写类。在最初的版本中，只有字典读写类的 API，它们也是共享字典最常用的功能。下面是一个最简单的示例：

```lua
$ resty --shdict='dogs 1m' -e 'local dict = ngx.shared.dogs
                               dict:set("Tom", 56)
                               print(dict:get("Tom"))'
```
除了 set 外，OpenResty 还提供了 safe_set、add、safe_add、replace 这四种写入的方法。这里safe 前缀的含义是，在内存占满的情况下，不根据 LRU 淘汰旧的数据，而是写入失败并返回 no memory 的错误信息。
`get_stale` 的读取数据的方法，相比 get 方法，它多了一个过期数据的返回值：

```
value, flags, stale = ngx.shared.DICT:get_stale(key)
```
可以调用 delete 方法来删除指定的 key，它和 set(key, nil) 是等价的。
 - 队列操作类

- lpush/rpush，表示在队列两端增加元素；
- lpop/rpop，表示在队列两端弹出元素；
- llen，表示返回队列的元素数量。

测试案例查看 `145-shdict-list.t` 文件

```perl
=== TEST 1: lpush & lpop
--- http_config
    lua_shared_dict dogs 1m;
--- config
    location = /test {
        content_by_lua_block {
            local dogs = ngx.shared.dogs
 
            local len, err = dogs:lpush("foo", "bar")
            if len then
                ngx.say("push success")
            else
                ngx.say("push err: ", err)
            end
 
            local val, err = dogs:llen("foo")
            ngx.say(val, " ", err)
 
            local val, err = dogs:lpop("foo")
            ngx.say(val, " ", err)
 
            local val, err = dogs:llen("foo")
            ngx.say(val, " ", err)
 
            local val, err = dogs:lpop("foo")
            ngx.say(val, " ", err)
        }
    }
--- request
GET /test
--- response_body
push success
1 nil
bar nil
0 nil
nil nil
--- no_error_log
[error]
```

- 管理类

1. get_keys(max_count?)，它默认也只返回前 1024 个 key；如果你把 max_count 设置为 0，那就返回所有 key。
2. capacity 和 free_space，这两个 API 都属于 lua-resty-core 仓库，所以需要你 require 后才能使用：
```lua
require "resty.core.shdict"
 
 local cats = ngx.shared.cats
 local capacity_bytes = cats:capacity()
 local free_page_bytes = cats:free_space()
```
它们分别返回的，是共享内存的大小（也就是 lua_shared_dict 中配置的大小）和空闲页的字节数。因为 shared dict 是按照页来分配的，即使 free_space 返回为 0，在已经分配的页面中也可能存在空间，所以它的返回值并不能代表共享内存实际被占用的情况。

# 5 cosocket
cosocket 是各种 `lua-resty-*` 非阻塞库的基础，没有 cosocket，开发者就无法用 Lua 来快速连接各种外部的网络服务。

## 5.1 什么是 cosocket？
cosocket 是 OpenResty 中的专有名词，是把协程和网络套接字的英文拼在一起形成的，即 cosocket = coroutine + socket。所以，你可以把 cosocket 翻译为“协程套接字”。
如果我们在 OpenResty 中调用一个 cosocket 相关函数，内部实现便是下面这张图的样子：
![](https://i.loli.net/2020/09/14/3KuNVFmaEhjIlUD.png)
遇到网络 I/O 时，它会交出控制权（yield），把网络事件注册到 Nginx 监听列表中，并把权限交给 Nginx；当有 Nginx 事件达到触发条件时，便唤醒对应的协程继续处理（resume）。

OpenResty 正是以此为蓝图，封装实现 connect、send、receive 等操作，形成了我们如今见到的 cosocket API。下面，我就以处理 TCP 的 API 为例来介绍一下。处理 UDP 和 Unix Domain Socket ，与 TCP 的接口基本是一样的。

## 5.2 cosocket API 和指令简介
TCP 相关的 cosocket API 可以分为下面这几类。
- 创建对象：ngx.socket.tcp。
- 设置超时：tcpsock:settimeout 和 tcpsock:settimeouts。
- 建立连接：tcpsock:connect。
- 发送数据：tcpsock:send。
- 接受数据：tcpsock:receive、tcpsock:receiveany 和 - tcpsock:receiveuntil。
- 连接池：tcpsock:setkeepalive。
- 关闭连接：tcpsock:close。

这些 API 可以使用的上下文：

```lua
rewrite_by_lua*, access_by_lua*, content_by_lua*, ngx.timer.*, ssl_certificate_by_lua*, ssl_session_fetch_by_lua*_
```

与这些 API 相关的，还有 8 个 `lua_socket_` 开头的 Nginx 指令：
- lua_socket_connect_timeout：连接超时，默认 60 秒。
- lua_socket_send_timeout：发送超时，默认 60 秒。
- lua_socket_send_lowat：发送阈值（low water），默认为 0。
- lua_socket_read_timeout： 读取超时，默认 60 秒。
- lua_socket_buffer_size：读取数据的缓存区大小，默认 4k/8k。
- lua_socket_pool_size：连接池大小，默认 30。
- lua_socket_keepalive_timeout：连接池 cosocket 对象的空闲时间，默认 60 秒。
- lua_socket_log_errors：cosocket 发生错误时，是否记录日志，默认为 on。

这里你也可以看到，有些指令和 API 的功能一样的，比如设置超时时间和连接池大小等。不过，如果两者有冲突的话，API 的优先级高于指令，会覆盖指令设置的值。

**示例：**

```lua
$ resty -e 'local sock = ngx.socket.tcp()
        sock:settimeout(1000)  -- one second timeout
        local ok, err = sock:connect("www.baidu.com", 80)
        if not ok then
            ngx.say("failed to connect: ", err)
            return
        end
 
        local req_data = "GET / HTTP/1.1\r\nHost: www.baidu.com\r\n\r\n"
        local bytes, err = sock:send(req_data)
        if err then
            ngx.say("failed to send: ", err)
            return
        end
 
        local data, err, partial = sock:receive()
        if err then
            ngx.say("failed to receive: ", err)
            return
        end
 
        sock:close()
        ngx.say("response is: ", data)'
```
我们来具体分析下这段代码：
- 首先，通过 ngx.socket.tcp() ，创建 TCP 的 cosocket 对象，名字是 sock。
- 然后，使用 settimeout() ，把超时时间设置为 1 秒。注意这里的超时没有区分 connect、receive，是统一的设置。
- 接着，使用 connect() 去连接指定网站的 80 端口，如果失败就直接退出。
- 连接成功的话，就使用 send() 来发送构造好的数据，如果发送失败就退出。
- 发送数据成功的话，就使用 receive() 来接收网站返回的数据。这里 receive() 的默认参数值是 `*l`，也就是只返回第一行的数据；如果参数设置为了`*a`，就是持续接收数据，直到连接关闭；
- 最后，调用 close() ，主动关闭 socket 连接。

下边做一些调整：
1. 对 socket 连接、发送和读取这三个动作，分别设置超时时间。

settimeout() ，作用是把超时时间统一设置为一个值。如果要想分开设置，就需要使用 settimeouts() 函数，比如下面这样的写法：
```lua
sock:settimeouts(1000, 2000, 3000) 
```
这行代码表示连接超时为 1 秒，发送超时为 2 秒，读取超时为 3 秒。
在 OpenResty 和 lua-resty 库中，大部分和时间相关的 API 的参数，都以毫秒为单位，但也有例外，需要你在调用的时候特别注意下。

2. receive 接收指定大小的内容。

刚才的 receive() 接口可以接收一行数据，也可以持续接收数据。如果要接收比如10K 大小的数据，使用如下接口：

```lua
local data, err, partial = sock:receiveany(10240)
```

receiveuntil()  可以持续获取数据，直到遇到指定字符串就停止。返回的是一个迭代器：

```lua
local reader = sock:receiveuntil("\r\n")
 while true do
     local data, err, partial = reader(4)
     if not data then
         if err then
             ngx.say("failed to read the data stream: ", err)
             break
         end
 
         ngx.say("read done")
         break
     end
     ngx.say("read chunk: [", data, "]")
 end
```
这段代码中的 receiveuntil 会返回 \r\n 之前的数据，并通过迭代器每次读取其中的 4 个字节，也就实现了我们想要的功能。

3. 不直接关闭 socket，而是放入连接池中。

没有连接池的话，每次请求进来都要新建一个连接，就会导致 cosocket 对象被频繁地创建和销毁，造成不必要的性能损耗。
为了避免这个问题，在你使用完一个 cosocket 后，可以调用 setkeepalive() 放到连接池中，比如下面这样的写法：

```lua
local ok, err = sock:setkeepalive(2 * 1000, 100)
if not ok then
    ngx.say("failed to set reusable: ", err)
end
```
这段代码设置了连接的空闲时间为 2 秒，连接池的大小为 100。这样，在调用 connect() 函数时，就会优先从连接池中获取 cosocket 对象。
关于连接池的使用，有两点需要注意一下：
- 第一，不能把发生错误的连接放入连接池，否则下次使用时，就会导致收发数据失败。这也是为什么需要判断每一个 API 调用是否成功的一个原因。
- 第二，要搞清楚连接的数量。连接池是 worker 级别的，每个 worker 都有自己的连接池。所以，如果你有 10 个 worker，连接池大小设置为 30，那么对于后端的服务来讲，就等于有 300 个连接。

# 6 特权任务和定时任务
## 6.1 OpenResty 中启动定时任务
OpenResty 的定时任务可以分为下面两种：
1. `ngx.timer.at` 用来执行一次性的定时任务
2. `ngx.time.every` 用来执行固定周期的定时任务

示例，启动了一个延时为 0 的定时任务。回调handler函数，并在函数中用cosocket访问一个网站：
```lua
init_worker_by_lua_block {
        local function handler()
            local sock = ngx.socket.tcp()
            local ok, err = sock:connect(“www.baidu.com", 80)
        end
 
        local ok, err = ngx.timer.at(0, handler)
    }
```
OpenResty中并没有接口可以取消设置的定时任务，所以如果定时任务的数量很多，就很容易耗尽系统资源：
OpenResty 提供了 `lua_max_pending_timers` 和 `lua_max_running_timers` 这两个指令，来对其进行限制。前者代表等待执行的定时任务的最大值，后者代表当前正在运行的定时任务的最大值。
你也可以通过 Lua API，来获取当前等待执行和正在执行的定时任务的值，下面是两个示例：

```lua
content_by_lua_block {
            ngx.timer.at(3, function() end)
            ngx.say(ngx.timer.pending_count())
        }
```
代码会打印出 1，表示有 1 个计划任务正在等待被执行。

```lua
content_by_lua_block {
            ngx.timer.at(0.1, function() ngx.sleep(0.3) end)
            ngx.sleep(0.2)
            ngx.say(ngx.timer.running_count())
        }
```
代码会打印出 1，表示有 1 个计划任务正在运行中。

## 6.2 特权进程
 OpenResty 在 Nginx 的基础上进行了扩展，增加了特权进程：privileged agent。特权进程很特别：
 - 它不监听任何端口，这就意味着不会对外提供任何服务；
- 它拥有和 master 进程一样的权限，一般来说是 root 用户的权限，这就让它可以做很多 worker 进程不可能完成的任务；
- 特权进程只能在 `init_by_lua` 上下文中开启；
- 另外，特权进程只有运行在 `init_worker_by_lua` 上下文中才有意义，因为没有请求触发，也就不会走到content、access 等上下文去。

下面，我们来看一个开启特权进程的示例：
```lua
init_by_lua_block {
    local process = require "ngx.process"
 
    local ok, err = process.enable_privileged_agent()
    if not ok then
        ngx.log(ngx.ERR, "enables privileged agent failed error:", err)
    end
}
```
配置以上之后，再启动OpenResty，可以看到Nginx进程中多了特权进行：
```bash
nginx: master process
nginx: worker process
nginx: privileged agent process
```
因为特权只在 `init_worker_by_lua` 阶段运行一次，所以如果需要定时执行，就需要使用上边的定时器功能，下边使用`ngx.timer`，周期性地触发：
```lua
init_worker_by_lua_block {
    local process = require "ngx.process"
 
    local function reload(premature)
        local f, err = io.open(ngx.config.prefix() .. "/logs/nginx.pid", "r")
        if not f then
            return
        end
        local pid = f:read()
        f:close()
        os.execute("kill -HUP " .. pid)
    end
 
    if process.type() == "privileged agent" then
         local ok, err = ngx.timer.every(5, reload)
        if not ok then
            ngx.log(ngx.ERR, err)
        end
    end
}
```
上面这段代码，实现了每 5 秒给 master 进程发送 HUP 信号量的功能。自然，你也可以在此基础上实现更多有趣的功能，比如轮询数据库，看是否有特权进程的任务并执行。因为特权进程是 root 权限，这显然就有点儿“后门”程序的意味了。

> 特权进程的使用场景，一般用特权进程来处理的是清理日志、重启 OpenResty 自身等需要高权限的任务。需要注意的是，不要把 worker 进程的任务交给特权进程来处理。这并非因为特权进程不能做到，而是其存在安全隐患。


## 6.3 非阻塞的 ngx.pipe
下边示例，给master发送信号的操作：
```lua
os.execute("kill -HUP " .. pid) 
```
这里使用了Lua的标准库，是阻塞操作，这里可以使用`lua-resty-shell` 来非阻塞的操作。

```lua
$ resty -e 'local shell = require "resty.shell"
local ok, stdout, stderr, reason, status =
    shell.run([[echo "hello, world"]])
    ngx.say(stdout)
```
这段代码可以算是 hello world 的另外一种写法了，它调用系统的 echo 命令来完成输出。类似的，可以用 resty.shell ，来替代 Lua 中的 os.execute 调用。
lua-resty-shell 的底层实现，依赖了 lua-resty-core 中的 [ngx.pipe](https://github.com/openresty/lua-resty-core/blob/master/lib/ngx/pipe.md) API，所以，这个使用 lua-resty-shell 打印出 hello wrold 的示例，改用 ngx.pipe ，可以写成下面这样：

```lua
$ resty -e 'local ngx_pipe = require "ngx.pipe"
local proc = ngx_pipe.spawn({"echo", "hello world"})
local data, err = proc:stdout_read_line()
ngx.say(data)'
```

# 7 OpenResy中使用正则、时间等API
## 7.1 正则
在OpenResty中的正则引擎是基于回溯的NFA来实现的，那么就有可能出现灾难性回溯，即正则在匹配的时候回溯过多，造成CPU 100%，正常服务被阻塞。
在OpenResty中可以使用如下配置进行规避：
```lua
lua_regex_match_limit 100000;
```
`lua_regex_match_limit` ，就是用来限制 PCRE 正则引擎的回溯次数的。这样，即使出现了灾难性回溯，后果也会被限制在一个范围内，不会导致你的 CPU 满载。

## 7.2 时间API

`ngx.now` 返回当前的时间戳，包括小数部分：

```lua
resty -e 'ngx.say(ngx.now())'
```
其他的 `ngx.localtime`、`ngx.utctime`、`ngx.cookie_time` 和 `ngx.http_time` ，主要是返回和处理时间的不同格式。

需要注意的是，这些返回当前时间的 API，如果没有非阻塞网络 IO 操作来触发，便会一直返回缓存的值，而不是像我们想的那样，能够返回当前的实时时间，下边示例：
```lua
$ resty -e 'ngx.say(ngx.now())
os.execute("sleep 1")
ngx.say(ngx.now())'
```
在两次调用 `ngx.now` 之间，我们使用 Lua 的阻塞函数 sleep 了 1 秒钟，但从打印的结果来看，这两次返回的时间戳却是一模一样的。

那么，如果换成是非阻塞的 sleep 函数呢？比如下面这段新的代码：

```lua
$ resty -e 'ngx.say(ngx.now())
ngx.sleep(1)
ngx.say(ngx.now())'
```
这里出现这种情况的原因是，`ngx.now()`这个获取当前时间的函数对时间做了缓存。下边是`ngx.now()` 的源码：
```c
static int
ngx_http_lua_ngx_now(lua_State *L)
{
    ngx_time_t              *tp;
 
    tp = ngx_timeofday();
 
    lua_pushnumber(L, (lua_Number) (tp->sec + tp->msec / 1000.0L));
 
    return 1;
}
```
可以看出，ngx.now()这个获取当前时间函数的背后，隐藏的其实是 Nginx 的 ngx_timeofday 函数。而ngx_timeofday 函数，其实是一个宏定义：
```c
#define ngx_timeofday()      (ngx_time_t *) ngx_cached_time
```
这里`ngx_cached_time` 的值，只在函数 `ngx_time_update` 中会更新。
ngx_time_update什么时候会被调用？如果你在 Nginx 的源码中去跟踪它的话，就会发现， ngx_time_update 的调用都出现在事件循环中。

## 7.3 真值和空值
在OpenResty中真值和空值的判断，一直是个比较头痛的点。下边列出不同情况下的空值和真值。
首先在Lua中，除了 nil 和 false 之外，都是真值。

- ngx.null 的布尔值为真

```lua
$ resty -e 'if ngx.null then
ngx.say("true")    ---> true
end'
```
- cdata:NULL
当你通过 LuaJIT FFI 接口去调用 C 函数，而这个函数返回一个 NULL 指针，那么你就会遇到另外一种空值，即`cdata:NULL` 。
和ngx.null一样，cdata:NULL 也是真
```lua
$ resty -e 'local ffi = require "ffi"
local cdata_null = ffi.new("void*", nil)
if cdata_null then
    ngx.say("true")    ---> true
end'
```
但是令人想不到的是下边的代码会打印出true，也就是说cdata:NULL 是和 nil 相等的：

```lua
$ resty -e 'local ffi = require "ffi"
local cdata_null = ffi.new("void*", nil)
ngx.say(cdata_null == nil)'
```
- cjson.null 布尔值为真
```lua
$ resty -e 'local cjson = require "cjson"
local data = cjson.encode(nil)
local decode_null = cjson.decode(data)
ngx.say(decode_null == cjson.null)'
```

# 8 OpenResty第三方库的阅读技巧
在OpenResty中没有自带的HTTP client库，下边列出两个比较优秀的：
- [lua-resty-http](https://github.com/ledgetech/lua-resty-http)
- [lua-resty-requests](https://github.com/tokers/lua-resty-requests)

先来看下第三方库的代码结构：
![](https://i.loli.net/2020/09/21/wT81XAtDPKsRMiJ.png)

在首页也有对开源库的接口介绍，可以根据文档进行测试学习。

# 9 OpenResty 处理四层流量
OpenResty中处理四层流量是在 stream上下文，OpenResty 其实还提供了 stream-lua-nginx-module 模块来处理四层的流量。它提供的指令和 API ，与 lua-nginx-module 基本一致。
下边实现一个服务端缓存session的功能，具体功能点如下：
1. 实现set操作，通过key:value方式保存
2. 实现get操作，通过key获取value
3. 支持过期

先来设置好 Nginx 的配置文件，因为 stream 和 shared dict 要在其中预设。
```
stream {
    lua_shared_dict memcached 100m;
    lua_package_path 'lib/?.lua;;';
    server {
        listen 11212;
        content_by_lua_block {
            local m = require("resty.memcached.server")
            m.run()
        }
    }
}
```
可以看到，这段配置文件中有几个关键的信息：
- 首先，代码运行在 Nginx 的 stream 上下文中，而非 HTTP 上下文中，并且监听了 11212 端口；
- 其次，shared dict 的名字为 memcached，大小是 100M，这些在运行期是不可以修改的；
- 另外，代码所在目录为 lib/resty/memcached, 文件名为 server.lua, 入口函数为 run()，这些信息你都可以从lua_package_path 和 `content_by_lua_block` 中找到。

下边是完整代码：
```lua
local new_tab = require "table.new"
local str_sub = string.sub
local re_find = ngx.re.find
local mc_shdict = ngx.shared.memcached
 
local _M = { _VERSION = '0.01' }
 
local function parse_args(s, start)
    local arr = {}
 
    while true do
        local from, to = re_find(s, [[\S+]], "jo", {pos = start})
        if not from then
            break
        end
 
        table.insert(arr, str_sub(s, from, to))
 
        start = to + 1
    end
 
    return arr
end
 
function _M.get(tcpsock, keys)
    local reply = ""
 
    for i = 1, #keys do
        local key = keys[i]
        local value, flags = mc_shdict:get(key)
        if value then
            local flags  = flags or 0
            reply = reply .. "VALUE" .. key .. " " .. flags .. " " .. #value .. "\r\n" .. value .. "\r\n"
        end
    end
    reply = reply ..  "END\r\n"
 
    tcpsock:settimeout(1000)  -- one second timeout
    local bytes, err = tcpsock:send(reply)
end
 
function _M.set(tcpsock, res)
    local reply =  ""
 
    local key = res[1]
    local flags = res[2]
    local exptime = res[3]
    local bytes = res[4]
 
    local value, err = tcpsock:receive(tonumber(bytes) + 2)
 
    if str_sub(value, -2, -1) == "\r\n" then
        local succ, err, forcible = mc_shdict:set(key, str_sub(value, 1, bytes), exptime, flags)
        if succ then
            reply = reply .. "STORED\r\n"
        else
            reply = reply .. "SERVER_ERROR " .. err .. "\r\n"
        end
    else
        reply = reply .. "ERROR\r\n"
    end
 
    tcpsock:settimeout(1000)  -- one second timeout
    local bytes, err = tcpsock:send(reply)
end
 
function _M.run()
    local tcpsock = assert(ngx.req.socket(true))
 
    while true do
        tcpsock:settimeout(60000) -- 60 seconds
        local data, err = tcpsock:receive("*l")
 
        local command, args
        if data then
            local from, to, err = re_find(data, [[(\S+)]], "jo")
            if from then
                command = str_sub(data, from, to)
                args = parse_args(data, to + 1)
            end
        end
 
        if args then
            local args_len = #args
            if command == 'get' and args_len > 0 then
                _M.get(tcpsock, args)
            elseif command == "set" and args_len == 4 then
                _M.set(tcpsock, args)
            end
        end
    end
end
 
return _M
```
## 9.1 测试框架

```bash
$ resty -e 'local memcached = require "resty.memcached"
    local memc, err = memcached:new()
 
    memc:set_timeout(1000) -- 1 sec
    local ok, err = memc:connect("127.0.0.1", 11212)
    local ok, err = memc:set("dog", 32)
    if not ok then
        ngx.say("failed to set dog: ", err)
        return
    end
 
    local res, flags, err = memc:get("dog")
    ngx.say("dog: ", res)'
```
这段测试代码，使用 lua-rety-memcached 客户端库发起 connect 和 set 操作，并假设 memcached 的服务端监听本机的 11212 端口。

## 9.2 使用test::nginx测试框架
```perl
use Test::Nginx::Socket::Lua::Stream;
use Test::Nginx::Socket 'no_plan';

run_tests();
 
__DATA__
  
=== TEST 1: basic get and set
--- config
        location /test {
            content_by_lua_block {
                local memcached = require "resty.memcached"
                local memc, err = memcached:new()
                if not memc then
                    ngx.say("failed to instantiate memc: ", err)
                    return
                end
 
                memc:set_timeout(1000) -- 1 sec
                local ok, err = memc:connect("127.0.0.1", 11212)
 
                local ok, err = memc:set("dog", 32)
                if not ok then
                    ngx.say("failed to set dog: ", err)
                    return
                end
 
                local res, flags, err = memc:get("dog")
                ngx.say("dog: ", res)
            }
        }
 
--- stream_config
    lua_shared_dict memcached 100m;
       lua_package_path '/home/workspace/geektime/lualib/lib/?.lua;;';
 
--- stream_server_config
    listen 11212;
    
    content_by_lua_block {
        local m = require("resty.memcached.server")
        m.run()
    }
 
--- request
GET /test
--- response_body
dog: nil
--- error_code: 200
```

