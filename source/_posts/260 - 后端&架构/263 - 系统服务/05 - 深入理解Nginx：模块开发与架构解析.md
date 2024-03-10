---
title: 深入理解Nginx：模块开发与架构解析
date: 2024-02-04
categories:
  - 后端&架构
tags:
  - Nginx
published: true
---
# 0 参考资料
[书中示例代码](https://github.com/russelltao/diveintonginx)

Nginx 源码注释：[https://github.com/chronolaw/annotated_nginx](https://github.com/chronolaw/annotated_nginx)

# 2 如何编写HTTP模块
## 2.7 Nginx提供的高级数据结构

```c
- ngx_queue_t 双向链表
- ngx_array_t 动态数组
- ngx_list_t 单向链表
- ngx_rbtree_t 红黑树
- ngx_radix_tree_t 基数树
```

# 第3部分 深入Nginx

## 8 Nginx 基础架构
### 8.2.1 Nginx的模块化设计
ngx_module_t接口及其对核心、事件、HTTP、mail等4类模块ctx上下文成员的具体化：

![](https://raw.githubusercontent.com/BaihlUp/Figurebed/master/2023/FDBDFFA5-CDE7-4CAA-9A0B-7E35BD5DA170.png)

官方Nginx共有五大类模块：核心模块、配置模块、事件模块、HTTP模块、mail模块。
这五类模块中，配置模块与核心模块是与Nginx框架密切相关的，是其他模块的基础。而事件模块是HTTP模块和mial模块的基础。HTTP模块和mail模块的“地位”相似，都更关注于应用层面。在事件模块中，ngx_event_core_module事件模块是其他有事件模块的基础；在HTTP模块中，ngx_module_core_module模块是其他所有HTTP模块的基础；在mail模块中，ngx_mail_core_module模块是其他所有mail模块的基础。
Nginx常用模块及其之间的关系：

![](https://raw.githubusercontent.com/BaihlUp/Figurebed/master/2023/7CE15B43-34D8-4D18-90FA-FD56ED2F69B3.png)

### 8.2.2 事件驱动框架
传统的Web服务器是进程或线程做为事件的消费者，一个请求产生的事件被进程处理，直到这个请求处理结束，才会释放进程资源。
传统Web服务器事件处理模型：
![](https://raw.githubusercontent.com/BaihlUp/Figurebed/master/2023/5A035685-7855-4CB5-B518-FC13467723EB.png)

nginx通过事件驱动的方式处理，事件的消费者是某个模块，没有事件只会交给对应的模块处理：
![](https://raw.githubusercontent.com/BaihlUp/Figurebed/master/2023/4C8C9827-FA91-4552-8D1B-622724B6A8F0.png)
以上的模型要求每个模块不能有导致进程阻塞的行为。否则会使进程休眠，大大影响并发性能。

### 8.2.3 请求的多阶段异步处理
多阶段异步处理和事件驱动架构是密切相关的。以下示例，获取一个静态文件的HTTP请求可以分为如下阶段：
![](https://raw.githubusercontent.com/BaihlUp/Figurebed/master/2023/79064433-500E-4391-BED2-C19D54DCF686.png)
这7个阶段可以重复发生。每个阶段都由事件分发器触发，然后继续调用事件消费者处理请求。通过划分阶段可以避免某个处理事件过长导致进程的延迟或者阻塞。

- 划分阶段的原则：一般是找到请求处理流程中的阻塞方法，在阻塞代码段上按照下边的4种方式划分阶段：

1. 讲阻塞进程的方法按照相关的触发事件分解为两个阶段

把阻塞方法改为调用非阻塞方法，调用非阻塞方法后将进程归还给事件分发器这就是第一个阶段，然后增加第二阶段处理非阻塞方法返回的结果。
示例：
例如，在使用send调用发送数据给用户时，如果使用阻塞socket句柄，那么send调用在向操作系统内核发出数据包后就必须把当前进程休眠，直到成功发出数据才能“醒来”。这时的send调用发送数据并等待结果。我们需要把send调用分解为两个阶段:发送且不等待结果阶段send结果返回阶段。因此，可以使用非阻塞socket句柄，这样调用send发送数据后，进程是不会进入休眠的，这就是发送且不等待结果阶段;再把socket句柄加入到事件收集器中就可以等待相应的事件触发下一个阶段，send发送的数据被对方收到后这个事件就会触发send结果返回阶段。这个send调用就是请求的划分阶段点。


2. 将阻塞方法调用按照时间分解为多个阶段的方法调用

例如读取文件，因为nginx使用的事件模块没有打开对异步I/O的支持，所以还是需要用调用阻塞的方式读取（可以使用内核的异步I/O，但不是所有平台都提供）。可以把读取文件的操作划分成多次，比如每次读取10KB，这样减少了每次占用进程的时间，每次读取操作完之后，如果要进入下个阶段，可以考虑使用定时器出发，或者把读取到的内容发出去，然后由网络时间触发进入下一个阶段。

3. 使用定时器时间触发划分阶段
4. 如果无法避免要调用阻塞方法，可以考虑使用独立的进程执行，然后进程执行完后触发事件。
> 比如有些模块实现的不合理，使用了阻塞方法，如果要使用这些模块，在执行阻塞方法时，可以使用独立的进程调用。


## 11 HTTP框架的执行流程
### 11.1 HTTP处理流程
HTTP框架在初始化时就会将每个监听ngx_listening_t结构体的handler方法设为ngx_http_init_connection方法。
ngx_http_init_connection方法的执行流程：

![](https://raw.githubusercontent.com/BaihlUp/Figurebed/master/2023/20230412112221.png)

第一次可读事件到来会执行ngx_http_init_request，流程如下：

![](https://raw.githubusercontent.com/BaihlUp/Figurebed/master/2023/20230412112351.png)
为了提升性能，Nginx并不会在创建连接时就分配内存，只有在第一个可读事件到来后，开始创建ngx_http_request_t，并进行初始化。
ngx_http_request_t结构体保存了很多整个请求的信息和处理过程中的数据。
ngx_http_init_request执行到最后会调用ngx_http_process_request_line方法开始接收、解析HTTP请求行，请求行的处理至少会调用一次，根据网络分包情况，会出现多次调用，流程如下：

![](https://raw.githubusercontent.com/BaihlUp/Figurebed/master/2023/20230412112841.png)
解析请求行后，就会把解析的一些内容保存到ngx_http_request_t结构中特定变量下。
下边开始在ngx_http_process_request_headers中接收HTTP头部：

![](https://raw.githubusercontent.com/BaihlUp/Figurebed/master/2023/20230412113214.png)

下边调用ngx_http_process_request开始处理请求：

![](https://raw.githubusercontent.com/BaihlUp/Figurebed/master/2023/20230412113314.png)

处理请求的过程中，会执行HTTP各阶段，并且调用阶段里要执行的各模块。
执行各阶段的模块，由ngx_http_core_run_phases调用各阶段的checker方法执行。
```c
// 启动引擎数组处理请求
// 从phase_handler的位置开始调用模块处理
void
ngx_http_core_run_phases(ngx_http_request_t *r)
{
    ngx_int_t                   rc;
    ngx_http_phase_handler_t   *ph;
    ngx_http_core_main_conf_t  *cmcf;

    // 得到core main配置
    cmcf = ngx_http_get_module_main_conf(r, ngx_http_core_module);

    // 获取引擎里的handler数组
    ph = cmcf->phase_engine.handlers;

    // 从phase_handler的位置开始调用模块处理
    // 外部请求的引擎数组起始序号是0，从头执行引擎数组,即先从Post read开始
    // 内部请求，即子请求.跳过post read，直接从server rewrite开始执行，即查找server
    while (ph[r->phase_handler].checker) {

        // 调用引擎数组里的checker
        rc = ph[r->phase_handler].checker(r, &ph[r->phase_handler]);

        // checker会检查handler的返回值
        // 如果handler返回again/done那么就返回ok
        // 退出引擎数组的处理
        // 由于r->write_event_handler = ngx_http_core_run_phases
        // 当再有写事件时会继续从之前的模块执行
        //
        // 如果checker返回again，那么继续在引擎数组里执行
        // 模块由r->phase_handler指定，可能会有阶段的跳跃
        if (rc == NGX_OK) {
            return;
        }
    }
}
```
模块的处理函数都是通过框架给的checker函数调用，不同阶段的checker函数存在差异，所以不同阶段也可以在调用模块处理时做一些操作。

![](https://raw.githubusercontent.com/BaihlUp/Figurebed/master/2023/20230412141549.png)
一个请求多半需要Nginx事件模块多次地调度HTTP模块处理， 这时就要看在ngx_http_process_request处理请求的第2步设置的读写事件的回调方法ngx_http_request_handler的功能了。
请求在处理的时候，第一次调用的是ngx_http_process_request，后边再次处理则使用ngx_http_request_handler，处理流程如下：

![](https://raw.githubusercontent.com/BaihlUp/Figurebed/master/2023/20230412141951.png)
通常来说， 在接收完HTTP头部后， 是无法在一次Nginx框架的调度中处理完一个请求的。 在第一次接收完HTTP头部后， HTTP框架将调度ngx_http_process_request方法开始处理请求， 这时 如果某个checker方法返回了NGX_OK， 则将会把控制权交还给Nginx框架。 当这个请求上对应的事件再次触发时， HTTP框架将不会再调度ngx_http_process_request方法处理请求， 而是由ngx_http_request_handler方法开始处理请求。 


### 11.2 读HTTP请求状态机流程

![](https://raw.githubusercontent.com/BaihlUp/Figurebed/master/2023/20230414102217.png)

参考文章：[https://www.codedump.info/post/20190131-nginx-read-http-request/](https://www.codedump.info/post/20190131-nginx-read-http-request/)

### 11.3 checker方法
ngx_http_core_run_phases函数中会遍历handlers数组，handlers数组是包含所有模块的处理函数，在ngx_http_init_phase_handlers函数中初始化所有HTTP阶段的模块时填充。不同的阶段会指定checker函数，每个阶段里又包括多个模块，每个模块都有自己的handler处理方法，如果当前阶段顺序调用模块处理时，如果不想处理剩下模块，可以直接通过next，跳到下一个阶段。
![](https://raw.githubusercontent.com/BaihlUp/Figurebed/master/2023/20230412181514.png)

下边是ngx_http_core_generic_phase的checker方法：
```c
// NGX_HTTP_POST_READ_PHASE/NGX_HTTP_PREACCESS_PHASE
// post read/pre-access只有一个模块会执行，之后的就跳过
//
// ok:模块已经处理成功，直接跳过本阶段
// decline:表示不处理,继续在本阶段（rewrite）里查找下一个模块
// again/done:暂时中断ngx_http_core_run_phases
//
// 由于r->write_event_handler = ngx_http_core_run_phases
// 当再有写事件时会继续从之前的模块执行
// 其他的错误，结束请求
// 但如果count>1，则不会真正结束
ngx_int_t
ngx_http_core_generic_phase(ngx_http_request_t *r, ngx_http_phase_handler_t *ph)
{
    ngx_int_t  rc;

    /*
     * generic phase checker,
     * used by the post read and pre-access phases
     */

    ngx_log_debug1(NGX_LOG_DEBUG_HTTP, r->connection->log, 0,
                   "generic phase: %ui", r->phase_handler);

    // 调用每个模块自己的处理函数
    rc = ph->handler(r);

    // 模块已经处理成功，直接跳过本阶段
    // 注意不是++，而是next
    // 意味着post read/pre-access只有一个模块会执行，之后的就跳过
    if (rc == NGX_OK) {
        r->phase_handler = ph->next;

        // again继续引擎数组的循环
        return NGX_AGAIN;
    }

    // 模块handler返回decline，表示不处理
    if (rc == NGX_DECLINED) {
        // 继续在本阶段（rewrite）里查找下一个模块
        // 索引加1
        r->phase_handler++;

        // again继续引擎数组的循环
        return NGX_AGAIN;
    }

    // again/done，暂时中断ngx_http_core_run_phases
    // 由于r->write_event_handler = ngx_http_core_run_phases
    // 当再有写事件时会继续从之前的模块执行
    if (rc == NGX_AGAIN || rc == NGX_DONE) {
        return NGX_OK;
    }

    /* rc == NGX_ERROR || rc == NGX_HTTP_...  */

    // 其他的错误，见上nginx注释

    // 结束请求
    // 但如果count>1，则不会真正结束
    ngx_http_finalize_request(r, rc);

    return NGX_OK;
}
```
以上方法中，如果模块处理方法返回NGX_OK，则跳过本阶段剩下的模块，直接跳到下一个阶段。如果返回NGX_DECLINED，则继续在本阶段查找下一个模块，NGX_AGAIN/NGX_DONE则暂时中断ngx_http_core_run_phases方法，等待下次写事件到来后被epoll再次调用。

### 11.4 subrequest与post请求
Nginx使用的完全无阻塞的事件驱动框架是难以编写功能复杂的模块的， 可以想见， 一个请求在处理一个TCP连接时， 将需要处理这个连接上的可读、 可写以及定时器事件， 而可读事件中又包含连接建立成功、 连接关闭事件， 正常的可读事件在接收到HTTP的不同部分时又要做不同的处理， 这就比较复杂了。 如果一个请求同时需要与多个上游服务器打交道， 同时处理多个TCP连接， 那么它需要处理的事件就太多了， 这种复杂度会使得模块难以维护。 Nginx解决这个问题的手段就是第5章中介绍过的subrequest机制。

### 11.5 处理HTTP包体
在ngx_http_request_t结构体中的count引用计数标识，因为HTTP模块在处理请求时，接受包体的同时可能还需要处理其他业务，如使用upstream机制与另一台服务器通信。所以在销毁请求时需要通过这个计数判断，否则可能引发严重错误，在为一个请求添加新的事件，或者把一些已经由定时器、epoll中移除的事件重新加入其中，都需要把这个请求的引用计数加1。通过这个标识可以让HTTP框架知道，HTTP模块对于该请求有独立的异步处理机制。
调用ngx_http_read_client_request_body方法相当于启动了接收包体这个动作。
读取请求包体的重要结构为ngx_http_request_body_t
```c
// 请求体的数据结构，用于读取或丢弃请求体数据
typedef struct {
    ngx_temp_file_t                  *temp_file;

    // 收到的数据都存在这个链表里
    // 最后一个节点b->last_buf = 1
    ngx_chain_t                      *bufs;

    // 当前使用的缓冲区
    ngx_buf_t                        *buf;

    // 剩余要读取的字节数
    // 对于确定长度（有content length）的就是r->headers_in.content_length_n
    // 在读取过程中会不断变化，最终为0
    off_t                             rest;

    off_t                             received;

    // 空闲节点链表，优化用，避免再向内存池要节点
    ngx_chain_t                      *free;

    ngx_chain_t                      *busy;

    // 读取chunk数据的结构体，用于ngx_http_parse_chunked()
    ngx_http_chunked_t               *chunked;

    // 当读取完毕后的回调函数
    // 即ngx_http_read_client_request_body的第二个参数
    ngx_http_client_body_handler_pt   post_handler;
    unsigned                          filter_need_buffering:1;
    unsigned                          last_sent:1;
    unsigned                          last_saved:1;
} ngx_http_request_body_t;
```
此结构存放在ngx_http_request_t结构体的request_body成员中。
ngx_http_read_client_request_body方法的流程图如下：

![](https://raw.githubusercontent.com/BaihlUp/Figurebed/master/2023/20230414103330.png)
如果下次再次触发可读事件，则变为调用ngx_http_do_read_client_request_body方法接收包体：

![](https://raw.githubusercontent.com/BaihlUp/Figurebed/master/2023/20230414103620.png)
在接收包体时需要根据配置文件中关于client_body_timeout配置项，配置相应的操作。

- 放弃接收包体

ngx_http_discard_request_body调用ngx_http_discarded_request_body_handler，nginx放弃接收包体，是会在接收完后再丢弃。

### 11.6 发送HTTP响应
ngx_http_send_header：发送响应头
ngx_http_output_filter方法用于发送响应包体， 它的第2个参数就是用于存放响应包体的缓冲区。
ngx_http_write_filter中会计算发送速率和根据sendfile_max_chunk进行处理。

ngx_http_send_header和ngx_http_output_filter都会调用所有模块注册在过滤链表上的处理方法，然后链表末尾都会调用ngx_http_write_filter函数真正发送数据。

Nginx执行的时候是怎么按照次序依次来执行各个过滤模块呢？它采用了一种很隐晦的方法，即通过局部的全局变量。比如，在每个filter模块，很可能看到如下代码：
```c
static ngx_http_output_header_filter_pt  ngx_http_next_header_filter;
static ngx_http_output_body_filter_pt    ngx_http_next_body_filter;

...

ngx_http_next_header_filter = ngx_http_top_header_filter;
ngx_http_top_header_filter = ngx_http_example_header_filter;

ngx_http_next_body_filter = ngx_http_top_body_filter;
ngx_http_top_body_filter = ngx_http_example_body_filter;
```
ngx_http_top_header_filter是一个全局变量。当编译进一个filter模块的时候，就被赋值为当前filter模块的处理函数。而ngx_http_next_header_filter是一个局部全局变量，它保存了编译前上一个filter模块的处理函数。所以整体看来，就像用全局变量组成的一条单向链表。
每个模块想执行下一个过滤函数，只要调用一下ngx_http_next_header_filter这个局部变量。而整个过滤模块链的入口，需要调用ngx_http_top_header_filter这个全局变量。ngx_http_top_body_filter的行为与header fitler类似。

响应头和响应体过滤函数的执行顺序如下所示：

![](https://raw.githubusercontent.com/BaihlUp/Figurebed/master/2023/20230414143010.png)

这图只表示了head_filter和body_filter之间的执行顺序，在header_filter和body_filter处理函数之间，在body_filter处理函数之间，可能还有其他执行代码。
nginx在发送数据时使用单链表，单链表负载的就是ngx_buf_t
```c
// 表示一个单块的缓冲区，既可以是内存也可以是文件
// start和end两个成员变量标记了数据所在内存块的边界
// 如果内存块是可以修改的，在操作时必须参考这两个成员防止越界
struct ngx_buf_s {
    u_char          *pos;           //内存数据的起始位置
    u_char          *last;          //内存数据的结束位置

    off_t            file_pos;      //文件数据的起始偏移量
    off_t            file_last;     //文件数据的结束偏移量

    u_char          *start;         /* start of buffer */   //内存数据的上界
    u_char          *end;           /* end of buffer */     //内存数据的下界

    ngx_buf_tag_t    tag;           //void*指针，可以是任意数据

    ngx_file_t      *file;          //存储数据的文件对象

    ngx_buf_t       *shadow;


    /* the buf's content could be changed */
    unsigned         temporary:1;   //内存块临时数据，可以修改

    /*
     * the buf's content is in a memory cache or in a read only memory
     * and must not be changed
     */
    unsigned         memory:1;      //内存块数据，不允许修改

    /* the buf's content is mmap()ed and must not be changed */
    unsigned         mmap:1;        //内存映射数据，不允许修改

    unsigned         recycled:1;
    unsigned         in_file:1;     //缓冲区在文件里
    unsigned         flush:1;       //要求Nginx立即输出本缓冲区
    unsigned         sync:1;        //要求Nginx同步操作本缓冲区
    unsigned         last_buf:1;    //最后一块缓冲区
    unsigned         last_in_chain:1;   //链里的最后一块缓冲区

    unsigned         last_shadow:1;
    unsigned         temp_file:1;       //缓冲区在临时文件里

    /* STUB */ int   num;
};

// 把缓冲区块简单地组织为一个单向链表
// 如果节点是链表的尾节点就必须要把next置为nullptr，表示链表结束
// ngx_chain_t (ngx_core.h)
struct ngx_chain_s {
    ngx_buf_t    *buf;      //缓冲区指针
    ngx_chain_t  *next;     //下一个链表节点
};

```
一般buffer结构体可以表示一块内存，内存的起始和结束地址分别用start和end表示，pos和last表示实际的内容。如果内容已经处理过了，pos的位置就可以往后移动。如果读取到新的内容，last的位置就会往后移动。所以buffer可以在多次调用过程中使用。如果last等于end，就说明这块内存已经用完了。如果pos等于last，说明内存已经处理完了。下面是一个简单的示意图，说明buffer中指针的用法：
![](https://raw.githubusercontent.com/BaihlUp/Figurebed/master/2023/20230414143222.png)

### 11.7 结束HTTP请求

## 12 upstream机制的设计与实现
### 12.1 upstream机制
upstream机制的场景示意图：

![](https://raw.githubusercontent.com/BaihlUp/Figurebed/master/2023/20230418141020.png)

upstream机制中两个核心结构体ngx_http_upstream_t和ngx_http_upstream_conf_t：
```c
// ngx_http_upstream_t
// 定义了upstream机制需要的所有信息
struct ngx_http_upstream_s {
    ngx_http_upstream_handler_pt     read_event_handler;
    ngx_http_upstream_handler_pt     write_event_handler;

    // 连接结构体
    ngx_peer_connection_t            peer;

    ngx_event_pipe_t                *pipe;

    // 发送的请求数据
    // u->request_bufs = r->request_body->bufs;
    ngx_chain_t                     *request_bufs;

    ngx_output_chain_ctx_t           output;
    ngx_chain_writer_ctx_t           writer;

    // 上游的连接参数设置
    ngx_http_upstream_conf_t        *conf;
    ngx_http_upstream_srv_conf_t    *upstream;
#if (NGX_HTTP_CACHE)
    ngx_array_t                     *caches;
#endif

    // 上游的响应头
    ngx_http_upstream_headers_in_t   headers_in;

    // 上游服务器的地址
    ngx_http_upstream_resolved_t    *resolved;

    ngx_buf_t                        from_client;

    // 数据缓冲区
    ngx_buf_t                        buffer;

    // 缓冲数据的长度
    off_t                            length;

    // 从上游接收到的数据
    ngx_chain_t                     *out_bufs;

    ngx_chain_t                     *busy_bufs;
    ngx_chain_t                     *free_bufs;

    // 处理上游服务器响应数据的回调函数
    ngx_int_t                      (*input_filter_init)(void *data);
    ngx_int_t                      (*input_filter)(void *data, ssize_t bytes);
    void                            *input_filter_ctx;

#if (NGX_HTTP_CACHE)
    ngx_int_t                      (*create_key)(ngx_http_request_t *r);
#endif

    // 发送接收请求的回调函数，9个
    ngx_int_t                      (*create_request)(ngx_http_request_t *r);
    ngx_int_t                      (*reinit_request)(ngx_http_request_t *r);
    ngx_int_t                      (*process_header)(ngx_http_request_t *r);
    void                           (*abort_request)(ngx_http_request_t *r);
    void                           (*finalize_request)(ngx_http_request_t *r,
                                         ngx_int_t rc);
    ngx_int_t                      (*rewrite_redirect)(ngx_http_request_t *r,
                                         ngx_table_elt_t *h, size_t prefix);
    ngx_int_t                      (*rewrite_cookie)(ngx_http_request_t *r,
                                         ngx_table_elt_t *h);

    ngx_msec_t                       start_time;

    // 处理的状态信息
    ngx_http_upstream_state_t       *state;

    ngx_str_t                        method;
    ngx_str_t                        schema;
    ngx_str_t                        uri;

#if (NGX_HTTP_SSL || NGX_COMPAT)
    ngx_str_t                        ssl_name;
#endif

    ngx_http_cleanup_pt             *cleanup;

    unsigned                         store:1;
    unsigned                         cacheable:1;
    unsigned                         accel:1;
    unsigned                         ssl:1;
#if (NGX_HTTP_CACHE)
    unsigned                         cache_status:3;
#endif

    // 是否使用更多的缓冲区来接收上游的数据
    unsigned                         buffering:1;

    unsigned                         keepalive:1;
    unsigned                         upgrade:1;
    unsigned                         error:1;

    // 是否已经发送请求
    unsigned                         request_sent:1;

    // 是否已经发送请求体数据
    unsigned                         request_body_sent:1;

    unsigned                         request_body_blocked:1;
    // 是否已经发送响应头
    unsigned                         header_sent:1;
};
```

### 12.2 启动upstream
通过ngx_http_upstream_create方法创建ngx_http_upstream_t结构体，其中的成员还需要各个http模块自行设置。
```c
ngx_int_t ngx_http_upstream_create(ngx_http_request_t *r)
```
启动upstream机制的ngx_http_upstream_init方法定义如下：
```c
// 启动upstream框架，开始与上游服务器异步交互
void ngx_http_upstream_init(ngx_http_request_t *r)
```
ngx_http_upstream_init方法的流程如下：

![](https://raw.githubusercontent.com/BaihlUp/Figurebed/master/2023/20230418142114.png)

### 12.3 与上游服务器建立连接
为了保证建立TCP连接这个操作不会阻塞进程， Nginx使用无阻塞的套接字来连接上游服务器。
调用的ngx_http_upstream_connect方法就是用来连接上游服务器的， 由于使用了非阻塞的套接字， 当方法返回时与上游之间的TCP连接未必会成功建立， 可能还需要等待上游服务器返回TCP的SYN/ACK包。
```c
// 尝试连接后端服务器
static void ngx_http_upstream_connect(ngx_http_request_t *r, ngx_http_upstream_t *u)
```
流程图如下：
![](https://raw.githubusercontent.com/BaihlUp/Figurebed/master/2023/20230418143131.png)
1. 上边把此连接 ngx_connection_t的读写事件都设置为了ngx_http_upstream_handler
2. 将upstream机制的write_event_handler方法设置为ngx_http_upstream_send_request_handler，此方法会多次触发，实际还是调用ngx_http_upstream_send_request方法发送。
3. upstream的read_event_handler方法设置为ngx_http_upstream_process_header
4. 连接建立成功后，则调用ngx_http_upstream_send_request方法

ngx_http_upstream_send_request方法的流程图：

![](https://raw.githubusercontent.com/BaihlUp/Figurebed/master/2023/20230418144312.png)

### 12.4 接收上游服务器的响应头部
在上边的ngx_http_upstream_send_request方法中，当请求全部发送给上游服务器时，开始准备接收来自上游服务器的响应。由ngx_http_upstream_process_header方法处理上游服务器的响应，此方法也会多次被调用。
Nginx可以代理多种不同的协议，分为两段方式，先处理响应头，然后处理响应体。
处理包体分为3种不同的方式：
1. 不转发响应：不转发包体是upstream机制最基本的功能， 特别是客户端请求派生出的子请求多半不需要转发包体。
2. 转发响应时下游网速优先
3. 转发响应时上游网速优先


### 12.5 以下游网速优先来转发响应
转发上游服务器的响应到下游客户端，必然由上游事件来驱动，下游网速优先实际上意味着需要开辟一块固定长度的内存作为缓冲区。
#### 12.5.1 转发响应的包头
在ngx_http_upstream_send_response方法中完成的，处理流程如下

![](https://raw.githubusercontent.com/BaihlUp/Figurebed/master/2023/20230418153127.png)

通过调用ngx_http_send_header方法向客户端发送HTTP包头，会调用header过滤链表，走一下所有模块：
```c
// 发送头，调用ngx_http_top_header_filter
// 如果请求处理有错误，修改输出的状态码
// 状态行同时清空
// 走过整个header过滤链表
ngx_int_t
ngx_http_send_header(ngx_http_request_t *r)
{
    if (r->post_action) {
        return NGX_OK;
    }

    // 检查是否已经发送过，会出个alert级别的错误，但其实无必要
    if (r->header_sent) {
        ngx_log_error(NGX_LOG_ALERT, r->connection->log, 0,
                      "header already sent");
        return NGX_ERROR;
    }

    // 如果请求处理有错误，修改输出的状态码
    // 状态行同时清空
    if (r->err_status) {
        r->headers_out.status = r->err_status;
        r->headers_out.status_line.len = 0;
    }

    // 发送头，调用ngx_http_top_header_filter
    // 走过整个header过滤链表
    return ngx_http_top_header_filter(r);
}
```

在方法中会判断配置的buffering标志，若为0，表示以下游网速优先，如果为1，则会以上游网速优先，因为上游网速一般比下游网速快很多，所有需要更大的缓冲区保存，如果达到上限，以磁盘文件的形式来缓存来不及向下游转发的响应。

![](https://raw.githubusercontent.com/BaihlUp/Figurebed/master/2023/20230418153520.png)

#### 12.5.2 转发响应包体
如果buffering为0，则后边转发响应包体将会由ngx_http_upstream_process_non_buffered_upstream方法处理连接上的都事件。
无论是接收上游服务器的响应， 还是向下游客户端发送响应， 最终调用的方法都是ngx_http_upstream_process_non_buffered_request， 唯一的区别是该方法的第2个参数不同， 当需要读取上游的响应时传递的是0， 当需要向下游发送响应时传递的是1。 

ngx_http_upstream_process_non_buffered_request的流程图：

![](https://raw.githubusercontent.com/BaihlUp/Figurebed/master/2023/20230418154143.png)

ngx_http_upstream_process_non_buffered_request方法中调用ngx_http_output_filter方法，走过整个body过滤链表：
```c
// 发送响应体，调用ngx_http_top_body_filter
// 走过整个body过滤链表
// 最后由ngx_http_write_filter真正的向客户端发送数据，调用send_chain
// 也由ngx_http_set_write_handler设置epoll的写事件触发
// 如果数据发送不完，就保存在r->out里，返回again
// 需要再次发生可写事件才能发送
// 不是last、flush，且数据量较小（默认1460）
// 那么这次就不真正调用write发送，减少系统调用的次数，提高性能
ngx_int_t
ngx_http_output_filter(ngx_http_request_t *r, ngx_chain_t *in)
{
    ngx_int_t          rc;
    ngx_connection_t  *c;

    c = r->connection;

    ngx_log_debug2(NGX_LOG_DEBUG_HTTP, c->log, 0,
                   "http output filter \"%V?%V\"", &r->uri, &r->args);

    // 发送响应体，调用ngx_http_top_body_filter
    // 走过整个body过滤链表
    // 最后由ngx_http_write_filter真正的向客户端发送数据，调用send_chain
    rc = ngx_http_top_body_filter(r, in);

    if (rc == NGX_ERROR) {
        /* NGX_ERROR may be returned by any filter */
        c->error = 1;
    }

    return rc;
}
```
### 12.6 以上游网速优先来转发响应
如果将ngx_http_upstream_conf_t配置结构体的buffering标志位设置为1， 那么ngx_event_pipe_t结构体必须要由HTTP模块创建。
ngx_event_pipe_t结构体维护着上下游间转发的响应包体， 它相当复杂。 例如， 缓冲区链表ngx_chain_t类型的成员就定义了6个（包括free_raw_bufs、 in、 out、 free、 busy、 preread_bufs） ， 为什么要用如此复杂的数据结构支撑看似简单的转发过程呢？ 这是因为Nginx的宗旨就是高效率， 所以它绝不会把相同内容复制到两块内存中， 而同一块内存如果既要用于接收上游发来的响应， 又要准备向下游发送， 很可能还要准备写入临时文件中， 这就带来了很高的复杂度， ngx_event_pipe_t结构体的任务就在于解决这个问题。

- 转发响应包头

转发响应包头还是通过调用ngx_http_upstream_send_response方法，
```c
    u->read_event_handler = ngx_http_upstream_process_upstream;
    r->write_event_handler = ngx_http_upstream_process_downstream;
```
方法中设置处理上游读事件回调方法为ngx_http_upstream_process_upstream。设置处理下游写事件的回调方法为ngx_http_upstream_process_downstream。

- 转发响应包头

处理上游读事件的方法是ngx_http_upstream_process_upstream， 处理下游写事件的方法是ngx_http_upstream_process_downstream， 但它们最终都是通过ngx_event_pipe方法实现缓存转发响应功能的。

### 12.7 结束upstream请求
结束upstream请求调用ngx_http_upstream_finalize_request方法完成
```c
// 结束请求，这时会调用finalize_request回调函数
// 让upstream模块有机会做一些自己的收尾工作
// 最后调用ngx_http_finalize_request
static void
ngx_http_upstream_finalize_request(ngx_http_request_t *r,
    ngx_http_upstream_t *u, ngx_int_t rc)
```


## 14 进程间的通信机制
### 14.1 概述
Nginx框架使用了3种消息传递方式： 共享内存、 套接字、 信号。 

### 14.2 共享内存
#### 14.2.1 共享内存创建和销毁
Linux提供了mmap和shmget系统调用在内存中创建一块连续的线性地址空间，通过munmap或shmdt系统调用释放这块内存。
nginx定义了ngx_shm_t结构体描述共享内存。
```c
// 真正操作共享内存的对象
typedef struct {
    // 共享内存的开始地址
    // 通常就是ngx_slab_pool_t
    u_char      *addr;

    // 共享内存的大小
    size_t       size;

    // 共享内存的名字
    ngx_str_t    name;

    ngx_log_t   *log;

    // 是否存在，即已经创建过了
    // 依据nginx官方文档，此字段仅用于windows
    ngx_uint_t   exists;   /* unsigned  exists:1;  */
} ngx_shm_t;
```
nginx中使用如下方式创建和释放共享内存：
```c
// 创建一块共享内存
ngx_int_t
ngx_shm_alloc(ngx_shm_t *shm)
{
    // MAP_ANON|MAP_SHARED
    shm->addr = (u_char *) mmap(NULL, shm->size,
                                PROT_READ|PROT_WRITE,
                                MAP_ANON|MAP_SHARED, -1, 0);

    if (shm->addr == MAP_FAILED) {
        ngx_log_error(NGX_LOG_ALERT, shm->log, ngx_errno,
                      "mmap(MAP_ANON|MAP_SHARED, %uz) failed", shm->size);
        return NGX_ERROR;
    }

    return NGX_OK;
}


// 销毁共享内存
void
ngx_shm_free(ngx_shm_t *shm)
{
    if (munmap((void *) shm->addr, shm->size) == -1) {
        ngx_log_error(NGX_LOG_ALERT, shm->log, ngx_errno,
                      "munmap(%p, %uz) failed", shm->addr, shm->size);
    }
}
```
以上是使用mmap创建，关于mmap的函数原型如下：
```c
void *mmap(void *start, size_t length, int prot, int flags, int fd, off_t offset);
```
mmap可以将磁盘文件映射到内存中， 直接操作内存时Linux内核将负责同步内存和磁盘文件中的数据， fd参数就指向需要同步的磁盘文件， 而offset则代表从文件的这个偏移量处开始共享， 当然Nginx没有使用这一特性。 当flags参数中加入MAP_ANON或者MAP_ANONYMOUS参数时表示不使用文件映射方式， 这时fd和offset参数就没有意义， 也不需要传递了， 此时的mmap方法和ngx_shm_alloc的功能几乎完全相同。length参数就是将要在内存中开辟的线性地址空间大小， 而prot参数则是操作这段共享内存的方式（如只读或者可读可写） ， start参数说明希望的共享内存起始映射地址， 当然， 通常都会把start设为NULL空指针。

同样创建共享内存nginx中也提供了使用shmget：
```c
ngx_int_t
ngx_shm_alloc(ngx_shm_t *shm)
{
    int  id;

    id = shmget(IPC_PRIVATE, shm->size, (SHM_R|SHM_W|IPC_CREAT));

    if (id == -1) {
        ngx_log_error(NGX_LOG_ALERT, shm->log, ngx_errno,
                      "shmget(%uz) failed", shm->size);
        return NGX_ERROR;
    }

    ngx_log_debug1(NGX_LOG_DEBUG_CORE, shm->log, 0, "shmget id: %d", id);

    shm->addr = shmat(id, NULL, 0);

    if (shm->addr == (void *) -1) {
        ngx_log_error(NGX_LOG_ALERT, shm->log, ngx_errno, "shmat() failed");
    }

    if (shmctl(id, IPC_RMID, NULL) == -1) {
        ngx_log_error(NGX_LOG_ALERT, shm->log, ngx_errno,
                      "shmctl(IPC_RMID) failed");
    }

    return (shm->addr == (void *) -1) ? NGX_ERROR : NGX_OK;
}
```

#### 14.2.2 共享内存使用实战--监控
ngx_http_stub_status_module模块对连接的状态监控就用到了共享内存，因为连接的状况展示的是多个worker进程的统计情况。

模块中使用共享内存保存各种统计指标，在统计过程中使用原子操作对统计指标进行修改。
```c
// 在ngx_init_cycle里调用，fork子进程之前
// 创建共享内存，存放负载均衡锁和统计用的原子变量
static ngx_int_t
ngx_event_module_init(ngx_cycle_t *cycle)
{
........
    // 分配一块共享内存
    if (ngx_shm_alloc(&shm) != NGX_OK) {
        return NGX_ERROR;
    }

    // shared是共享内存的地址指针
    shared = shm.addr;

    // 第一个就是负载均衡锁
    ngx_accept_mutex_ptr = (ngx_atomic_t *) shared;

    // spin是-1则不使用信号量
    // 只会自旋，不会导致进程睡眠等待
    // 这样避免抢accept锁时的性能降低
    ngx_accept_mutex.spin = (ngx_uint_t) -1;

    // 初始化互斥锁
    // spin是-1则不使用信号量
    // 只会自旋，不会导致进程睡眠等待
    if (ngx_shmtx_create(&ngx_accept_mutex, (ngx_shmtx_sh_t *) shared,
                         cycle->lock_file.data)
        != NGX_OK)
    {
        return NGX_ERROR;
    }

    // 连接计数器
    ngx_connection_counter = (ngx_atomic_t *) (shared + 1 * cl);

    // 计数器置1
    (void) ngx_atomic_cmp_set(ngx_connection_counter, 0, 1);

    ngx_log_debug2(NGX_LOG_DEBUG_EVENT, cycle->log, 0,
                   "counter: %p, %uA",
                   ngx_connection_counter, *ngx_connection_counter);

    // 临时文件用
    ngx_temp_number = (ngx_atomic_t *) (shared + 2 * cl);

    tp = ngx_timeofday();

    // 随机数
    // 每个进程不同
    ngx_random_number = (tp->msec << 16) + ngx_pid;

#if (NGX_STAT_STUB)

    ngx_stat_accepted = (ngx_atomic_t *) (shared + 3 * cl);
    ngx_stat_handled = (ngx_atomic_t *) (shared + 4 * cl);
    ngx_stat_requests = (ngx_atomic_t *) (shared + 5 * cl);
    ngx_stat_active = (ngx_atomic_t *) (shared + 6 * cl);
    ngx_stat_reading = (ngx_atomic_t *) (shared + 7 * cl);
    ngx_stat_writing = (ngx_atomic_t *) (shared + 8 * cl);
    ngx_stat_waiting = (ngx_atomic_t *) (shared + 9 * cl);

#endif

```
### 14.3 原子操作
#### 14.3.1 原子操作方法
原子操作在不同的架构下实现方式不同，下边是nginx在x86架构下使用嵌入汇编实现。
```c
static ngx_inline ngx_atomic_uint_t
ngx_atomic_cmp_set(ngx_atomic_t *lock, ngx_atomic_uint_t old,
    ngx_atomic_uint_t set)
{
    u_char  res;

    __asm__ volatile (

        //锁住总线
         NGX_SMP_LOCK
    "    cmpxchgl  %3, %1;   "
    "    sete      %0;       "

    : "=a" (res) : "m" (*lock), "a" (old), "r" (set) : "cc", "memory");

    return res;
}
```

```c
//比较lock和old的值，如果相等，则把lock设置为set
ngx_atomic_cmp_set(ngx_atomic_t *lock, ngx_atomic_uint_t old,
    ngx_atomic_uint_t set)；
//对value的值加上add
ngx_atomic_fetch_add(ngx_atomic_t *value, ngx_atomic_int_t add)；
```

#### 14.3.2 自旋锁
nginx中基于原子操作实现了spinlock自旋锁，自旋锁不会导致进程睡眠，当发现锁已经被其他进程获得时，则始终保持进程在可执行状态，每当内核调度到这个进程执行时就持续检查是否可以获取到锁。
```c
// 自旋锁，尽量不让出cpu抢锁
// 操作原子变量，设置为值value
// spin通常是2048,即2^11
// 目前仅在线程池里需要使用自旋锁
void
ngx_spinlock(ngx_atomic_t *lock, ngx_atomic_int_t value, ngx_uint_t spin)
{

    // 必须支持原子操作，否则无法编译
#if (NGX_HAVE_ATOMIC_OPS)

    ngx_uint_t  i, n;

    for ( ;; ) {

        // 检查lock值，为0表示没有被锁
        // 使用cas操作赋值，成功则获得锁
        // 不会阻塞,失败则继续后续代码
        // 相当于try_lock
        if (*lock == 0 && ngx_atomic_cmp_set(lock, 0, value)) {
            return;
        }

        // 多核cpu，不必让出cpu，等待一下
        if (ngx_ncpu > 1) {

            // n按2的幂增加
            for (n = 1; n < spin; n <<= 1) {

                // cpu等待的时间逐步加长
                for (i = 0; i < n; i++) {
                    // #define ngx_cpu_pause()             __asm__ ("pause")
                    // 自旋等待，降低功耗，不会引起性能下降
                    ngx_cpu_pause();
                }

                // 再次try_lock
                if (*lock == 0 && ngx_atomic_cmp_set(lock, 0, value)) {
                    return;
                }
            }
        }

        // 占用cpu过久，让出cpu
        // 单cpu必须让出cpu让其他进程运行
        // 之后继续try_lock，直至lock成功
        // yield不会进入睡眠
        ngx_sched_yield();
    }

// not NGX_HAVE_ATOMIC_OPS
#else

// 使用了--with-threads开启多线程功能
// 但没有原子操作，则无法通过编译
// 因为线程池需要使用自旋锁
#if (NGX_THREADS)

#error ngx_spinlock() or ngx_atomic_cmp_set() are not defined !

#endif

// 不使用线程池，则自旋锁是空实现

#endif

}
```

### 14.4 Nginx频道 
ngx_channel_t频道是Nginx master进程与worker进程之间通信的常用工具， 它是使用本机套接字实现的。 下面先来看看socketpair方法， 它用于创建父子进程间使用的套接字。
```c
int socketpair(int d, int type, int protocol, int sv[2]);
```
这个方法可以创建一对关联的套接字sv[2]。 下面依次介绍它的4个参数： 参数d表示域， 在Linux下通常取值为AF_UNIX； type取值为SOCK_STREAM或者SOCK_DGRAM， 它表示在套接字上使用的是TCP还是UDP； protocol必须传递0； sv[2]是一个含有两个元素的整型数组，实际上就是两个套接字。 当socketpair返回0时， sv[2]这两个套接字创建成功， 否则socketpair返回–1表示失败。
nginx中的ngx_channel结构如下：
```c
typedef struct {
    //传递的tcp消息中的命令
    ngx_uint_t  command;
    //进程PID，一般是发送命令方的进程
    ngx_pid_t   pid;
    //表示发送命令方在ngx_processes进程数组间的进程序号
    ngx_int_t   slot;
    //通信的套接字句柄
    ngx_fd_t    fd;
} ngx_channel_t;
```
在nginx中的master进程中创建worker进程前执行创建channel
```c
ngx_pid_t
ngx_spawn_process(ngx_cycle_t *cycle, ngx_spawn_proc_pt proc, void *data,
    char *name, ngx_int_t respawn)
{
....
     // 创建进程间通信用的channel
    if (respawn != NGX_PROCESS_DETACHED) {

        /* Solaris 9 still has no AF_LOCAL */

        // 创建socketpair，进程间通信用
        if (socketpair(AF_UNIX, SOCK_STREAM, 0, ngx_processes[s].channel) == -1)
        {
            ngx_log_error(NGX_LOG_ALERT, cycle->log, ngx_errno,
                          "socketpair() failed while spawning \"%s\"", name);
            return NGX_INVALID_PID;
        }

        ngx_log_debug2(NGX_LOG_DEBUG_CORE, cycle->log, 0,
                       "channel %d:%d",
                       ngx_processes[s].channel[0],
                       ngx_processes[s].channel[1]);

        // 进程间通信非阻塞
        if (ngx_nonblocking(ngx_processes[s].channel[0]) == -1) {
            ngx_log_error(NGX_LOG_ALERT, cycle->log, ngx_errno,
                          ngx_nonblocking_n " failed while spawning \"%s\"",
                          name);
            ngx_close_channel(ngx_processes[s].channel, cycle->log);
            return NGX_INVALID_PID;
        }
}
```
nginx中目前只是master向worker进程发送，worker进程接收，其实socketpair是双向通信，但目前nginx没有worker向master进程发送的。

操作channel的函数
```c
ngx_int_t ngx_write_channel(ngx_socket_t s, ngx_channel_t *ch, size_t size,
    ngx_log_t *log);
ngx_int_t ngx_read_channel(ngx_socket_t s, ngx_channel_t *ch, size_t size,
    ngx_log_t *log);
ngx_int_t ngx_add_channel_event(ngx_cycle_t *cycle, ngx_fd_t fd,
    ngx_int_t event, ngx_event_handler_pt handler);
void ngx_close_channel(ngx_fd_t *fd, ngx_log_t *log);
```
nginx的master进程通过channel向worker进程发送退出、重新打开进程已经打开过的文件等信号，如果使用channel发送失败，则master进程会提供kill系统调用发送。

### 14.5 信号
nginx接收信号，执行不同指令，如接收到SIGUSR1信号就意味着需要重新打开文件。
定义信号的结构体如下：
```c
// 标记unix信号，handler=ngx_signal_handler
typedef struct {
    //需要处理的信号
    int     signo;
    //信号对应的字符串名称
    char   *signame;
    //信号对应的Nginx命令
    char   *name;

    // 原接口：void  (*handler)(int signo);
    // 1.13.0 变动了函数接口
    // 可以多获取一些信号的信息
    //收到信号后执行的回调函数
    void  (*handler)(int signo, siginfo_t *siginfo, void *ucontext);
} ngx_signal_t;
```

Nginx定义了一个数组，用来定义进程将会处理的所有信号：
```c
// 命令行-s参数关联数组
// 所有信号都用ngx_signal_handler处理
ngx_signal_t  signals[] = {
    // #define NGX_RECONFIGURE_SIGNAL   HUP
    // 即sighup
    { ngx_signal_value(NGX_RECONFIGURE_SIGNAL),
      "SIG" ngx_value(NGX_RECONFIGURE_SIGNAL),
      "reload",
      ngx_signal_handler },

    // #define NGX_REOPEN_SIGNAL        USR1
    // 即sigusr1
    { ngx_signal_value(NGX_REOPEN_SIGNAL),
      "SIG" ngx_value(NGX_REOPEN_SIGNAL),
      "reopen",
      ngx_signal_handler },

    { ngx_signal_value(NGX_NOACCEPT_SIGNAL),
      "SIG" ngx_value(NGX_NOACCEPT_SIGNAL),
      "",
      ngx_signal_handler },

    // #define NGX_TERMINATE_SIGNAL     TERM
    // sigterm
    { ngx_signal_value(NGX_TERMINATE_SIGNAL),
      "SIG" ngx_value(NGX_TERMINATE_SIGNAL),
      "stop",
      ngx_signal_handler },

    // #define NGX_SHUTDOWN_SIGNAL      QUIT
    // sigquit
    { ngx_signal_value(NGX_SHUTDOWN_SIGNAL),
      "SIG" ngx_value(NGX_SHUTDOWN_SIGNAL),
      "quit",
      ngx_signal_handler },

    // ngx_config.h
    // #define NGX_CHANGEBIN_SIGNAL     USR2
    // hot upgrade, kill -s SIGUSR2 masterpid
    { ngx_signal_value(NGX_CHANGEBIN_SIGNAL),
      "SIG" ngx_value(NGX_CHANGEBIN_SIGNAL),
      "",
      ngx_signal_handler },

    { SIGALRM, "SIGALRM", "", ngx_signal_handler },

    { SIGINT, "SIGINT", "", ngx_signal_handler },

    { SIGIO, "SIGIO", "", ngx_signal_handler },

    { SIGCHLD, "SIGCHLD", "", ngx_signal_handler },

    { SIGSYS, "SIGSYS, SIG_IGN", "", NULL },

    { SIGPIPE, "SIGPIPE, SIG_IGN", "", NULL },

    { 0, NULL, "", NULL }
};
```
以上的所有信号在ngx_init_signals方法中初始化：
```c
// 初始化signals数组
ngx_int_t ngx_init_signals(ngx_log_t *log);
```

### 14.6 信号量
信号量与信号不同，信号用来传递消息，信号量用来保证两个或多个代码段不被并发访问，是一种共享资源有序访问的工具。
nginx中创建和销毁信号量的函数：
```c
// 初始化互斥锁
// spin是-1则不使用信号量
// 只会自旋，不会导致进程睡眠等待
ngx_int_t ngx_shmtx_create(ngx_shmtx_t *mtx, ngx_shmtx_sh_t *addr,
    u_char *name);

// 销毁使用的信号量
// spin是-1则不使用信号量
void ngx_shmtx_destroy(ngx_shmtx_t *mtx);
```
信号量是如何实现互斥锁功能的呢？ 例如， 最初的信号量sem值为0， 调用sem_post方法将会把sem值加1， 这个操作不会有任何阻塞； 调用sem_wait方法将会把信号量sem的值减1， 如果sem值已经小于或等于0了， 则阻塞住当前进程（进程会进入睡眠状态） ， 直到其他进程将信号量sem的值改变为正数后， 这时才能继续通过将sem减1而使得当前进程继续向下执行。 因此， sem_post方法可以实现解锁的功能， 而sem_wait方法可以实现加锁的功能。

在ngx_shmtx_lock中可能用到信号量中sem_wait试图获取锁。

### 14.7 文件锁
Linux内核提供了基于文件的互斥锁， 而Nginx框架封装了3个方法， 提供给Nginx模块使用文件互斥锁来保护共享数据。 下面首先介绍一下这种基于文件的互斥锁是如何使用的， 其实很简单， 通过fcntl方法就可以实现。
```c
int fcntl(int fd, int cmd, struct flock *lock);
```
其中参数fd是打开的文件句柄， 参数cmd表示执行的锁操作， 参数lock描述了这个锁的信息。
参数fd必须是已经成功打开的文件句柄。 实际上， nginx.conf文件中的lock_file配置项指定的文件路径， 就是用于文件互斥锁的， 这个文件被打开后得到的句柄， 将会作为fd参数传递给fcntl方法， 提供一种锁机制。
这里的cmd参数在Nginx中只会有两个值： F_SETLK和F_SETLKW， 它们都表示试图获得互斥锁， 但使用F_SETLK时如果互斥锁已经被其他进程占用， fcntl方法不会等待其他进程释放锁且自己拿到锁后才返回， 而是立即返回获取互斥锁失败； 使用F_SETLKW时则不同， 锁被占用后fcntl方法会一直等待， 在其他进程没有释放锁时， 当前进程就会阻塞在fcntl方法中， 这种阻塞会导致当前进程由可执行状态转为睡眠状态。

关于文件锁的实现函数：
```
ngx_err_t ngx_trylock_fd(ngx_fd_t fd);
ngx_err_t ngx_lock_fd(ngx_fd_t fd);
ngx_err_t ngx_unlock_fd(ngx_fd_t fd);
```
在使用文件锁是要注意是否会导致进程睡眠，根据实际情况抉择。

### 14.8 互斥锁
基于原子操作、信号量、文件锁，nginx在更高层次封装了一个互斥锁，许多Nginx模块也是更多直接使用它。操作方法如下：

![](https://raw.githubusercontent.com/BaihlUp/Figurebed/master/2023/20230410164924.png)

互斥锁接口的内部实现中使用了原子操作、信号量和文件锁，可以通过参数控制使用什么逻辑。
以上接口都是通过操作ngx_shmtx_t类型的结构体来实现互斥操作：
```c
// ngx_shmtx_sh_t
// 互斥锁使用的两个原子变量
typedef struct {

    // 锁变量
    // 使用原子操作实现锁
    ngx_atomic_t   lock;

    // 信号量等待变量
    // 标记等待的进程数量
#if (NGX_HAVE_POSIX_SEM)
    ngx_atomic_t   wait;
#endif
} ngx_shmtx_sh_t;


// ngx_shmtx_t
typedef struct {
#if (NGX_HAVE_ATOMIC_OPS)
    // 指向ngx_shmtx_sh_t.lock
    ngx_atomic_t  *lock;

    // 使用进程间信号量

#if (NGX_HAVE_POSIX_SEM)
    // 指向ngx_shmtx_sh_t.wait
    ngx_atomic_t  *wait;

    // 是否使用信号量的标志
    // 可以手动置0强制不使用信号量
    ngx_uint_t     semaphore;

    // unix信号量对象
    sem_t          sem;
#endif

    // 不会使用文件锁
#else
    ngx_fd_t       fd;
    u_char        *name;
#endif

    // 类似自旋锁的等待周期
    // spin是-1则不使用信号量
    // 只会自旋，不会导致进程睡眠等待
    // 目前只有accept_mutex使用了-1
    ngx_uint_t     spin;
} ngx_shmtx_t;
```
在函数中Nginx也会判断当前环境是否支持原子操作，信号量、文件锁等。然后执行对应的分支。

### 14.9 总结
Nginx是一个能够并发处理几十万甚至几百万个TCP连接的高性能服务器， 因此， 在进行进程间通信时， 必须充分考虑到不能过分影响正常请求的处理。 例如， 使用14.4节介绍的套接字通信时， 套接字都被设为了无阻塞模式， 防止执行时阻塞了进程导致其他请求得不到处理， 又如， Nginx封装的锁都不会直接使用信号量， 因为一旦获取信号量互斥锁失败， 进程就会进入睡眠状态， 这会导致其他请求“饿死”。
当用户开发复杂的Nginx模块时， 可能会涉及不同的worker进程间通信， 这时可以从本章介绍的进程间通信方式上进行选择， 从使用上说， ngx_shmtx_t互斥锁和共享内存应当是第三方Nginx模块最常用的进程间通信方式了， ngx_shmtx_t互斥锁在实现中充分考虑了是否引发睡眠的问题， 用户在使用时需要明确地判断出是否会引发进程睡眠。 当然， 如果不使用Nginx封装过的进程间通信方式， 则需要注意跨平台，以及是否会阻塞进程的运行等问题。


## 16 slab共享内存
在Nginx中多个Woker进程共享数据，如果是简单的进程间通信，可以使用以上的方式，如果需要共享不同大小的结构对象，如链表、树、图等，可以通过一段共享内存进行共享，为了高效的管理共享内存，Nginx使用了slab内存管理机制。
### 16.1 操作slab的方法
slab中只有下边5个接口：
```c
// 初始化新创建的共享内存
void ngx_slab_init(ngx_slab_pool_t *pool);
// 加锁保护的内存分配方法
void *ngx_slab_alloc(ngx_slab_pool_t *pool, size_t size);
// 不加锁保护的内存分配方法
void *ngx_slab_alloc_locked(ngx_slab_pool_t *pool, size_t size);
// 加锁保护的内存释放方法
void ngx_slab_free(ngx_slab_pool_t *pool, void *p);
// 不加锁保护的内存释放方法
void ngx_slab_free_locked(ngx_slab_pool_t *pool, void *p);
```

### 16.2 使用slab共享内存示例（未看）

### 16.3 slab内存管理实现原理
slab中把整块内存按4KB分整许多页，每一页只存固定大小的内存块，由于一页上能够分配的内存块数量是有限的，可以在页首用bitmap方式，按二进制位表示页对应位置的内存块是否在使用中。只是遍历bitmap二进制位去寻找页上的空闲内存块， 使得消耗的时间很有限， 例如bitmap占用的内存空间小导致CPU缓存命中率高， 可以按32或64位这样的总线长度去寻找空闲位以减少访问次数等。
关于对页的管理，分为空闲页、半满页和全满页，不同的页通过链表维护。
slab会把一页分成不同的内存块大小，内存块分为8，16，32，64.。。。字节。当申请的字节数大于8小于等于16时， 就会使用16字节的内存块， 以此类推。
按照不同页中含有的内存块大小分类，然后包含相同内存块大小的页组成页链表，并且页的首部放在slots数组中，slots数组也是按序排列，比如开始元素存放的地址是8字节内存块所属页的链表，依次递增。

![](https://raw.githubusercontent.com/BaihlUp/Figurebed/master/2023/20230411111313.png)

上图包含了空闲页、半满页、全满页的链表和分别存在两个slot中。

slab中使用ngx_slab_pool_t结构管理共享内存，在常见slab后，ngx_slab_pool_t结构存储在共享内存开始位置，并且通过初始化结构中的属性管理后边的内存。结构如下：

```c
// ngx_slab_pool_t
// 管理共享内存的池
// 存放page管理信息、空闲页数量、共享内存的开始地址等
// 64位系统上占用200个字节
// 使用best fit算法
// 分成8/16/32...2k/4k的多个slot，找最合适的分配
// 但也可以直接管理内部的非共享内存
// 不使用锁即可
typedef struct {
    // 互斥锁使用的两个原子变量
    ngx_shmtx_sh_t    lock;

    // 最小分配数量，通常是8字节
    size_t            min_size;

    // 最小左移，通常是3，即2^3=8
    // ngx_init_zone_pool里设置
    // 在shm_zone[i].init之前，不能自己修改
    size_t            min_shift;

    // 页数组
    // 4k大小，对齐管理内存
    ngx_slab_page_t  *pages;

    // 页链表指针，最后一页
    // 用于合并空闲页的末尾计算
    ngx_slab_page_t  *last;

    // 空闲页链表头节点
    // 也作为链表的尾节点哨兵
    // 注意不是指针
    ngx_slab_page_t   free;

    // 统计信息数组
    // 在slots之后
    // 目前供商业模块ngx_api来调用
    // 目前暂无公开接口使用
    // 只能自己定位获取信息
    ngx_slab_stat_t  *stats;

    // 空闲页数量
    ngx_uint_t        pfree;

    // 共享内存的开始地址
    // 经过了多次计算，前面有很多管理信息
    u_char           *start;

    // 共享内存的末尾地址
    // 使用start和end来判断指针是否属于本内存
    u_char           *end;

    // 互斥锁
    // mtx.lock指向sh.lock
    // ngx_shmtx_create():mtx->lock = &addr->lock;
    ngx_shmtx_t       mutex;

    // 记录日志的额外字符串，用户可以指定
    // 共享内存错误记录日志时区分不同的共享内存
    // 不指定则指向zero，即无特殊字符串
    // 被ngx_slab_error使用，外界不能用
    u_char           *log_ctx;

    // '\0'字符
    u_char            zero;

    // 是否记录无内存异常
    // 可以置为0,减少记录日志的操作
    unsigned          log_nomem:1;

    // 供用户使用，关联任意数据
    // 方便使用本内存里最常用的数据
    // 例如红黑树指针
    void             *data;

    // 内存的起始地址
    // 在ngx_init_zone_pool时检测内存是否正确
    void             *addr;
} ngx_slab_pool_t;
```

![](https://raw.githubusercontent.com/BaihlUp/Figurebed/master/2023/20230411111607.png)

下边是一个页的结构ngx_slab_page_t
```c
// ngx_slab_page_t
// slab页信息
// 管理每个内存页
// 只有三个指针大小，64位系统上是3*8=24字节
struct ngx_slab_page_s {
    // 有多种含义：
    // 指示连续空闲页的数量,NGX_SLAB_PAGE
    // 标记页面的状态：busy
    // 位图方式标记页面内部的使用情况
    uintptr_t         slab;

    // 后链表指针，串联多个可分配内存页
    // 全满页的next是null
    ngx_slab_page_t  *next;

    // 半满页指向管理头节点
    // prev的后两位标记页类型
    // 全满页低位作为页标记
    // ngx_slab_page_prev计算
    uintptr_t         prev;
};
```
如果页链表中有多个连续页空闲，则可以进行合并，合并后页的数量计入slab中，然后修改页的next指针，指向后边的页。

![](https://raw.githubusercontent.com/BaihlUp/Figurebed/master/2023/20230411111950.png)
上边有5个页，其中有连续的页，如上边slab=2，全满页会脱离链表，所以next和prev指针为0。

ngx_slab_max_size指定了最大内存块的大小。
```c
    // 最大slab，是page的一半
    // 超过此大小则直接分配整页
    // ngx_pagesize是4k
    ngx_slab_max_size = ngx_pagesize / 2;
```
分配内存流程：
![](https://raw.githubusercontent.com/BaihlUp/Figurebed/master/2023/20230411112227.png)

> 通过slots数组管理包含相同类型内存块大小的页面，slots数组有序，通过线性偏移，则可以直接找到需要的内存块大小所属页面链表地址，然后在页链表中找能满足的页，如果分配的内存大于了ngx_slab_max_size，则直接分配空闲页，如果小于则看看有没有半满页能满足，在页内部包含多个内存块，通过bitmap管理，标识内存块是否可用。如果bitmap全部可用，则表示当前页为全满页，则加入全满页链表。