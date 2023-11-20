
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
```bash
mkdir geektime
cd geektime
mkdir logs/ conf/
```
3. 编写nginx.conf

下边编写一个最简单的nginx.conf配置
```shell
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

```bash
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
```bash
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
```shell
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
```shell
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

# 4 盘点OpenResty的各种调试手段

## 4.1 OpenResty编码指南
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
```bash
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




