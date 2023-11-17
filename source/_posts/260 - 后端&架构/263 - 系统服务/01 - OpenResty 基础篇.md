
## 0 参考资料
1. [nginx-lua-module 中文版](https://github.com/iresty/nginx-lua-module-zh-wiki)
2. [nginx-lua-module 原版](https://github.com/openresty/lua-nginx-module)
3. API网关：
    1. KONG基于nginx+OpenResty
    2. Envoy
    3. APISIX

### 0.1 课程目录
![](https://raw.githubusercontent.com/BaihlUp/Figurebed/master/2023/202311171102436.png)

## 1 入门篇
### 1.1 初探OpenResty
**OpenResty的三大特性：**
1. 详尽的文档和测试用例
2. 同步非阻塞
3. 动态

> 传统的Web服务器，比如NGINX，如果发生任何的变动，都需要去修改磁盘上的配置文件，然后重新加载才能生效，这是因为没有提供API，来控制运行时的行为。但OpenResty可以使用脚本语言lua来控制逻辑的，动态时Lua天生的优势。

### 1.2 第一个程序
 下边通过使用Lua语言，使用OpenResty启动服务，然后响应打印出“hello world”
 1. 安装OpenResty

可以取官网下载源码进行编译安装（[enter link description here](https://openresty.org/cn/)）
 2. 创建工作目录

创建工作目录，然后目录下创建日志和配置文件保存地方
```
mkdir geektime
cd geektime
mkdir logs/ conf/
```
3. 编写nginx.conf

下边编写一个最简单的nginx.conf配置
```
events {
    worker_connections 1024;
}
 
http {
    server {
        listen 8080;
        lua_code_cache off; #可以实时修改lua，不建议开启，影响性能
        location / {
            content_by_lua '
                ngx.say("hello, world")
            ';
        }
    }
}
```
`lua_code_cache off` 可以在调试的时候使用，但即使设置off，对直接写在配置文件里的Lua代码，或者被"init_by_lua_file/init_worker_by_lua_file"加载的Lua代码无效（都是在启动时一次性加载的），这时仍要使用"-s reload"的方式。

4. 启动openresty

```
openresty -p `pwd` -c conf/nginx.conf
```
如果正常启动，可以使用curl访问下服务
```shell
$ curl -i 127.0.0.1:8080
HTTP/1.1 200 OK
Server: openresty/1.13.6.2
Content-Type: text/plain
Transfer-Encoding: chunked
Connection: keep-alive
 
hello, world
```
可以看到，正常响应“hello， world”

### 1.3 OpenResty CLI
在安装好的openresty下有一个resty，resty命令行工具功能很强大，可以通过 `resty -h` 查看使用手册。
下边是一个使用示例：
```
[root@localhost ~]# resty --shdict='dogs 1m' -e 'local dict = ngx.shared.dogs dict:set("Tom", 56)  print(dict:get("Tom"))'
56
```
这个示例结合了 NGINX 配置和 Lua 代码，一起完成了一个共享内存字典的设置和查询。dogs 1m 是 NGINX 的一段配置，声明了一个共享内存空间，名字是 dogs，大小是 1m；在 Lua 代码中用字典的方式使用共享内存。

### 1.4 OpenRest项目概览
- **NGINX C 模块**

OpenResty 中一共包含了 20 多个 C 模块，我们在本节最开始使用的 openresty -V 中，也可以看到这些 C 模块：
```shell
$ openresty -V
nginx version: openresty/1.13.6.2
built by clang 10.0.0 (clang-1000.10.44.4)
built with OpenSSL 1.1.0h  27 Mar 2018
TLS SNI support enabled
configure arguments: --prefix=/usr/local/Cellar/openresty/1.13.6.2/nginx --with-cc-opt='-O2 -I/usr/local/include -I/usr/local/opt/pcre/include -I/usr/local/opt/openresty-openssl/include' --add-module=../ngx_devel_kit-0.3.0 --add-module=../echo-nginx-module-0.61 --add-module=../xss-nginx-module-0.06 --add-module=../ngx_coolkit-0.2rc3 --add-module=../set-misc-nginx-module-0.32 --add-module=../form-input-nginx-module-0.12 --add-module=../encrypted-session-nginx-module-0.08 --add-module=../srcache-nginx-module-0.31 --add-module=../ngx_lua-0.10.13 --add-module=../ngx_lua_upstream-0.07 --add-module=../headers-more-nginx-module-0.33 --add-module=../array-var-nginx-module-0.05 --add-module=../memc-nginx-module-0.19 --add-module=../redis2-nginx-module-0.15 --add-module=../redis-nginx-module-0.3.7 --add-module=../ngx_stream_lua-0.0.5 --with-ld-opt='-Wl,-rpath,/usr/local/Cellar/openresty/1.13.6.2/luajit/lib -L/usr/local/lib -L/usr/local/opt/pcre/lib -L/usr/local/opt/openresty-openssl/lib' --pid-path=/usr/local/var/run/openresty.pid --lock-path=/usr/local/var/run/openresty.lock --conf-path=/usr/local/etc/openresty/nginx.conf --http-log-path=/usr/local/var/log/nginx/access.log --error-log-path=/usr/local/var/log/nginx/error.log --with-pcre-jit --with-ipv6 --with-stream --with-stream_ssl_module --with-stream_ssl_preread_module --with-http_v2_module --without-mail_pop3_module --without-mail_imap_module --without-mail_smtp_module --with-http_stub_status_module --with-http_realip_module --with-http_addition_module --with-http_auth_request_module --with-http_secure_link_module --with-http_random_index_module --with-http_geoip_module --with-http_gzip_static_module --with-http_sub_module --with-http_dav_module --with-http_flv_module --with-http_mp4_module --with-http_gunzip_module --with-threads --with-dtrace-probes --with-stream --with-stream_ssl_module --with-http_ssl_module
```
这里--add-module=后面跟着的，就是 OpenResty 的 C 模块。其中，最核心的就是 lua-nginx-module 和 stream-lua-nginx-module，前者用来处理七层流量，后者用来处理四层流量。

- **lua-resty- 周边库**

OpenResty 官方仓库中包含 18 个 lua-resty-* 库，涵盖 Redis、MySQL、memcached、websocket、dns、流量控制、字符串处理、进程内缓存等常用库。除了官方自带的之外，还有更多的第三方库。它们非常重要，所以下一章节，我们会花更多的篇幅来专门介绍这些周边库。

### 1.5 第三方包管理工具
- OPM

OPM是OpenResty自带的包管理器，在安装好OpenResty之后，可以直接使用，如下搜索一个http的库
```
$ opm search lua-resty-http
ledgetech/lua-resty-http                          Lua HTTP client cosocket driver for OpenResty/ngx_lua
pintsized/lua-resty-http                          Lua HTTP client cosocket driver for OpenResty/ngx_lua
agentzh/lua-resty-http                            Lua HTTP client cosocket driver for OpenResty/ngx_lua
```
OPM 使用了贡献者的 GitHub 仓库地址作为包名，即 GitHub ID / repo name。上面返回了三个 lua-resty-http 第三方库。
[OPM 的网站](https://opm.openresty.org/)上并没有提供包的下载次数，也没有这个包的依赖关系。

- **LUAROCKS**

[LUAROCKS](https://luarocks.org/) 是 OpenResty 世界的另一个包管理器，诞生在 OPM 之前。不同于 OPM 里只包含 OpenResty 相关的包，LuaRocks 里面还包含 Lua 世界的库。
```
$ luarocks search lua-resty-http
```
这次只返回了一个包，可以看下[包的详细信息](https://luarocks.org/modules/pintsized/lua-resty-http)
这里面包含了作者、License、GitHub 地址、下载次数、功能简介、历史版本、依赖等。和 OPM 不同的是，LuaRocks 并没有直接使用 GitHub 的用户信息，而是需要开发者单独在 LuaRocks 上进行注册。
其实，开源的 API 网关项目 Kong，就是使用 LuaRocks 来进行包的管理。

- **AWESOME-RESTY**

讲了这么多包管理的内容，其实呢，即使有了 OPM 和 LuaRocks，对于 OpenResty 的 lua-resty 包，我们还是管中窥豹的状态。到底有没有地方可以让我们一览全貌呢？
[awesome-resty](https://github.com/bungle/awesome-resty) 这个项目，维护了几乎所有 OpenResty 可用的包，并且都分门别类地整理好了。当你不确定是否存在适合的第三方包时，来这里“按图索骥”，可以说是最好的办法。

还是以 HTTP 库为例， 在 awesome-resty 中，它自然是属于 [networking](https://github.com/bungle/awesome-resty#networking) 分类：
```
lua-resty-http by @pintsized — Lua HTTP client cosocket driver for OpenResty / ngx_lua
lua-resty-http by @liseen — Lua http client driver for the ngx_lua based on the cosocket API
lua-resty-http by @DorianGray — Lua HTTP client driver for ngx_lua based on the cosocket API
lua-resty-http-simple — Simple Lua HTTP client driver for ngx_lua
lua-resty-httpipe — Lua HTTP client cosocket driver for OpenResty / ngx_lua
lua-resty-httpclient — Nonblocking Lua HTTP Client library for aLiLua & ngx_lua
lua-httpcli-resty — Lua HTTP client module for OpenResty
lua-resty-requests — Yet Another HTTP Library for OpenResty
```

### 1.6 OpenResty开源项目推荐
- **OPM**

[OPM](https://github.com/openresty/opm/) 做为OpenResty的包管理器，同时也是一个可以学习的项目示例。opm 是 OpenResty 中为数不多的网站类项目，而里面的代码，基本上是由 OpenResty 的作者亲自操刀完成的。

- OpenResty网站

[OpenResty网站](https://github.com/openresty/openresty.org) 网站也是一个开源的OpenResty项目

- **lua-nginx-module**

[lua-nginx-module](https://github.com/openresty/lua-nginx-module) 是OpenResty中比较常用的一个库。

### 1.7 OpenResty 中用到的 NGINX 知识
OpenResty 的两个基石：NGINX 和 LuaJIT
- **MASTER-WORKER模式**

下边是nginx中的master-worker模型，master是个“管理者”的角色，并不负责处理终端的请求，它是用来管理Worker进程，包括接受管理员发送的信号量、监控Worker的运行状态。当 Worker 进程异常退出时，Master 进程会重新启动一个新的 Worker 进程。
Worker 进程则是“一线员工”，用来处理终端用户的请求。它是从 Master 进程 fork 出来的，彼此之间相互独立，互不影响。
![](https://raw.githubusercontent.com/BaihlUp/Figurebed/master/2023/202311171103179.png)

而 OpenResty 在 NGINX Master-Worker 模式的前提下，又增加了独有的特权进程（privileged agent）。这个进程并不监听任何端口，和 NGINX 的 Master 进程拥有同样的权限，所以可以做一些需要高权限才能完成的任务，比如对本地磁盘文件的一些写操作等。

- **执行阶段**

下边是nginx在ngx_http_core_module.h中定义的11个阶段
```c
typedef enum {
    NGX_HTTP_POST_READ_PHASE = 0,
 
    NGX_HTTP_SERVER_REWRITE_PHASE,
 
    NGX_HTTP_FIND_CONFIG_PHASE,
    NGX_HTTP_REWRITE_PHASE,
    NGX_HTTP_POST_REWRITE_PHASE,
 
    NGX_HTTP_PREACCESS_PHASE,
 
    NGX_HTTP_ACCESS_PHASE,
    NGX_HTTP_POST_ACCESS_PHASE,
 
    NGX_HTTP_PRECONTENT_PHASE,
 
    NGX_HTTP_CONTENT_PHASE,
 
    NGX_HTTP_LOG_PHASE
} ngx_http_phases;
```

OpenResty 也有 11 个 `*_by_lua` 指令，它们和 NGINX 阶段的关系如下图所示（图片来自 lua-nginx-module 文档）：
![](https://i.loli.net/2020/09/10/rmH87gfUyoCnMsw.png)
其中， `init_by_lua` 只会在 Master 进程被创建时执行，`init_worker_by_lua` 只会在每个 Worker 进程被创建时执行。其他的 `*_by_lua` 指令则是由终端请求触发，会被反复执行。

> NGINX 支持的功能，OpenResty 并不一定支持，需要看 OpenResty 的版本号

### 1.8 快速上手Lua
#### 1.8.1 执行Hello World
在安装完OpenResty后，同时也安装了luajit 和 Resty，luajit 是OpenResty维护的一个lua的解释器，下边细说，现在先执行一个简单的程序：
```
$ cat 1.lua
print("hello world")
 
$ luajit 1.lua
 hello world
```
下边使用resty执行，它最终其实也是用LuaJIT 来执行的：
```
$ resty -e 'print("hello world")'
 hello world
```

#### 1.8.2 数据类型
下边使用type函数打印下lua中常见的数据类型：
```
$ resty -e 'print(type("hello world")) 
 print(type(print)) 
 print(type(true)) 
 print(type(360.0))
 print(type({}))
 print(type(nil))
 '
```
输出
```
 string
 function
 boolean
 number
 table
 nil
```

- **字符串**

在 Lua 中，字符串是不可变的值，如果你要修改某个字符串，就等于创建了一个新的字符串。这种做法显然有利有弊：好处是即使同一个字符串出现了很多次，在内存中也只有一份；但劣势也很明显，如果你想修改、拼接字符串，会额外地创建很多不必要的字符串。
```
$ resty -e 'local s  = ""
 for i = 1, 10 do
     s = s .. tostring(i)
 end
 print(s)'
```
这里我们循环了 10 次，但只有最后一次是我们想要的，而中间新建的 9 个字符串都是无用的。它们不仅占用了额外的空间，也消耗了不必要的 CPU 运算。
另外，在 Lua 中，你有三种方式可以表达一个字符串：单引号、双引号，以及长括号（[[]]）。
```
$ resty -e 'print([[string has \n and \r]])'
 string has \n and \r
```
可以看到，长括号中的字符串不会做任何的转义处理。
如果字符串中包括了长括号本身，需要在长括号中间增加一个或者多个 = 符号：
```
$ resty -e 'print([=[ string has a [[]]. ]=])'
  string has a [[]].
```
- **布尔值**

在lua中，**只有 nil 和 false 为假，其他都为真，包括 0 和空字符串也为真**。

- **数字**

Lua 的 `number` 类型，是用双精度浮点数来实现的。值得一提的是，LuaJIT 支持 `dual-number`（双数）模式，也就是说， LuaJIT 会根据上下文来用整型来存储整数，而用双精度浮点数来存放浮点数。

此外，LuaJIT 还支持**长长整型**的大整数，比如下面的例子：
```
$ resty -e 'print(9223372036854775807LL - 1)'
9223372036854775806LL
```

- **函数**

函数在 Lua 中是一等公民，你可以把函数存放在一个变量中，也可以当作另外一个函数的入参和出参。
下面两个函数的声明是完全等价的：
```
function foo()
 end
```
和
```
foo = function ()
 end
```

 - **table**

table 是 Lua 中唯一的数据结构。下边是示例：
```
$ resty -e 'local color = {first = "red"}
print(color["first"])'
 red
```

- **空值**

在 Lua 中，空值就是 nil。如果你定义了一个变量，但没有赋值，它的默认值就是 nil：
```
$ resty -e 'local a
 print(type(a))'
 nil
```
当你真正进入 OpenResty 体系中后，会发现很多种空值，比如 `ngx.null` 等等，我们后面再细聊。

#### 1.8.3 常用标准库
下边介绍几个Lua中原生的标准库，但是在OpenResty中，Lua库的优先级是最低的。对于同一个功能，我更推荐你优先使用 OpenResty 的 API 来解决，然后是 LuaJIT 的库函数，最后才是标准 Lua 的函数。
**OpenResty的API > LuaJIT的库函数 > 标准Lua的函数**

- **string 库**

有一个简单的原则，那就是如果涉及到正则表达式的，请一定要使用 OpenResty 提供的 `ngx.re.*` 来解决，不要用 Lua 的 `string.*` 处理。这是因为，Lua 的正则独树一帜，不符合 PCRE 的规范，我相信绝大部分工程师是玩不转的。

其中 `string.byte(s [, i [, j ]])`，是比较常用到的一个 string 库函数，它返回字符 `s[i]、s[i + 1]、s[i + 2]、······、s[j]` 所对应的 ASCII 码。i 的默认值为 1，即第一个字节，j 的默认值为 i。
示例代码：
```
$ resty -e 'print(string.byte("abc", 1, 3))
 print(string.byte("abc", 3)) -- 缺少第三个参数，第三个参数默认与第二个相同，此时为 3
 print(string.byte("abc"))    -- 缺少第二个和第三个参数，此时这两个参数都默认为 1
 '
```
输出
```
 979899
 99
 97
```

- **table 库**

在 OpenResty 的上下文中，对于 Lua 自带的 table 库，除了 `table.concat` 、`table.sort` 等少数几个函数，大部分我都不推荐使用。
`table.concat`一般用在字符串拼接的场景下，比如下面这个例子。它可以避免生成很多无用的字符串。
```
$ resty -e 'local a = {"A", "b", "C"}
 print(table.concat(a))'
```

- **math 库**

`math.random()` 和 `math.randomseed()` 两个函数比较常用，比如下面的这段代码，它可以在指定的范围内，随机地生成两个数字。
```
$ resty -e 'math.randomseed (os.time()) 
print(math.random())
 print(math.random(100))'
```
- **虚变量**

在一个函数返回多个变量时，我们可以不需要接收某些返回值，这时候可以使用虚变量的方式接收，如下使用 `string.find` 这个标准库函数为例，这个标准库函数会返回两个值，分别代表开始和结束的下标。
如果我们只需要获取开始的下标，那么很简单，只声明一个变量来接收 string.find 的返回值即可：
```
$ resty -e 'local start = string.find("hello", "he")
 print(start)'
 1
```

但如果你只想获取结束的下标，那就必须使用虚变量了：
```
$ resty -e 'local  _, end_pos = string.find("hello", "he")
 print(end_pos)'
 2
```

除了在返回值里使用，虚变量还经常用于循环中，比如下面这个例子：
```
$ resty -e 'for _, v in ipairs({4,5,6}) do
     print(v)
 end'
 4
 5
 6
```

### 1.9 LuaJIT 分支和 Lua
#### 1.9.1 LuaJIT
先来看下LuaJIT 在OpenResty 整体架构中的位置：
![](https://i.loli.net/2020/09/11/y4GX9EJgubIdVKL.png)
OpenResty 的 worker 进程都是 fork master 进程而得到的， 其实， master 进程中的 LuaJIT 虚拟机也会一起 fork 过来。在同一个 worker 内的所有协程，都会共享这个 LuaJIT 虚拟机，Lua 代码的执行也是在这个虚拟机中完成的。

- **Lua 和 LuaJIT的区别**

标准 Lua 和 LuaJIT 是两回事儿，LuaJIT 只是兼容了 Lua 5.1 的语法。
所谓 LuaJIT 的性能优化，本质上就是让尽可能多的 Lua 代码可以被 JIT 编译器生成机器码，而不是回退到 Lua 解释器的解释执行模式。

LuaJIT 除了兼容了Lua5.1的语法外，还紧密结合了 FFI（Foreign Function Interface），可以让你直接在 Lua 代码中调用外部的 C 函数和使用 C 的数据结构。

示例：
```
local ffi = require("ffi")
ffi.cdef[[
int printf(const char *fmt, ...);
]]
ffi.C.printf("Hello %s!", "world")
```
短短这几行代码，就可以直接在 Lua 中调用 C 的 printf 函数，打印出 Hello world!。你可以使用 resty 命令来运行它，看下是否成功。
类似的，我们可以用 FFI 来调用 NGINX、OpenSSL 的 C 函数，来完成更多的功能。实际上，FFI 方式比传统的 Lua/C API 方式的性能更优，这也是 lua-resty-core 项目存在的意义。下一节我们就来专门讲讲 FFI 和 lua-resty-core。

#### 1.9.2 Lua 特别之处
1. Lua的下标是从1开始
2. 使用 .. 来拼接字符串
3. 只有table数据结构
4. 默认是全局变量，需要local 定义局部变量

#### 1.9.3 Lua 独有概念
- **弱表**

弱表（weak table），它是 Lua 中很独特的一个概念，和垃圾回收相关。
举个例子，我们把一个 Lua 的对象 Foo（table 或者函数）插入到 table tb 中，这就会产生对这个对象 Foo 的引用。即使没有其他地方引用 Foo，tb 对它的引用也还一直存在，那么 GC 就没有办法回收 Foo 所占用的内存。这时候，我们就只有两种选择：
1. 一是手工释放 Foo；
2. 二是让它常驻内存。

比如下边的代码：
```
$ resty -e 'local tb = {}
tb[1] = {red}
tb[2] = function() print("func") end
print(#tb) -- 2
 
collectgarbage()
print(#tb) -- 2
 
table.remove(tb, 1)
print(#tb) -- 1
```
下边使用弱表优化：
```
$ resty -e 'local tb = {}
tb[1] = {red}
tb[2] = function() print("func") end
setmetatable(tb, {__mode = "v"})
print(#tb)  -- 2
 
collectgarbage()
print(#tb) -- 0
'
```
可以看到，没有被使用的对象都被 GC 了。这其中，最重要的就是下面这一行代码：
```
setmetatable(tb, {__mode = "v"})
```
当一个 table 的元表中存在 __mode 字段时，这个 table 就是弱表（weak table）了。

1. 如果 __mode 的值是 k，那就意味着这个 table 的 键 是弱引用。
2. 如果 __mode 的值是 v，那就意味着这个 table 的 值 是弱引用。
3. 当然，你也可以设置为 kv，表明这个表的键和值都是弱引用。

这三者中的任意一种弱表，只要它的 键 或者 值 被回收了，那么对应的整个键值 对象都会被回收。

- **闭包 和 upvalue**

示例：
```
$ resty -e '
local function foo()
     local i = 1
     local function bar()
         i = i + 1
         print(i)
     end
     return bar
end
 
local fn = foo()
print(fn()) -- 2
'
```
bar 这个函数可以读取函数 foo 里面的局部变量 i，并修改它的值，即使这个变量并不在 foo 里面定义。这个特性叫做词法作用域（lexical scoping）。

事实上，Lua 的这些特性正是闭包的基础。所谓闭包 ，简单地理解，它其实是一个函数，不过它访问了另外一个函数词法作用域中的变量。
实际上，upvalue 就是闭包中捕获的自己词法作用域外的那个变量。还是继续看上面那段代码：

```lua
local foo, bar
local function fn()
     foo = 1
     bar = 2
end
```
你可以看到，函数 fn 捕获了两个不在自己词法作用域的局部变量 foo 和 bar，而这两个变量，实际上就是函数 fn 的 upvalue。

- **变量的个数限制**

 Lua 中，一个函数的局部变量的个数，和 upvalue 的个数都是有上限的，你可以从 Lua 的源码中得到印证：
```
/*
@@ LUAI_MAXVARS is the maximum number of local variables per function
@* (must be smaller than 250).
*/
#define LUAI_MAXVARS            200
 
 
/*
@@ LUAI_MAXUPVALUES is the maximum number of upvalues per function
@* (must be smaller than 250).
*/
#define LUAI_MAXUPVALUES        60
```
分别被硬编码为 200 和 60。虽说你可以手动修改源码来调整这两个值，不过最大也只能设置为 250。
我们不会超过这个阈值，但写 OpenResty 代码的时候，你还是要留意这个事情，不要过多地使用局部变量和 upvalue，而是要尽可能地使用 do .. end 做一层封装，来减少局部变量和 upvalue 的个数。
```
local re_find = ngx.re.find
  function foo() ... end
function bar() ... end
function fn() ... end
```

#### 1.9.4 面向对象
lua-resty-mysql 是 OpenResty 官方的 MySQL 客户端，里面就使用元表模拟了类和类方法，它的使用方式如下所示：
```
$ resty -e 'local mysql = require "resty.mysql" -- 先引用 lua-resty 库
local db, err = mysql:new() -- 新建一个类的实例
db:set_timeout(1000) -- 调用类的方法'
```
在这里冒号和点号都是可以的，`db:set_timeout(1000)` 和 `db.set_timeout(db, 1000)` 是完全等价的。冒号是 Lua 中的一个语法糖，可以省略掉函数的第一个参数 self。
下边看下具体实现：

```lua
local _M = { _VERSION = '0.21' } -- 使用 table 模拟类
local mt = { __index = _M } -- mt 即 metatable 的缩写，__index 指向类自身
 
-- 类的构造函数
function _M.new(self) 
     local sock, err = tcp()
     if not sock then
         return nil, err
     end
     return setmetatable({ sock = sock }, mt) -- 使用 table 和 metatable 模拟类的实例
end
 
-- 类的成员函数
 function _M.set_timeout(self, timeout) -- 使用 self 参数，获取要操作的类的实例
     local sock = self.sock
     if not sock then
        return nil, "not initialized"
     end
 
    return sock:settimeout(timeout)
end
```
你可以看到，`_M` 这个 table 模拟了一个类，初始化时，它只有 _VERSION 这一个成员变量，并在随后定义了 `_M.set_timeout` 等成员函数。在 `_M.new(self)` 这个构造函数中，我们返回了一个 table，这个 table 的元表就是 mt，而 mt 的 `__index` 元方法指向了 `_M`，这样，返回的这个 table 就模拟了类 `_M` 的实例。

### 1.10 剖析Lua 唯一的数据结构table 和 metatable 特性
#### 1.10.1 table 库函数
- **table.getn 获取元素个数**

在lua中获取table中的元素个数是一个比较难的问题，在**序列中**，用table.getn 或者一元操作符 # ，可以正确返回元素的个数。
```
[root@localhost bin]# resty -e 'local t = { 1, 2, 3 }
> print(table.getn(t)) '
3
```
在 OpenResty 的环境下，除非你明确知道，你正在获取序列的长度，否则请不要使用函数 table.getn 和一元操作符 # 。
> `table.getn` 和一元操作符 # 并不是 O(1) 的时间复杂度，而是 O(n)，这也是尽量避免使用它们的另外一个理由。

- **table.remove 删除指定元素**

它的作用是在 table 中根据下标来删除元素，也就是说只能删除 table 中数组部分的元素。我们还是来看color的例子：
```
resty -e 'local color = {first = "red", "blue", third = "green", "yellow"}
  table.remove(color, 1)
  for k, v in pairs(color) do
      print(v)
  end'
```
这段代码会把下标为 1 的 blue 删除掉。删除table中的哈希部分，可以直接把key对应的value 设置为 nil。

- **table.concat 元素拼接函数**

它可以按照下标，把 table 中的元素拼接起来。既然这里又是根据下标来操作的，那么显然还是针对 table 的数组部分。
```
resty -e 'local color = {first = "red", "blue", third = "green", "yellow"}
print(table.concat(color, ", "))'
```
使用table.concat函数后，它输出的是 `blue, yellow`，哈希的部分被跳过了
另外，这个函数还可以指定下标的起始位置来做拼接，比如下面这样的写法：
```
$ resty -e 'local color = {first = "red", "blue", third = "green", "yellow", "orange"}
print(table.concat(color, ", ", 2, 3))'
```
这次输出是 `yellow, orange`，跳过了 blue。

- **table.insert 插入一个元素**

它可以下标插入一个新的元素，自然，影响的还是 table 的数组部分。还是用color例子来说明：
```
 resty -e 'local color = {first = "red", "blue", third = "green", "yellow"}
table.insert(color, 1,  "orange")
print(color[1])
'
```
以上输出 orange，可以发现color的第一个元素变为了 orange。当然，你也可以不指定下标，这样就会默认插入队尾。

#### 1.10.2 LuaJIT 的 table 扩展函数
- **table.new(narray, nhash) 新建 table**

第一个是`table.new(narray, nhash)` 函数。这个函数，会预先分配好指定的数组和哈希的空间大小，而不是在插入元素时自增长，这也是它的两个参数 narray 和 nhash 的含义。自增长是一个代价比较高的操作，会涉及到空间分配、resize 和 rehash 等，我们应该尽量避免。
示例：
```
local new_tab = require "table.new"
local t = new_tab(100, 0)
for i = 1, 100 do
   t[i] = i
end
```
你可以看到，这段代码新建了一个 table，里面包含 100 个数组元素和 0 个哈希元素。当然，你也可以根据实际需要，新建一个同时包含 100 个数组元素和 50 个 哈希元素的 table，这都是合法的：
```
local t = new_tab(100, 50)
```
另外，超出预设的空间大小，也可以正常使用，只不过性能会退化，也就失去了使用 `table.new` 的意义。
> 参考资料：[table.new](https://blog.csdn.net/senlin1202/article/details/86021545)

- **table.clear() 清空 table**

它用来清空某个 table 里的所有数据，但并不会释放数组和哈希部分占用的内存。所以，它在循环利用 Lua table 时非常有用，可以避免反复创建和销毁 table 的开销。
```
$ resty -e 'local clear_tab =require "table.clear"
local color = {first = "red", "blue", third = "green", "yellow"}
clear_tab(color)
for k, v in pairs(color) do
     print(k)
end'
```

#### 1.10.3 OpenResty 的 table 扩展函数
OpenResty 自己维护的 LuaJIT 分支，也对 table 做了扩展，它[新增了几个 API](https://github.com/openresty/luajit2/#new-api)：`table.isempty、table.isarray、 table.nkeys 和 table.clone`。

下边使用 `table.nkeys` 示例：
```
local nkeys = require "table.nkeys"
 
print(nkeys({}))  -- 0
print(nkeys({ "a", nil, "b" }))  -- 2
print(nkeys({ dog = 3, cat = 4, bird = nil }))  -- 2
print(nkeys({ "a", dog = 3, cat = 4 }))  -- 3
```
table.nkeys函数，返回的是 table 的元素个数，包括数组和哈希部分的元素。

#### 1.10.4 元表
元表是 Lua 中独有的概念，在实际项目中的使用非常广泛。不夸张地说，在几乎所有的 `lua-resty-*` 库中，你都能看到它的身影。

 Lua 提供了两个处理元表的函数：
- 第一个是setmetatable(table, metatable), 用于为一个 table 设置元表；
- 第二个是getmetatable(table)，用于获取 table 的元表。

使用示例：
```
$ resty -e ' local version = {
  major = 1,
  minor = 1,
  patch = 1
  }
version = setmetatable(version, {
    __tostring = function(t)
      return string.format("%d.%d.%d", t.major, t.minor, t.patch)
    end
  })
  print(tostring(version))
'
```
首先定义了一个 名为 version的 table ，你可以看到，这段代码的目的，是想把 version 中的版本号打印出来。
所以，我们需要自定义这个 table 的字符串转换函数，也就是 `__tostring`，到这一步也就是元表的用武之地了。我们用 setmetatable ，重新设置 version 这个 table 的 `__tostring` 方法，就可以打印出版本号: 1.1.1。

 -  **重载元表中的元方法**

示例：
```
$ resty -e ' local version = {
  major = 1,
  minor = 1
  }
version = setmetatable(version, {
     __index = function(t, key)
         if key == "patch" then
             return 2
         end
     end,
     __tostring = function(t)
      return string.format("%d.%d.%d", t.major, t.minor, t.patch)
    end
  })
  print(tostring(version))
'
```
这样的话，`t.patch` 其实获取不到值，那么就会走到 `__index` 这个函数中，结果就会打印出 1.1.2。

事实上，__index 不仅可以是一个函数，也可以是一个 table。你试着运行下面这段代码，就会看到，它们实现的效果是一样的。
```
$ resty -e ' local version = {
  major = 1,
  minor = 1
  }
version = setmetatable(version, {
     __index = {patch = 2},
     __tostring = function(t)
      return string.format("%d.%d.%d", t.major, t.minor, t.patch)
    end
  })
  print(tostring(version))
'
```

- 元方法`__call`

示例：

```
$ resty -e '
local version = {
  major = 1,
  minor = 1,
  patch = 1
  }
 
local function print_version(t)
     print(string.format("%d.%d.%d", t.major, t.minor, t.patch))
end
 
version = setmetatable(version,
     {__call = print_version})
 
  version()
'
```
这段代码中，我们使用 `setmetatable`，给 version 这个 table 增加了元表，而里面的 `__call` 元方法指向了函数 `print_version` 。那么，如果我们尝试把 `version` 当作函数调用，这里就会执行函数 `print_version`。

而 `getmetatable` 是和 `setmetatable` 配对的操作，可以获取到已经设置的元表，比如下面这段代码：

```
$ resty -e ' local version = {
  major = 1,
  minor = 1
  }
version = setmetatable(version, {
     __index = {patch = 2},
     __tostring = function(t)
      return string.format("%d.%d.%d", t.major, t.minor, t.patch)
    end
  })
  print(getmetatable(version).__index.patch)
'
```
> 元方法参考：http://lua-users.org/wiki/MetamethodsTutorial



### 1.11 答疑
#### 1.11.1 关于空值的困惑
Q：我遇到一些让人困惑的地方是ngx.null、nil、null和""。在网上搜索的时候，看到有人说null是ngx.null的一个定义。Redis 返回的时候，经常会判断返回结果是否为空，那么，判断的时候是和哪个值进行比较呢？关于这些值，有没有其他一些使用上的坑呢？一直以来我都没有一个明确的认识，想和老师确认一下。

A：在回答你的问题之前，我建议你在 lua-resty-redis 里，使用下面的代码去查找一个 key：
```
local res, err = red:get("dog")
```
如果返回值 res 是 nil，就说明函调用失败了；如果 res 是 ngx.null ，就说明 redis 中不存在 dog 这个 key。这是因为， Lua 的 nil 无法作为 table 的 value，所以 OpenResty 引入了 `ngx.null`，作为 table 中的空值。
我们可以用下面的代码，打印出 ngx.null 和它的类型：
```
# 打印 ngx.null
$ resty -e  'print(ngx.null)'
null
 
# 打印类型
$ resty -e 'print(type(ngx.null))'
userdata
```
你可以看到， ngx.null 并非nil，而是 userdata 类型。

#### 1.11.2 配置文件的规则优先级
Q：当 OpenResty 中的 Lua 规则和 NGINX 配置文件产生冲突时，比如 NGINX 配置了 rewrite 规则，又同时引用了 rewrite_by_lua_file，那么这两条规则的优先级是什么？

A：其实，这个具体要看 NGINX 配置的 rewrite 规则是怎么写的了，是 break 还是 last。这一点，在 OpenResty 的官方文档中有注明，并且配了一个示例代码：

```
 location /foo {
     rewrite ^ /bar;
     rewrite_by_lua 'ngx.exit(503)';
 }
 location /bar {
     ...
 }
```
在示例代码的这个配置中，ngx.exit(503) 是不会被执行的。

但是，如果你改成下面这样的写法，ngx.exit(503) 就可以被执行。
```
rewrite ^ /bar break；
```

## 2 API 篇
### 2.1 OpenResty开发平台
#### 2.1.1 协程和事件驱动
在 OpenResty 层面，Lua 的协程会与 NGINX 的事件机制相互配合。如果 Lua 代码中出现类似查询 MySQL 数据库这样的 I/O 操作，就会先调用 Lua 协程的 yield 把自己挂起，然后在 NGINX 中注册回调；在 I/O 操作完成（也可能是超时或者出错）后，再由 NGINX 回调 resume 来唤醒 Lua 协程。这样就完成了 Lua 协程和 NGINX 事件驱动的配合，避免在 Lua 代码中写回调。
我们可以来看下面这张图，描述了这整个流程。其中，`lua_yield` 和 `lua_resume` 都属于 Lua 提供的 `lua_CFunction`。
![](https://i.loli.net/2020/09/14/sdrcA42uWpTaQ9x.png)

下面是 `ngx.sleep` 的一段源码，可以帮你更清晰理解这一点。 这段代码位于 `ngx_http_lua_sleep.c` 中，你可以在 `lua-nginx-module` 项目的 [src](https://github.com/openresty/lua-nginx-module/tree/master/src) 目录中找到它。
在`ngx_http_lua_sleep.c` 中，我们可以看到 sleep 函数的具体实现。你需要先通过 C 函数 `ngx_http_lua_ngx_sleep`，来注册 `ngx.sleep` 这个 Lua API：

```
void
ngx_http_lua_inject_sleep_api(lua_State *L)
{
     lua_pushcfunction(L, ngx_http_lua_ngx_sleep);
     lua_setfield(L, -2, "sleep");
}
```
下面便是 sleep 的主函数，这里我只摘取了几行主要的代码：

```
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

当 sleep 操作完成后， `ngx_http_lua_sleep_handler` 这个回调函数就被触发了。它里面调用了 `ngx_http_lua_sleep_resume`, 并最终使用 `lua_resume` 唤醒了 Lua 协程。更具体的调用过程，你可以自己去代码里面检索，这里我就不展开描述了。

#### 2.1.2 OpenResty 的阶段
- `set_by_lua`，用于设置变量；
- `rewrite_by_lua`，用于转发、重定向等；
- `access_by_lua`，用于准入、权限等；
- `content_by_lua`，用于生成返回内容；
- `header_filter_by_lua`，用于应答头过滤处理；
- `body_filter_by_lua`，用于应答体过滤处理；
- `log_by_lua`，用于日志记录。

在编写模块时，不同的API在使用前应该弄清楚这个API的context，如果使用不当会报错。如下示例：
以`ngx.sleep` 为例，通过查阅文档，我知道它只能用于下面列出的上下文中，并不包括 log 阶段：

```
context: rewrite_by_lua*, access_by_lua*, content_by_lua*, ngx.timer.*, ssl_certificate_by_lua*, ssl_session_fetch_by_lua*_
```
而如果你不知道这一点，在它不支持的 log 阶段使用 sleep 的话：

```
location / {
    log_by_lua_block {
        ngx.sleep(1)
     }
}
```
在 NGINX 的错误日志中，就会出现 error 级别的提示：

```
[error] 62666#0: *6 failed to run log_by_lua*: log_by_lua(nginx.conf:14):2: API disabled in the context of log_by_lua*
stack traceback:
    [C]: in function 'sleep'
```

### 2.2 文档和测试案例
在学习 OpenResty 时一定要多看文档和测试用例，在模块的 `t` 目录下有模块的完整用例集，可以方便测试和学习。
下边简单以`shdict get API` 为例：
[文档链接](https://github.com/openresty/lua-nginx-module/#ngxshareddictget) 为对照。下边是一个get方法：

```
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
```
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

### 2.3 OpenResty 如何处理终端请求和响应
#### 2.3.1 API 分类
- OpenResty 的 API 主要分为下面几个大类：
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

#### 2.3.2 请求行
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

```
[root@localhost]#resty -e 'print(ngx.HTTP_POST)'
8
```
- `ngx.req.set_uri` 和 `ngx.req.set_uri_args` 这两个 API，可以用来改写 uri 和 args。

#### 2.3.3 请求头
`ngx.req.get_headers` 来解析和获取请求头，返回值的类型则是 table：

```
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

#### 2.3.4 请求体
出于性能考虑，OpenResty 不会主动读取请求体的内容，除非你在 nginx.conf 中强制开启了 `lua_need_request_body` 指令。此外，对于比较大的请求体，OpenResty 会把内容保存在磁盘的临时文件中，所以读取请求体的完整流程是下面这样的：

```
ngx.req.read_body()
local data = ngx.req.get_body_data()
if not data then
    local tmp_file = ngx.req.get_body_file()
     -- io.open(tmp_file)
     -- ...
 end
```


#### 2.3.5 响应状态
在默认情况下，返回的 HTTP 状态码是 200，也就是 OpenResty 中内置的常量 ngx.HTTP_OK。
OpenResty 的 HTTP 状态码中，有一个特别的常量：ngx.OK。当 ngx.exit(ngx.OK) 时，请求会退出当前处理阶段，进入下一个阶段，而不是直接返回给客户端。

```
ngx.exit(ngx.HTTP_BAD_REQUEST)
```
改写状态码：

```
ngx.status = ngx.HTTP_FORBIDDEN
```
更多的状态码常量，查看[文档](https://github.com/openresty/lua-nginx-module/#http-status-constants)

#### 2.3.6 响应头
响应头，其实，你有两种方法来设置它。第一种是最简单的：
```
ngx.header.content_type = 'text/plain'
ngx.header["X-My-Header"] = 'blah blah'
ngx.header["X-My-Header"] = nil -- 删除
```
这里的 ngx.header 保存了响应头的信息，可以读取、修改和删除。
第二种设置响应头的方法是 `ngx_resp.add_header` ，来自 lua-resty-core 仓库，它可以增加一个头信息，用下面的方法来调用：

```
local ngx_resp = require "ngx.resp"
ngx_resp.add_header("Foo", "bar")
```
与第一种方法的不同之处在于，add header 不会覆盖已经存在的同名字段。

#### 2.3.7 响应体
可以使用 `ngx.say` 和 `ngx.print` 来输出响应体：

```
ngx.say('hello, world')
```
这两个 API 的功能是一致的，唯一的不同在于， `ngx.say` 会在最后多一个换行符。

### 2.4 OpenResty 中数据共享的方式
OpenResty中的共享内存字典 shared dict，是最重要的数据结构。它不仅支持数据的存放和读取，还支持原子计数和队列操作。
基于 shared dict，你可以实现多个 worker 之间的缓存和通信，以及限流限速、流量统计等功能。你可以把 shared dict 当作简单的 Redis 来使用，只不过 shared dict 中的数据不能持久化，所以你存放在其中的数据，一定要考虑到丢失的情况。
除了shared dict 还有其他几种数据共享的方式。

#### 2.4.1 Nginx 中的变量
它可以在 Nginx C 模块之间共享数据，自然的，也可以在 C 模块和 OpenResty 提供的 `lua-nginx-module` 之间共享数据，比如下面这段代码：

```
location /foo {
     set $my_var ''; # this line is required to create $my_var at config time
     content_by_lua_block {
         ngx.var.my_var = 123;
         ...
     }
 }
```
不过，使用 Nginx 变量这种方式来共享数据是比较慢的，因为它涉及到 hash 查找和内存分配。同时，这种方法有其局限性，只能用来存储字符串，不能支持复杂的 Lua 类型。
#### 2.4.2 ngx.ctx
可以在同一个请求的不同阶段之间共享数据。它其实就是一个普通的 Lua 的 table，所以速度很快，还可以存储各种 Lua 的对象。它的生命周期是请求级别的，当一个请求结束的时候，ngx.ctx 也会跟着被销毁掉。

下面是一个典型的使用场景，我们用 ngx.ctx 来缓存 Nginx 变量 这种昂贵的调用，并在不同阶段都可以使用到它：

```
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

```
curl -i 127.0.0.1:8080/test -H 'host:openresty.org'
```
则响应 `test.com`。
需要注意的是 ngx.ctx 的生命周期是请求级别的，所以并不能在模块级别进行缓存。
如下使用就是错误的：
```
local ngx_ctx = ngx.ctx
 
local function bar()
    ngx_ctx.host =  'test.com'
end
```
应该在函数级别进行调用和缓存：

```
local ngx = ngx
 
local function bar()
	local ngx_ctx = ngx.ctx
    ngx_ctx.host =  'test.com'
end
```

#### 2.4.3 模块级别的变量
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

```
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

#### 2.4.4 shared dict
这些数据可以在多个 worker 之间共享。
这种方法是基于红黑树实现的，性能很好，但也有自己的局限性——你必须事先在 Nginx 的配置文件中，声明共享内存的大小，并且这不能在运行期更改：
```
lua_shared_dict dogs 10m;
```
shared dict 同样只能缓存字符串类型的数据，不支持复杂的 Lua 数据类型。详细API查看[文档](https://github.com/openresty/lua-nginx-module#ngxshareddict)
> 前面三种数据共享的范围都是在请求级别，或者单个 worker 级别。所以，在当前的 OpenResty 的实现中，只有 shared dict 可以完成 worker 间的数据共享，并借此实现 worker 之间的通信，这也是它存在的价值。

shared dict 在实现多个进程共享时，并不需要加锁，因为其中的接口已经实现了原子操作。

 - 读写类

首先来看字典读写类。在最初的版本中，只有字典读写类的 API，它们也是共享字典最常用的功能。下面是一个最简单的示例：

```
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

```
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
```
require "resty.core.shdict"
 
 local cats = ngx.shared.cats
 local capacity_bytes = cats:capacity()
 local free_page_bytes = cats:free_space()
```
它们分别返回的，是共享内存的大小（也就是 lua_shared_dict 中配置的大小）和空闲页的字节数。因为 shared dict 是按照页来分配的，即使 free_space 返回为 0，在已经分配的页面中也可能存在空间，所以它的返回值并不能代表共享内存实际被占用的情况。

### 2.5 cosocket
cosocket 是各种 `lua-resty-*` 非阻塞库的基础，没有 cosocket，开发者就无法用 Lua 来快速连接各种外部的网络服务。

#### 2.5.2 什么是 cosocket？
cosocket 是 OpenResty 中的专有名词，是把协程和网络套接字的英文拼在一起形成的，即 cosocket = coroutine + socket。所以，你可以把 cosocket 翻译为“协程套接字”。
如果我们在 OpenResty 中调用一个 cosocket 相关函数，内部实现便是下面这张图的样子：
![](https://i.loli.net/2020/09/14/3KuNVFmaEhjIlUD.png)
遇到网络 I/O 时，它会交出控制权（yield），把网络事件注册到 Nginx 监听列表中，并把权限交给 Nginx；当有 Nginx 事件达到触发条件时，便唤醒对应的协程继续处理（resume）。

OpenResty 正是以此为蓝图，封装实现 connect、send、receive 等操作，形成了我们如今见到的 cosocket API。下面，我就以处理 TCP 的 API 为例来介绍一下。处理 UDP 和 Unix Domain Socket ，与 TCP 的接口基本是一样的。
#### 2.5.2 cosocket API 和指令简介
TCP 相关的 cosocket API 可以分为下面这几类。
- 创建对象：ngx.socket.tcp。
- 设置超时：tcpsock:settimeout 和 tcpsock:settimeouts。
- 建立连接：tcpsock:connect。
- 发送数据：tcpsock:send。
- 接受数据：tcpsock:receive、tcpsock:receiveany 和 - tcpsock:receiveuntil。
- 连接池：tcpsock:setkeepalive。
- 关闭连接：tcpsock:close。

这些 API 可以使用的上下文：

```
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

```
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

```
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

```
local ok, err = sock:setkeepalive(2 * 1000, 100)
if not ok then
    ngx.say("failed to set reusable: ", err)
end
```
这段代码设置了连接的空闲时间为 2 秒，连接池的大小为 100。这样，在调用 connect() 函数时，就会优先从连接池中获取 cosocket 对象。
关于连接池的使用，有两点需要我们注意一下：
- 第一，不能把发生错误的连接放入连接池，否则下次使用时，就会导致收发数据失败。这也是为什么我们需要判断每一个 API 调用是否成功的一个原因。
- 第二，要搞清楚连接的数量。连接池是 worker 级别的，每个 worker 都有自己的连接池。所以，如果你有 10 个 worker，连接池大小设置为 30，那么对于后端的服务来讲，就等于有 300 个连接。

### 2.6 特权任务和定时任务
#### 2.6.1 OpenResty 中启动定时任务
OpenResty 的定时任务可以分为下面两种：
1. `ngx.timer.at` 用来执行一次性的定时任务
2. `ngx.time.every` 用来执行固定周期的定时任务

示例，启动了一个延时为 0 的定时任务。回调handler函数，并在函数中用cosocket访问一个网站：
```
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

```
content_by_lua_block {
            ngx.timer.at(3, function() end)
            ngx.say(ngx.timer.pending_count())
        }
```
代码会打印出 1，表示有 1 个计划任务正在等待被执行。

```
content_by_lua_block {
            ngx.timer.at(0.1, function() ngx.sleep(0.3) end)
            ngx.sleep(0.2)
            ngx.say(ngx.timer.running_count())
        }
```
代码会打印出 1，表示有 1 个计划任务正在运行中。

#### 2.6.2 特权进程
 OpenResty 在 Nginx 的基础上进行了扩展，增加了特权进程：privileged agent。特权进程很特别：
 - 它不监听任何端口，这就意味着不会对外提供任何服务；
- 它拥有和 master 进程一样的权限，一般来说是 root 用户的权限，这就让它可以做很多 worker 进程不可能完成的任务；
- 特权进程只能在 `init_by_lua` 上下文中开启；
- 另外，特权进程只有运行在 `init_worker_by_lua` 上下文中才有意义，因为没有请求触发，也就不会走到content、access 等上下文去。

下面，我们来看一个开启特权进程的示例：
```
init_by_lua_block {
    local process = require "ngx.process"
 
    local ok, err = process.enable_privileged_agent()
    if not ok then
        ngx.log(ngx.ERR, "enables privileged agent failed error:", err)
    end
}
```
配置以上之后，再启动OpenResty，可以看到Nginx进程中多了特权进行：
```
nginx: master process
nginx: worker process
nginx: privileged agent process
```
因为特权只在 `init_worker_by_lua` 阶段运行一次，所以如果需要定时执行，就需要使用上边的定时器功能，下边使用`ngx.timer`，周期性地触发：
```
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

> 特权进程的使用场景，我们一般用特权进程来处理的是清理日志、重启 OpenResty 自身等需要高权限的任务。你需要注意的是，不要把 worker 进程的任务交给特权进程来处理。这并非因为特权进程不能做到，而是其存在安全隐患。


#### 2.6.3 非阻塞的 ngx.pipe
在上边的代码中，给master发送信号的操作：
```
os.execute("kill -HUP " .. pid) 
```
这里使用了Lua的标准库，是阻塞操作，这里可以使用`lua-resty-shell` 来非阻塞的操作。

```
$ resty -e 'local shell = require "resty.shell"
local ok, stdout, stderr, reason, status =
    shell.run([[echo "hello, world"]])
    ngx.say(stdout)
```
这段代码可以算是 hello world 的另外一种写法了，它调用系统的 echo 命令来完成输出。类似的，你可以用 resty.shell ，来替代 Lua 中的 os.execute 调用。
lua-resty-shell 的底层实现，依赖了 lua-resty-core 中的 [ngx.pipe](https://github.com/openresty/lua-resty-core/blob/master/lib/ngx/pipe.md) API，所以，这个使用 lua-resty-shell 打印出 hello wrold 的示例，改用 ngx.pipe ，可以写成下面这样：

```
$ resty -e 'local ngx_pipe = require "ngx.pipe"
local proc = ngx_pipe.spawn({"echo", "hello world"})
local data, err = proc:stdout_read_line()
ngx.say(data)'
```

### 2.7 OpenResy中使用正则、时间等API
#### 2.7.1 正则
在OpenResty中的正则引擎是基于回溯的NFA来实现的，那么就有可能出现灾难性回溯，即正则在匹配的时候回溯过多，造成CPU 100%，正常服务被阻塞。
在OpenResty中可以使用如下配置进行规避：
```
lua_regex_match_limit 100000;
```
`lua_regex_match_limit` ，就是用来限制 PCRE 正则引擎的回溯次数的。这样，即使出现了灾难性回溯，后果也会被限制在一个范围内，不会导致你的 CPU 满载。

#### 2.7.2 时间API

`ngx.now` 返回当前的时间戳，包括小鼠部分：

```
resty -e 'ngx.say(ngx.now())'
```
其他的 `ngx.localtime`、`ngx.utctime`、`ngx.cookie_time` 和 `ngx.http_time` ，主要是返回和处理时间的不同格式。

需要注意的是，这些返回当前时间的 API，如果没有非阻塞网络 IO 操作来触发，便会一直返回缓存的值，而不是像我们想的那样，能够返回当前的实时时间，下边示例：
```
$ resty -e 'ngx.say(ngx.now())
os.execute("sleep 1")
ngx.say(ngx.now())'
```
在两次调用 `ngx.now` 之间，我们使用 Lua 的阻塞函数 sleep 了 1 秒钟，但从打印的结果来看，这两次返回的时间戳却是一模一样的。

那么，如果换成是非阻塞的 sleep 函数呢？比如下面这段新的代码：

```
$ resty -e 'ngx.say(ngx.now())
ngx.sleep(1)
ngx.say(ngx.now())'
```
这里出现这种情况的原因是，`ngx.now()`这个获取当前时间的函数对时间做了缓存。下边是`ngx.now()` 的源码：
```
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
```
#define ngx_timeofday()      (ngx_time_t *) ngx_cached_time
```
这里`ngx_cached_time` 的值，只在函数 `ngx_time_update` 中会更新。
ngx_time_update什么时候会被调用？如果你在 Nginx 的源码中去跟踪它的话，就会发现， ngx_time_update 的调用都出现在事件循环中。

#### 2.7.3 真值和空值
在OpenResty中真值和空值的判断，一直是个比较头痛的点。下边列出不同情况下的空值和真值。
首先在Lua中，除了 nil 和 false 之外，都是真值。

- ngx.null 的布尔值为真

```
$ resty -e 'if ngx.null then
ngx.say("true")    ---> true
end'
```
- cdata:NULL
当你通过 LuaJIT FFI 接口去调用 C 函数，而这个函数返回一个 NULL 指针，那么你就会遇到另外一种空值，即`cdata:NULL` 。
和ngx.null一样，cdata:NULL 也是真
```
$ resty -e 'local ffi = require "ffi"
local cdata_null = ffi.new("void*", nil)
if cdata_null then
    ngx.say("true")    ---> true
end'
```
但是令人想不到的是下边的代码会打印出true，也就是说cdata:NULL 是和 nil 相等的：

```
$ resty -e 'local ffi = require "ffi"
local cdata_null = ffi.new("void*", nil)
ngx.say(cdata_null == nil)'
```
- cjson.null 布尔值为真
```
$ resty -e 'local cjson = require "cjson"
local data = cjson.encode(nil)
local decode_null = cjson.decode(data)
ngx.say(decode_null == cjson.null)'
```

### 2.8 OpenResty第三方库的阅读技巧
在OpenResty中没有自带的HTTP client库，下边列出两个比较优秀的：
- [lua-resty-http](https://github.com/ledgetech/lua-resty-http)
- [lua-resty-requests](https://github.com/tokers/lua-resty-requests)

先来看下第三方库的代码结构：
![](https://i.loli.net/2020/09/21/wT81XAtDPKsRMiJ.png)

在首页也有对开源库的接口介绍，可以根据文档进行测试学习。

### 2.9 OpenResty 处理四层流量
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
#### 2.9.1 测试框架

```
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

#### 2.9.2 使用test::nginx测试框架
```
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


## 3 测试篇
### 3.1 test::nginx 简介
`test::nginx` 是 OpenResty 测试体系中的核心，OpenResty 本身和周边的 lua-rety 库，都是使用它来组织和编写测试集的。
`test::nginx` 糅合了 Perl、数据驱动以及 DSL（领域小语言）。对于同一份测试案例集，通过对参数和环境变量的控制，可以实现乱序执行、多次重复、内存泄漏检测、压力测试等不同的效果。

#### 3.1.1 安装
下边通过源码进行安装
1. 先安装perl的包管理器cpanminus
```
 yum install cpanminus
```
2. 下载最新的 test-nginx代码，并编译安装
```
git clone https://github.com/agentzh/test-nginx.git
cd test-nginx & perl Makefile.PL
sudo make install
```
3. 安装完之后可以运行test-nginx中带的测试用例，如下：
![](https://i.loli.net/2020/09/23/VAbIDRK4Y5q2hps.png)
以上会运行每个 t 目录下的测试用例，最后显示运行结果 PASS

### 3.2 测试用例介绍
test::nginx 中提供了很多 DSL 的原语，下边按照 Nginx 配置、发送请求、处理响应、检查日志这个流程，做了一个简单的分类。
#### 3.2.1 Nginx配置
`test::nginx` 的原语中带有 config 这个关键字的，就和 Nginx 配置相关，还有 `config`、`stream_config`、`http_config` 等。
他们的作用都一样，即在Nginx的不同上下文中，插入指定的Nginx配置。这些配置可以是Nginx指令，也可以是 `content_by_lua_block` 封装起来的Lua代码。
 config是最常用的原语，在其中可以加载Lua库，并调用函数来做白盒测试。下边是一段测试代码：
```
 === TEST 1: sanity
--- config
    location /t {
        content_by_lua_block {
            local plugin = require("apisix.plugins.key-auth")
            local ok, err = plugin.check_schema({key = 'test-key'})
            if not ok then
                ngx.say(err)
            end
            ngx.say("done")
        }
    }
```
这个测试案例的目的，是为了测试代码文件 `plugins.key-auth` 中， check_schema 这个函数能否正常工作。它在`location /t` 中使用 `content_by_lua_block` 这个 Nginx 指令，require 需要测试的模块，并直接调用需要检查的函数。

#### 3.2.2 发送请求
这个主要是模拟客户端发送请求。下边先从发送单个请求入手。
- **request**

想要单元测试的代码被运行，就要发送一个HTTP请求，访问的地址是config中注明的 /t ，如下：

```
--- request
GET /t
```
这段代码在 request 原语中，发起了一个 GET 请求，地址是 /t。这里，我们并没有注明访问的 ip 地址、域名和端口，也没有指定是 HTTP 1.0 还是 HTTP 1.1，这些细节都被 test::nginx 隐藏了，你不用去关心。这就是 DSL 的好处之一——你只需要关心业务逻辑，不用被各种细节所打扰。
如果想要测试HTTP 1.0 也可以显示指定：
```
--- request
GET /t  HTTP/1.0
```
除了 GET 方法之外，POST 方法也是需要支持的。下面这个示例，可以 POST hello world 这个字符串到指定的地址：
```
--- request
POST /t  
hello world
```
`test::nginx` 在这里为你自动计算了请求体长度，并自动增加了 host 和 `connection` 这两个请求头，以保证这是一个正常的请求。
为了可读性，以#开头的，会自动被识别为代码注释：
```
--- request
   # post request
POST /t  
hello world
```
> 有很多测试用例可以嵌入perl脚本，需要对perl有一定的了解

- **pipelined_requests**

下边来看下发送多个请求，
在 `test::nginx` 中，可以使用 `pipelined_requests` 这个原语，在同一个 keep-alive 的连接里面，依次发送多个请求：
```
--- pipelined_requests eval
["GET /hello", "GET /world", "GET /foo", "GET /bar"]
```
比如这个示例就会在同一个连接中，依次访问这 4 个接口。这样做会有两个好处：
- 第一是可以省去不少重复的测试代码，把 4 个测试案例压缩到一个测试案例中完成；
- 第二也是最重要的原因，你可以用流水线的请求，来检测代码逻辑在多次访问的情况下，是否会有异常。

基于它，你可以模拟出限流、限速、限并发等多种情况，用更真实和复杂的场景来检测你的系统是否正常。
> 在单个请求的用例里，每次执行用例时，`test::nginx` 都会单独启动Nginx进程，在执行完后，Nginx进程会退出，但是多个请求时会启动Nginx进行，然后各请求依次执行，适用于连续请求各请求之间有关联的业务。

- **repeat_each**
	

刚才我们提到了测试多个请求的情况，那么应该如何对同一个测试执行多次呢？

`test::nginx` 提供了一个全局的设置：`repeat_each`。它其实是一个 perl 函数，默认情况下是 `repeat_each(1)`，表示测试案例只运行一次。所以之前的测试案例中，我们都没有去单独设置它。

可以在 run_test() 函数之前来设置它，比如将参数改为 2：
```
repeat_each(2);
run_tests();
```
那么，每个测试案例就都会被运行两次，以此类推。

- **more_headers**

上边的`test::nginx` 在发送请求的时候，默认会带上 host 和 connection 这两个请求头。那么其他的请求头如何设置呢？
`more_headers` 就是专门做这件事儿的：
```
--- more_headers
X-Foo: blah
```
如果想设置多个头，那设置多行就可以了：
```
--- more_headers
X-Foo: 3
User-Agent: openresty
```

#### 3.2.3 处理响应
发送完请求后，`test::nginx` 中最重要的部分就来了，那就是处理响应，我们会在这里判断响应是否符合预期。这里我们分为 4 个部分依次介绍，分别是响应体、响应头、响应码和日志。
1. **response_body**

与 request 原语对应的就是 `response_body`，下面是它们两个配置使用的例子：
```
=== TEST 1: sanity
--- config
    location /t {
        content_by_lua_block {
            ngx.say("hello")
        }
    }
--- request
GET /t
--- response_body
hello
```
这个测试案例，在响应体是 hello 的情况下会通过，其他情况就会报错。
但是如果返回体很长，`test::nginx` 也支持正则表达式检测响应体，如下：
```
--- response_body_like
^he\w+$
```
这样你就可以对响应体进行非常灵活的检测了
`test::nginx` 还支持 unlike 的操作：
```
--- response_body_unlike
^he\w+$
```
这时候，如果响应体是hello，测试就不能通过了。
了解完单个请求的检测后，我们再来看下多个请求的检测。下面是配合 `pipelined_requests` 一起使用的示例：
```
--- pipelined_requests eval
["GET /hello", "GET /world", "GET /foo", "GET /bar"]
--- response_body eval
["hello", "world", "oo", "bar"]
```
这里需要注意的是，你发送了多少个请求，就需要有多少个响应来对应。

2. **response_headers**

响应头和请求头类似，每一行对应一个 header 的 key 和 value：
```
--- response_headers
X-RateLimit-Limit: 2
X-RateLimit-Remaining: 1
```
和响应体的检测一样，响应头也支持正则表达式和 unlike 操作，分别是 `response_headers_like` 、`raw_response_headers_like` 和 `raw_response_headers_unlike`。

3. **error_code**

响应码的检测支持直接的比较，同时也支持 like 操作：

```
--- error_code: 302
 
--- error_code_like: ^(?:500)?$
```

而对于多个请求的情况，error_code 自然也需要检测多次：

```
--- pipelined_requests eval
["GET /hello", "GET /hello", "GET /hello", "GET /hello"]
--- error_code eval
[200, 200, 503, 503]
```

4. **error_log**

用 `no_error_log` 来检测错误日志：

```
--- no_error_log
[error]
```
在上面的例子中，如果 Nginx 的错误日志 error.log 中，出现 [error] 这个字符串，测试就会失败。这是一个很常用的功能。
我们也可以指定错误日志中是否出现指定的字符串：

```
--- error_log
hello world
```
上面这段配置，其实就在检测 error.log 中是否出现了 hello world。

你可以在其中，用 eval 嵌入 perl 代码的方式，来实现正则表达式的检测，比如下面这样的写法：

```
--- error_log eval
qr/\[notice\] .*?  \d+ hello world/
```

#### 3.2.4 测试中的调试
下边是几个在调试阶段可能用到的原语，这些调试的原语也许不会提交到最终的代码中。
1. **ONLY**

如果在原有的测试案例集基础上，新增了一个测试案例。如果这个测试文件包含了很多的测试案例，那么从头到尾跑一遍显然是比较耗时的，这在你需要反复修改测试案例的时候尤为明显。

那么，有没有什么方法只运行你指定的某一个测试案例呢？ ONLY 这个标记可以轻松实现这一点：
```
=== TEST 1: sanity
=== TEST 2: get
--- ONLY
```
把 `--- ONLY` 放在需要单独运行的测试案例的最后一行，那么使用 prove 来运行这个测试案例文件的时候，就会忽略其他所有的测试案例，只运行这一个测试了。

2. **SKIP**

与只执行一个测试案例对应的需求，就是忽略掉某一个测试案例。SKIP 这个标记，一般用于测试尚未实现的功能：

```
=== TEST 1: sanity
=== TEST 2: get
--- SKIP
```

它的用法和 ONLY 类似。

3. **LAST**

在它之前的测试案例集都会被执行，后面的就会被忽略掉：

```
=== TEST 1: sanity
=== TEST 2: get
--- LAST
=== TEST 3: set
```
如果有时候你的测试案例是有依赖关系的，需要你执行完前面几个测试案例后，之后的测试才有意义。那么，在这种情况下去调试的话，LAST 就非常有用了。

![](https://raw.githubusercontent.com/BaihlUp/Figurebed/master/2023/202311171113762.png)

#### 3.2.5 测试计划 plan
它源自于 perl 的 `Test::Plan` 模块，所以文档并不在 `test::nginx`中
在 OpenResty 官方测试集的每一个文件的开始部分，你都能看到类似的配置：
```
plan tests => repeat_each() * (3 * blocks());
```
这里 plan 的含义是，在整个测试文件中，按照计划应该会做多少次检测项。如果最终运行的结果和计划不符，整个测试就会失败。
这个使用比较复杂，可以直接关掉
```
use Test::Nginx::Socket 'no_plan';
```

#### 3.2.6 预处理器
在同一个测试文件的不同测试案例之间，可能会有一些共同的设置。如果在每一个测试案例中都重复设置，就会让代码显得冗余，后面修改起来也比较麻烦。
你就可以使用 `add_block_preprocessor` 指令，来增加一段 perl 代码，比如下面这样来写：
```
add_block_preprocessor(sub {
    my $block = shift;
 
    if (!defined $block->config) {
        $block->set_value("config", <<'_END_');
    location = /t {
        echo $arg_a;
    }
    _END_
    }
});
```
这个预处理器，就会为所有的测试案例，都增加一段 config 的配置，而里面的内容就是 location /t。这样，在你后面的测试案例里，就都可以省略掉 config，直接访问即可：

```
=== TEST 1:
--- request
    GET /t?a=3
--- response_body
3
 
=== TEST 2:
--- request
    GET /t?a=blah
--- response_body
blah
```

#### 3.2.7 自定义函数
除了在预处理器中增加 perl 代码之外，你还可以在 run_tests 原语之前，随意地增加 perl 函数，也就是我们所说的自定义函数。
下面是一个示例，它增加了一个读取文件的函数，并结合 eval 指令，一起实现了 POST 文件的功能：
```
sub read_file {
    my $infile = shift;
    open my $in, $infile
        or die "cannot open $infile for reading: $!";
    my $content = do { local $/; <$in> };
    close $in;
    $content;
}
 
our $CONTENT = read_file("t/test.jpg");
 
run_tests;
 
__DATA__
 
=== TEST 1: sanity
--- request eval
"POST /\n$::CONTENT"
```

#### 3.2.8 乱序
`test::nginx` 可以实现 默认乱序、随机来执行测试案例，而非按照测试案例的前后顺序和编号来执行。
它的初衷是想测试出更多的问题。毕竟，每一个测试案例运行完后，都会关闭 Nginx 进程，并启动新的 Nginx 来执行，结果不应该和顺序相关才对。
这个功能谨慎使用，因为在实际业务中可能我们的测试案例就是顺序执行的，乱序可能导致各种不同的问题，可以使用下边两行代码关闭这个功能：
```
no_shuffle();
run_tests;
```
其中，`no_shuffle` 原语就是用来禁用随机，让测试严格按照测试案例的前后顺序来运行。

#### 3.2.9 reindex
OpenResty 的测试案例集，对格式有着严格的要求。每个测试案例之间都需要有 3 个换行来分割，测试案例的编号也要严格保持自增长。
幸好，我们有对应的自动化工具 reindex 来做这些繁琐的事情，它隐藏在 [ openresty-devel-utils](https://github.com/openresty/openresty-devel-utils) 项目中，因为没有文档来介绍，知道的人很少。
可以尝试着把测试案例的编号打乱，或者增删分割的换行个数，然后用这个工具来整理下，看看是否可以还原。

### 3.3 压测工具 wrk
参考资料：[https://www.cnblogs.com/xinzhao/p/6233009.html](https://www.cnblogs.com/xinzhao/p/6233009.html)
以上是wrk工具的安装和使用。
#### 3.3.1 测试环境
下边介绍下在进行压测前，需要对测试环境的配置。
1. 关闭SELinux

```
$ sestatus
SELinux status: disabled
```
可以执行`setenforce 0`来临时关闭；同时修改 /etc/selinux/config 文件来永久关闭，将 `SELINUX=enforcing` 改为 `SELINUX=disabled`。
2. 设置最大打开文件数

查看最大打开文件数：
```
$ cat /proc/sys/fs/file-nr
3984 0 3255296
```
这里的最后一个数字，就是最大打开文件数。如果你的机器中这个数字比较小，那就需要修改 /etc/sysctl.conf 文件来增大：

```
fs.file-max = 1020000
net.ipv4.ip_conntrack_max = 1020000
net.ipv4.netfilter.ip_conntrack_max = 1020000
```
修改完以后，还需要重启系统服务来生效：

```
sudo sysctl -p /etc/sysctl.conf
```

3. 进程限制

除了系统的全局最大打开文件数，一个进程可以打开的文件数也是有限制的，你可以通过命令 ulimit 来查看：
```
$ ulimit -n
1024
```
压力测试会产生大量的请求，所以我们需要增大这个数值，把它改为百万级别，你可以用下面的命令来临时修改：

```
$ ulimit -n 1024000
```
也可以修改配置文件 /etc/security/limits.conf 来永久生效：

```
* hard nofile 1024000
* soft nofile 1024000
```

## 4 性能优化篇
### 4.1 性能下降10倍的真凶：阻塞函数
OpenResty 之所以可以保持很高的性能，简单来说，是因为它借用了 Nginx 的事件处理和 Lua 的协程机制，所以：
* 在遇到网络 I/O 等需要等待返回才能继续的操作时，就会先调用 Lua 协程的 yield 把自己挂起，然后在 Nginx 中注册回调；
* 在 I/O 操作完成（也可能是超时或者出错）后，由 Nginx 回调 resume，来唤醒 Lua 协程。

这样的流程，保证了 OpenResty 可以一直高效地使用 CPU 资源，来处理所有的请求。
在这个处理流程中，如果没有使用 cosocket 这种非阻塞的方式，而是用阻塞的函数来处理 I/O，那么 LuaJIT 就不会把控制权交给 Nginx 的事件循环。这就会导致，其他的请求要一直排队等待阻塞的事件处理完，才会得到响应。

下面，我再来介绍几个常见的坑，也就是一些经常会被误用的阻塞函数；

#### 4.1.2 执行外部命令
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

#### 4.1.3 磁盘 I/O
再来看下处理磁盘I/O的场景。在服务端程序中，读取本地的配置文件是一个很常见的操作，如下下边的代码：
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

#### 4.1.4 luasocket
luasocket 也是容易被开发者用到的一个 Lua 内置库，经常有人分不清 luasocket 和 OpenResty 提供的 cosocket。luasocket 也可以完成网络通信的功能，但它并没有非阻塞的优势。如果你使用了 luasocket，那么性能也会急剧下降。
前面讲过的cosocket在不少阶段无法使用，这时可以考虑使用luasocket。

[lua-resty-socket](https://github.com/thibaultcha/lua-resty-socket/) 是一个二次封装的开源库，它做到了 luasocket 和 cosocket 的兼容。

### 4.2 让人又恨又爱的字符串操作
#### 4.2.1 性能优化背后
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

#### 4.2.2 字符串是不可变的
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
使用trble做一次封装，去掉所有临时的中间字符串，只保留原始数据和最终结果。

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
用 table 依次保存了每一个字符串，下标由 #t + 1 来决定，也就是用 table 的当前长度加 1；最后，使用 table.concat 函数，把数组的每一个元素进行拼接，直接得到最终结果。这样自然就跳过了所有的临时字符串，避免了 10 万次 lj_str_new 和 GC。
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

#### 4.2.3 减少其他临时字符串
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

#### 4.2.4 利用 SDK 对 table 类型的支持
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

### 4.3 性能提升10倍的秘诀：必须用好 table
table 相关的优化，有一个自己的简单原则：**尽量复用，避免不必要的 table 创建。**

#### 4.3.1 预先生成数组
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

#### 4.3.2 自己计算table下标
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
然后，它使用 nbits 这个变量，来自己维护 req 的下标，自然就抛弃了 Lua 内置的 table.insert 函数和获取长度的操作符 #。你可以看到，在 for 循环中，nbits + 1 等一些运算，就是直接用下标的方式插入元素；并在最后用 nbits = nbits + 5 ，让下标保持一个正确的值。
这种的好处很明显，它省略了获取数组大小这个 O(n) 的操作，而是直接用下标访问，时间复杂度也变成了 O(1) 。当然，缺点也一样明显，那就是降低了代码的可读性，并且出错概率大大提高，可以说，这是一把双刃剑。

#### 4.3.3 循环使用单个table
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

#### 4.3.4 tablepool池
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

### 4.4 OpenResty编码指南
1. 函数之间用两个空行分隔
2. 全部使用局部变量，变量命名snake_case 风格，常量使用全部大写形式
3. 数组使用table.new()提前分配
4. 函数名使用snake_case 命名风格，函数使用尽早返回
5. require 的库都要 local 化：

```lua
--Yes
local timer_at = ngx.timer.at
local function foo()
    local ok, err = timer_at(delay, handler)
end
```
6. 如果函数返回错误，对其处理

**参考资料：**
[代码规范](https://github.com/apache/apisix/blob/master/CODE_STYLE.md)


### 4.5 实际项目中的性能优化：ingress-nginx中的几个PR解读

给K8S中ingress-nginx提交的两个PR：
- [https://github.com/kubernetes/ingress-nginx/pull/3673](https://github.com/kubernetes/ingress-nginx/pull/3673)
- [https://github.com/kubernetes/ingress-nginx/pull/3674](https://github.com/kubernetes/ingress-nginx/pull/3674)

### 4.6 盘点OpenResty的各种调试手段
#### 4.6.1 断点和打印日志
一般通过打印日志和GDB设置断点可以定位的问题有一个前提，问题可以稳定复现，然后通过多次复现查看日志或者设置断点进行定位。
但是有些问题是不能稳定复现的，可以使用Mozilla RR，这个可以把程序执行的行为录制下来，然后反复重放，只要能把bug复现录制一次，就可以通过断点等手段进行定位。
资料：[Mozilla rr快速学习](https://my.oschina.net/u/4001231/blog/3017627)
项目地址：[https://github.com/rr-debugger/rr](https://github.com/rr-debugger/rr)

#### 4.6.2 分布式链路追踪
如果有多个服务，bug可能出现在其中一个服务中，定位起来相对困难，可以通过排除法，但相对效率低。
推荐使用 OpenTracing 这样的标准，来进行分布式追踪。OpenTracing 可以在系统的各处埋点，通过 Trace ID 把多个 Span 组成的调用链和埋点数据上报到服务端，进行分析和图形化的展现。这样就可以发现很多隐藏的问题，而且历史数据都会保存下来，方便我们随时对比和查看。
如果你的系统比较复杂，比如是在微服务的环境下，那么 Zipkin、Apache SkyWalking 都是不错的选择。

#### 4.6.3 动态调试
动态调试工具：Dtrace、Systemtap、eBPF、VTune
Systemtap 有自己的 DSL，也就是小语言，可以用来设置探测点。可以直接安装systemtap：
```bash
yum install systemtap
```

用 Systemtap 写的 hello world 程序是什么样子的：
```bash
# cat hello-world.stp
probe begin
{
  print("hello world!")
  exit()
}
```
执行：
```
stap helloworld.stp 
```
在大部分场景下，我们都不需要自己写 stap 脚本来进行分析，因为 OpenResty 已经有了很多现成的 stap 脚本来做常规的分析，下节课我就会为你介绍这些脚本。所以，今天我们只用对 stap 脚本有一个简单的认识就行了。

Systemtap 的工作原理，是将上述 stap 脚本转换为 C，运行系统 C 编译器来创建 kernel 模块。当模块被加载的时候，它会通过 hook 内核的方式，来激活所有的探测事件。
比如，刚刚这个示例代码中的 probe 就是一个探针。begin 会在探测的最开始运行，与之对应的是 end，所以上面的 hello world 程序也可以写成下面的这种方式：
```bash
probe begin
{
  print("hello ")
  exit()
}
 
probe end
{
  print("world!") 
}
```
以上程序同样输出“hello world!”
> Systemtap 的作者 Frank Ch. Eigler 写了一本电子书《Systemtap tutorial》，详细地介绍了 Systemtap。

eBPF（extended BPF）则是最近几年 Linux 内核中新增的特性。相比 Systemtap，eBPF 有内核直接支持、不会死机、启动速度快等优点；同时，它并没有使用 DSL，而是直接使用了 C 语言的语法，所以也大大降低了它的上手难度。
除了开源的解决方案外，Intel 出品的 VTune 也是神兵利器之一。它直观的界面操作和数据展示，可以让你不写代码也能分析出性能的瓶颈。

#### 4.6.4 火焰图
perf 和 Systemtap 等工具产生的数据，都可以通过火焰图的方式，来进行更加直观的展示。火焰图的示例：
![](https://raw.githubusercontent.com/BaihlUp/Figurebed/master/2023/202311171108702.png)

在火焰图中，色块的颜色和深浅都是没有意义的，只是为了对不同的色块儿做出简单的区分。火焰图其实是把每次采样的数据进行叠加，所以，真正有意义的是色块的宽度和长度。
对于 on CPU 火焰图来说，色块的宽度是函数占用的 CPU 时间百分比，色块越宽，则说明性能消耗越大。如果出现一个平顶的山峰，那它就是性能的瓶颈所在。而色块的长度，代表的是函数调用的深度，最顶端的框显示正在运行的函数，在它之下的都是这个函数的调用者。所以，在下面的函数是上面函数的父函数，山峰越高，则说明调用的函数层级越深。

- 火焰图的类型

常见的火焰图类型有 On-CPU，Off-CPU，还有 Memory，Hot/Cold，Differential 等等。他们分别适合处理什么样的问题呢？

![](https://raw.githubusercontent.com/BaihlUp/Figurebed/master/2023/202311171109198.png)

- 火焰图分析技巧

1. 纵轴代表调用栈的深度（栈桢数），用于表示函数间调用关系：下面的函数是上面函数的父函数。
2. 横轴代表调用频次，一个格子的宽度越大，越说明其可能是瓶颈原因。
3. 不同类型火焰图适合优化的场景不同，比如 on-cpu 火焰图适合分析 cpu 占用高的问题函数，off-cpu 火焰图适合解决阻塞和锁抢占问题。
4. 无意义的事情：横向先后顺序是为了聚合，跟函数间依赖或调用关系无关；火焰图各种颜色是为方便区分，本身不具有特殊含义
5. 多练习：进行性能优化有意识的使用火焰图的方式进行性能调优（如果时间充裕）

- **安装 SystemTap**

```bash
yum install systemtap systemtap-runtime
```

 - **SystemTap 脚本**
 
[https://github.com/openresty/openresty-systemtap-toolkit](https://github.com/openresty/openresty-systemtap-toolkit)
[https://github.com/openresty/stapxx](https://github.com/openresty/stapxx)

- **将统计数据转换成火焰图**

[https://github.com/brendangregg/FlameGraph](https://github.com/brendangregg/FlameGraph)

资料：[性能调优利器：火焰图](https://www.infoq.cn/article/a8kmnxdhbwmzxzsytlga)


### 4.7 systemtap-toolkit和stapxx：如何用数据搞定“疑难杂症”？

在 OpenResty 中有两个开源项目：openresty-systemtap-toolkit 和 stapxx 。它们是基于 Systemtap 封装好的工具集，用于 Nginx 和 OpenResty 的实时分析和诊断。它们可以覆盖 on CPU、off CPU、共享字典、垃圾回收、请求延迟、内存池、连接池、文件访问等常用的功能和调试场景。
需要特别注意的是，OpenResty 的最新版本 1.15.8 默认开启了 [LuaJIT GC64 模式](https://blog.openresty.com.cn/cn/luajit-gc64-mode/)，但是 openresty-systemtap-toolkit 和 stapxx 并没有跟着做对应的修改，这就会导致里面的工具都无法正常使用。所以，你最好在 OpenResty 旧的 1.13 版本中来使用这些工具。
> 现在[OpenResty XRay](https://openresty.com/en/xray/)同时支持LuaJIT GC64模式和非GC64模式[issue](https://github.com/openresty/stapxx/pull/48)

**注意：** 由于1.13版本以后的openresty默认开启了LuaJIT GC64模式，所以在编译安装openresty是可以配置`--without-luajit-gc64  --with-luajit`，关闭LuaJIT GC64模式


### 4.8 巧用wrk和火焰图，科学定位性能瓶颈
使用如下项目练习使用火焰图定位性能瓶颈
[https://github.com/iresty/lua-performance-demo](https://github.com/iresty/lua-performance-demo)

使用wrk进行压测：
 
```
wrk -c100 -t10 -d20s http://127.0.0.1:8080
```
压测20s，10个线程，100个连接

### 4.9 高性能的关键：shareddict缓存和lru缓存
#### 4.9.1 缓存
一般来说，缓存有两个原则。

- 一是越靠近用户的请求越好。比如，能用本地缓存的就不要发送 HTTP 请求，能用 CDN 缓存的就不要打到源站，能用 OpenResty 缓存的就不要打到数据库。
- 二是尽量使用本进程和本机的缓存解决。因为跨了进程和机器甚至机房，缓存的网络开销就会非常大，这一点在高并发的时候会非常明显。

**OpenResty 中有两个缓存的组件：shared dict 缓存和 lru 缓存。**
- shared dict缓存：只能缓存字符串对象，缓存的数据有且只有一份，每一个 worker 都可以进行访问，所以常用于 worker 之间的数据通信。
- lru缓存：可以缓存所有的 Lua 对象，但只能在单个 worker 进程内访问，有多少个 worker，就会有多少份缓存数据。

**下边的两个表格，可以说明shared dict和lru缓存的区别：**
![](https://raw.githubusercontent.com/BaihlUp/Figurebed/master/2023/202311171113687.png)
![](https://raw.githubusercontent.com/BaihlUp/Figurebed/master/2023/202311171112644.png)

shared dict 和 lru 缓存，并没有哪一个更好的说法，而是应该根据你的场景来配合使用。

- 如果你没有 worker 之间共享数据的需求，那么 lru 可以缓存数组、函数等复杂的数据类型，并且性能最高，自然是首选。
- 但如果你需要在 worker 之间共享数据，那就可以在 lru 缓存的基础上，加上 shared dict 的缓存，构成两级缓存的架构。

#### 4.9.2 共享字典缓存
使用方法

```
$ resty --shdict='dogs 1m' -e 'local dict = ngx.shared.dogs
        dict:set("Tom", 56)
        print(dict:get("Tom"))'
```
你需要事先在 Nginx 的配置文件中，声明一个内存区 dogs，然后在 Lua 代码中才可以使用。如果你在使用的过程中，发现给 dogs 分配的空间不够用，那么是需要先修改 Nginx 配置文件，然后重新加载 Nginx 才能生效的。因为我们并不能在运行时进行扩容和缩容。

- **缓存数据的序列化**

第一个问题，缓存数据的序列化。由于共享字典中只能缓存字符串对象，所以，如果你想要缓存数组，就少不了要在 set 的时候要做一次序列化，在 get 的时候做一次反序列化：

```
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

#### 4.9.3 lru缓存
lru 缓存的接口只有 5 个：new、set、get、delete 和 flush_all。和上面问题相关的就只有 get 接口，让我们先来了解下这个接口是如何使用的：

```
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

### 4.10 缓存风暴
#### 4.10.1 什么是缓存风暴？
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

#### 4.10.2 如何避免缓存风暴？
##### 1. 主动更新缓存

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

##### 2. lua-resty-lock

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

##### 3. lua-resty-shcache
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

##### 4. Nginx配置指令
即使你没有使用 OpenResty 的 lua-resty 库，你也可以用 Nginx 的配置指令，来实现加锁和获取过期数据——即proxy_cache_lock 和 proxy_cache_use_stale。不过，这里我并不推荐使用 Nginx 指令这种方式，它显然不够灵活，性能也比不上 Lua 代码。


### 4.11 lua-resty-* 封装，让你远离多级缓存之痛
#### 4.11.1 lua-resty-memcached-shdict
让我们回到缓存的封装上来。lua-resty-memcached-shdict 是 OpenResty 官方的一个项目，它使用 shared dict 为 memcached 做了一层封装，处理了缓存风暴和过期数据等细节。如果你的缓存数据正好存储在后端的 memcached 中，那么你可以尝试使用这个库。
如果你想在本地测试，需要先把它的[源码](https://github.com/openresty/lua-resty-memcached-shdict)下载到本地 OpenResty 的查找路径下。

这个封装库，其实和我们上节课中提到的解决方案是一样的。它使用 lua-resty-lock 来做到互斥，在缓存失效的情况下，只有一个请求去 memcached 中获取数据，避免缓存风暴。如果没有获取到最新数据，则使用 stale 数据返回给终端。

在这个封装库的文档中，其实也提到了进一步的优化方向：
- 一是使用 lua-resty-lrucache ，来增加 worker 层的缓存，而不仅仅是 server 级别的 shared dict 缓存；
- 二是使用 ngx.timer ，来做异步的缓存更新操作。

#### 4.11.2 lua-resty-mlcache
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

### 4.12 应对突发流量：漏桶和令牌桶的概念
#### 4.12.1 流量控制
我们需要从成本、用户体验、系统稳定性等多个方面来综合考虑。不管使用哪一种算法，都不可避免地会造成正常用户请求变慢甚至被拒绝，牺牲部分的用户体验。所以，流量控制是需要在业务稳定和用户体验之间做平衡的。

举个例子，比如我们假定一个上游服务的设计上限是每分钟处理 1 万条请求。在高峰期的时候，如果入口处没有限流的控制，每分钟堆积的任务达到了 2 万条，那么这个上游服务的处理性能就会下降，可能只有每分钟 5000 条的处理速度，并且持续恶化，最终或许会导致服务不可用。这显然不是我们希望看到的结果。
应对这种突发的流量，我们常用的流量控制算法，便是**漏桶**和**令牌桶**。

#### 4.12.2 漏桶算法
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

#### 4.12.3 令牌桶算法
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

#### 4.12.4 Nginx的限速模块
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

### 4.13 动态限流限速
在 OpenResty 中，我们推荐使用 [lua-resty-limit-traffic](https://github.com/openresty/lua-resty-limit-traffic) 来做流量的限制。它里面包含了 limit-req（限制请求速率）、 limit-count（限制请求数） 和 limit-conn （限制并发连接数）这三种不同的限制方式；并且提供了limit.traffic ，可以把这三种方式进行聚合使用。

#### 4.13.1 限制请求速率
让我们先来看下 limit-req，它使用的是漏桶算法来限制请求的速率。
示例代码：
```
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

#### 4.13.2 限制请求数
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
#### 4.13.3 限制并发连接数
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
#### 4.13.4 限速器组合
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

### 4.14 OpenResty的杀手锏：动态
#### 4.14.1 动态加载代码
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

#### 4.14.2 FaaS（函数即服务）
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

#### 4.14.3 边缘计算
OpenResty 的动态不仅可以用于 FaaS，让脚本语言的动态细化到函数级别，还可以在边缘计算上发挥动态的优势。
得益于 Nginx 和 LuaJIT 良好的多平台支持特性，OpenResty 不仅能运行在 X86 架构下，对于 ARM 的支持也很完善。同时， OpenResty 支持七层和四层的代理，这样一来，常见的各种协议都可以被 OpenResty 解析和代理，这其中也包括了 IoT 中的几种协议。
因为这些优势，我们便可以把 OpenResty 的触角，从 API 网关、WAF、web 服务器等服务端的领域，伸展到物联网设备、CDN 边缘节点、路由器等最靠近用户的边缘节点上去。
以 CDN 的边缘节点为例，OpenResty 的最大使用者 CloudFlare 很早就借助 OpenResty 的动态特性，实现了对于 CDN 边缘节点的动态控制。
CloudFlare 的做法和上面动态加载代码的原理是类似的，大概可以分为下面几个步骤：

* 首先，从键值数据库集群中获取到有变化的代码文件，获取的方式可以是后台 timer 轮询，也可以是用“发布 - 订阅”的模式来监听；
* 然后，用更新的代码文件替换本地磁盘的旧文件，然后使用 loadstring 和 pcall的方式，来更新内存中加载的缓存；

这样，下一个被处理的终端请求，就会走更新后的代码逻辑。
当然，实际的应用要比上面的步骤考虑更多的细节，比如版本的控制和回退、异常的处理、网络的中断、边缘节点的重启等，但整体的流程是不变的。

#### 4.14.4 动态上游
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

### 4.15 OpenResty常用第三方库
#### 4.15.1 lua-resty库推荐
首先推荐的是由 Aapo 维护的 awesome-resty [仓库](https://github.com/bungle/awesome-resty)，这个仓库分门别类地整理了和 OpenResty 相关的库，可以说是包罗万象，包括了 Nginx 的 C 模块、lua-resty 库、web 框架、路由库、模板、测试框架等，是你寻找 OpenResty 资源的首选。

还可以去 luarocks、opm 和 GitHub 碰碰运气。有一些开源时间不长的、或者关注不多的库，可能就藏在其中。
还可以去 luarocks、opm 和 GitHub 碰碰运气。有一些开源时间不长的、或者关注不多的库，可能就藏在其中。

#### 4.15.2 lua-var-nginx-module
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

#### 4.15.3 JSON Schema
[lua-rapidjson](https://github.com/xpol/lua-rapidjson) 。它是对 rapidjson 这个腾讯开源的 JSON 库的封装，以性能见长。这里，我们着重介绍下它和 cjson 的不同之处，也就是支持 JSON Schema。
JSON Schema 是一个通用的标准，借助这个标准，我们就可以精确地描述接口中参数的格式，以及如何校验的问题。下面是一个简单的示例：

```
"stringArray": {
    "type": "array",
    "items": { "type": "string" },
    "minItems": 1,
    "uniqueItems": true
}
```
这段 JSON 准确地描述了 stringArray 这个参数的类型是字符串数组，并且数组不能为空，数组元素也不能重复。
而lua-rapidjson，则是可以让我们在 OpenResty 中来使用 JSON Schema，这能给接口的校验带来极大的便利。举个例子，比如对于前面介绍过的 limit count 限流接口，我们就可以用下面的 schema 来描述：
```
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

#### 4.15.4 worker 间通信
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

### 4.16 答疑
#### 4.16.1 问题一：如何完成Lua模块的动态加载？
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

#### 4.16.2 问题二：LuaJIT 的 NYI 的操作，是否会对性能有很大影响？
Q：loadstring 在 LuaJIT 的 NYI 列表是 never，会不会对性能有很大影响？

A：关于 LuaJIT 的 NYI，我们不用矫枉过正。对于可以 JIT 的操作，自然是 JIT 的方式最好；但对于还不能 JIT 的操作，我们也不是不能使用。

对于性能优化，我们需要用基于统计的科学方法来看待，这也就是火焰图采样的意义。过早优化是万恶之源。对于那些调用次数频繁、消耗 CPU 很高的热代码，我们才有优化的必要。
回到 loadstring 的问题，我们只会在代码发生变化的时候，才会调用它重新加载，和请求多少无关，所以它并不是一个频繁的操作。这个时候，我们就不用担心它对系统整体性能的影响。
结合第二个阻塞的问题，在 OpenResty 中，我们有些时候也会在 init 和 init worker 阶段，去调用阻塞的文件 I/O 操作。这种操作比 NYI 更加影响性能，但因为它只在服务启动的时候执行一次，所以也是可以被我们接受的。
还是那句话，性能优化要从宏观的视角来看待，这是你特别需要注意的一个点。否则，纠结于某一细节，就很有可能优化了半天，却并没有起到很好的效果。

#### 4.16.3 问题三：共享字典的缓存是必须的吗？
Q：在实际的生产应用中，我认为 shared dict 这一层缓存是必须的。貌似大家都只记得 lruca che 的好，数据格式没限制、不需要反序列化、不需要根据 k/v 体积算内存空间、worker 间独立不相互争抢、没有读写锁、性能高云云。
但是，却忘记了它最致命的一个弱点，就是 lru cache 的生命周期是跟着 worker 走的。每当 Nginx reload 时，这部分缓存会全部丢失，这时候，如果没有 shared dict，那 L3 的数据源分分钟被打挂
。当然，这是并发比较高的情况下，但是既然用到了缓存，就说明业务体量肯定不会小，也就是刚刚的分析仍然适用。不知道我的这个观点对吗？

A：大部分情况下，确实如你所说，共享字典在 reload 的时候不会丢失，所以它有存在的必要性。但也有一种特例，那就是，如果在 init 阶段或者 init_worker 阶段，就能从 L3 也就是数据源主动获取到所有数据，那么只有 lru cache 也是可以接受的。
举例来说，比如开源 API 网关 [APISIX](https://github.com/apache/apisix) 的数据源在 etcd 中，它只在 init_worker 阶段，从 etcd 中获取数据并缓存在 lru cache 中，后面的缓存更新，都是通过 etcd 的 watch 机制来主动获取的。这样一来，即使 Nginx reload ，也不会有缓存风暴产生。
所以，对待技术的选择，我们可以有倾向，但还是不要一概而论绝对化，因为并没有一个可以适合所有缓存场景的银弹。根据实际场景的需要，构建一个最小化可用的方案，然后逐步地增加，是一个不错的法子。


## 5 API网关篇
基于[APISIX](https://github.com/apache/apisix) 讲解
### 5.1 微服务API网关搭建
#### 5.1.1 微服务API网关有什么用？
有以下传统功能：

* 反向代理和负载均衡，这和 Nginx 的定位和功能是一致的；
* 动态上游、动态 SSL 证书和动态限流限速等运行时的动态功能，这是开源版本 Nginx 并不具备的功能；
* 上游的主动和被动健康检查，以及服务熔断功能；
* 在 API 网关的基础上进行扩展，成为全生命周期的 API 管理平台。

在最近几年，业务相关的流量，不再仅仅由 PC 客户端和浏览器发起，更多的来自手机、IoT 设备等，未来随着 5G 的普及，这些流量会越来越多。同时，随着微服务架构的结构变迁，服务之间的流量也开始爆发性地增长。在这种新的业务场景下，自然也催生了 API 网关更多、更高级的功能：
在最近几年，业务相关的流量，不再仅仅由 PC 客户端和浏览器发起，更多的来自手机、IoT 设备等，未来随着 5G 的普及，这些流量会越来越多。同时，随着微服务架构的结构变迁，服务之间的流量也开始爆发性地增长。在这种新的业务场景下，自然也催生了 API 网关更多、更高级的功能：

1. 云原生友好，架构要变得轻巧，便于容器化；
2. 对接 Prometheus、Zipkin、Skywalking 等统计、监控组件；
3. 支持 gRPC 代理，以及 HTTP 到 gRPC 之间的协议转换，把用户的 HTTP 请求转为内部服务的 gPRC 请求；
4. 承担 OpenID Relying Party 的角色，对接 Auth0、Okta 等身份认证提供商的服务，把流量安全作为头等大事来对待；
5. 通过运行时动态执行用户函数的方式来实现 Serverless，让网关的边缘节点更加灵活；
6. 不锁定用户，支持混合云的部署架构；
7. 最后，网关节点要状态无关，可以随意地扩容和缩容。

当一个微服务 API 网关具备了上述十几项功能时，就可以让用户的服务只关心业务本身；而和业务实现无关的功能，比如服务发现、服务熔断、身份认证、限流限速、统计、性能分析等，就可以在独立的网关层面来解决。当一个微服务 API 网关具备了上述十几项功能时，就可以让用户的服务只关心业务本身；而和业务实现无关的功能，比如服务发现、服务熔断、身份认证、限流限速、统计、性能分析等，就可以在独立的网关层面来解决。

### 5.2 答疑
#### 5.2.1 OpenResty的数据库封装
Q：根据你的指点，要尽量少用 ..字符串拼接，特别是在代码热区。但是我在处理数据库访问时，需要动态构建 SQL 语句（在语句中插入变量），这应该是非常常见的使用场景。可是对于这个需求，我目前感觉，只有字符串拼接是最简单的办法，其他真的想不到既简单又高性能的办法。

A：你可以先用我们前面课程介绍过的 SystemTap 或者其他工具分析下，看 SQL 语句的拼接是否是系统的瓶颈。如果不是，自然就没有优化的必要性，毕竟，过早的优化是万恶之源。

如果瓶颈确实是 SQL 语句的拼接，那么我们可以利用数据库的 prepare 语句来做优化，也可以用数组的方式来做拼接。但 lua-resrty-mysql 对 prepare 的支持一直处于 TODO 状态，所以只剩下数组拼接的方式了。这也是一些 lua-resty 库的通病，实现了大部分的功能，处于能用的状态，但更新得并不够及时。除了数据库的 prepare 语句外，lua-resty-redis 对 cluster 也一直没有支持。
字符串拼接，包括 lua-resty 库的这类问题，OpenResty 是希望用 DSL 来彻底解决的——使用编译器的技术自动生成数组来拼接字符串，把这些细节隐藏起来，上层的用户不用感知；使用小语言 wirelang 来自动生成各种 lua-resty 网络通信库，不再需要手写。

#### 5.2.2 修改了响应体，怎么修改响应头中的 content-length？
Q：如果需要修改 respones body 的内容，就只能在 body filter 里做修改，但这样会引起 body 长度与 content-length 长度不一致，应该如何处理呢？

A：在这种情况下，我们需要在 body filter 之前的 header filter 阶段中，把 content length 这个响应头置为 nil，不再返回，改为流式输出。

下面是一段示例代码：
```
server {
    listen 8080;
 
    location /test {
            proxy_pass http://www.baidu.com;
            header_filter_by_lua_block {
                     ngx.header.content_length = nil
            }
            body_filter_by_lua_block {
                    ngx.arg[1] = ngx.arg[1] .. "abc"
            }
     }
}
```
通过这段代码你可以看到，在 body filter 阶段中，ngx.arg[1] 代表的就是响应体。如果我们在它后面增加了字符串 abc，响应头 content length 就不准确了，所以，我们在 header filter 阶段直接把它禁用掉就可以了。




