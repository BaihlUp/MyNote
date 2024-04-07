---
title: VPP 详解
date: 2024-03-16
categories:
  - 虚拟化&云计算
tags:
  - DPDK
  - VPP
published: false
---
# 0 参考资料


# 1 VPP 安装和使用
## 1.1 简介
VPP:（vector packet processor）是一个可扩展框架，可提供开箱即用的交换机/路由器功能。是Linux基金会下开源项目FD.io的一个子项目，由思科贡献的开源版本，目前是FD.io的最核心的项目。
VPP是一个模块化和可扩展的软件框架，用于创建网络数据面应用程序。更重要的是，VPP代码为现代通用处理器平台(x86、ARM、PowerPC等)而生，并把重点放在优化软件和硬件接口上，以便用于实时的网络输入输出操作和报文处理。
为了提高性能，vpp数据平面是由转发节点的有向图组成，这些节点在每次调用时处理多个数据包。这种模式支持各种微处理器优化:流水线处理和预取功能降低依赖数据的读取延迟，固有的I-cache阶段行为，向量指令。除了硬件输入和硬件输出节点，整个转发图都是可移植的代码。模块化设计框架允许任何人“插入”新的图形节点，而不需要更改核心/内核代码。
VPP一次从网卡的硬件队列Rx ring收到多个数据包，组成一个Packet vector，借助于报文处理图Packet processing graph来实现数据面转发业务处理流程。图中graph node把业务流程分解为一个个先后连接的业务node，每个业务node完成特定功能后流转到下个node处理。基于这种graph node的组织方式，使我们可以根据业务需求，通过plugin方式插入新的node节点或重新排列graph node，扩展非常方便，不会影响原有核心处理流程。
![](https://raw.githubusercontent.com/BaihlUp/Figurebed/master/2024/20240316171044.png)

## 1.2 VPP 分层模型
1. VPP Infra （ VPP infrastructure layer 基础结构层）

提供一些基本的通用的功能函数库：包括内存管理，向量操作，hash， timer、pool、bimap等

2. Vlib （vector processing library）

主要提供基本的应用管理库：buffer管理，graph node管理，线程，CLI，trace等

3. Vnet （vpp network stack）

提供网络资源能力：比如设备，L2,L3,L4功能，session管理，控制管理，流量管理等。

4. Plugins

主要为实现一些功能，在程序启动的时候加载，一般情况下会在插件中加入一些node节点去实现相关功能，比如qos、nat等。 

## 1.3 VPP 的安装

- 源码下载
```bash
# git clone https://github.com/FDio/vpp.git
# cd vpp
```
> 如果下载失败，可以换成 ssh 方式

- 源码编译
```bash
./extras/vagrant/build.sh
```
会下载很多依赖，时间有点久
- 也可以分步执行：
```bash
##分步执行的命令  
make install-dep
make install-ext-deps

##编译release版本  
make build-release

##编译debug版本
make build
##build deb包
make pkg-deb
```
- 最后安装所有的deb包
```bash
dpkg -i build-root/*.deb
```
到此为止，编译工作全部完成，可以用`systemctl status vpp`看下运行的状态，也可以执行命令手动启动`/usr/bin/vpp -c /etc/vpp/startup.conf`。

## 1.4 VPP 的使用
如果VPP 要基于 DPDK 运行，需要先让 DPDK 绑定网卡，可以通过如下命令查看 网卡绑定信息：
```bash
root@baihl-ubuntu:/home/workspace/vpp# ./extras/vpp_config/scripts/dpdk-devbind.py -s

Network devices using DPDK-compatible driver
============================================
<none>

Network devices using kernel driver
===================================
0000:00:05.0 'Virtio network device' if=enp0s5 drv=virtio-pci unused=
0000:00:06.0 'Virtio network device' if= drv=uio_pci_generic unused=

Other network devices
=====================
<none>
```
目前网卡都被内核接管，需要绑定到 DPDK。
```bash
root@baihl-ubuntu:/home/workspace/vpp# ./extras/vpp_config/scripts/dpdk-devbind.py --bind vfio-pci 0000:0b:00.0
```

修改`/etc/vpp/startup.conf`，将要加载进 vpp的 网卡的 pci添加进去：
```conf
dpdk {
      dev 0000:13:00.0
      dev 0000:0b:00.0
}
```
配置完成后，重启 VPP：`/usr/bin/vpp -c /etc/vpp/startup.conf`

# 2 VPP 的启动流程

## 2.1 VPP 初始化流程
### 2.1.1 加载 plugin（插件）

通过遍历插件目录，加载 插件 so 文件：
```bash
vlib_unix_main --> vlib_plugin_early_init --> vlib_load_new_plugins --> load_one_plugin --> dlopen/dlsym
```

### 2.1.2 注册 node
开发一个 node，需要通过如下方式进行注册到框架中：
```c
VLIB_REGISTER_NODE (tcp4_established_node) = {
  .name = "tcp4-established",
  /* Takes a vector of packets. */
  .vector_size = sizeof (u32),
  .n_errors = TCP_N_ERROR,
  .error_counters = tcp_input_error_counters,
  .format_trace = format_tcp_rx_trace_short,
};
```
通过使用 `VLIB_REGISTER_NODE` 宏把添加的 node 加入注册链表中：
```c
#ifndef CLIB_MARCH_VARIANT
#define VLIB_REGISTER_NODE(x, ...)                                            \
  __VA_ARGS__ vlib_node_registration_t x;                                     \
  static void __vlib_add_node_registration_##x (void)                         \
    __attribute__ ((__constructor__));                                        \
  static void __vlib_add_node_registration_##x (void)                         \
  {                                                                           \
    vlib_global_main_t *vgm = vlib_get_global_main ();                        \
    x.next_registration = vgm->node_registrations;                            \
    vgm->node_registrations = &x;                                             \
  }                                                                           \
  static void __vlib_rm_node_registration_##x (void)                          \
    __attribute__ ((__destructor__));                                         \
  static void __vlib_rm_node_registration_##x (void)                          \
  {                                                                           \
    vlib_global_main_t *vgm = vlib_get_global_main ();                        \
    VLIB_REMOVE_FROM_LINKED_LIST (vgm->node_registrations, &x,                \
				  next_registration);                         \
  }                                                                           \
  __VA_ARGS__ vlib_node_registration_t x
#else
#define VLIB_REGISTER_NODE(x,...)                                       \
STATIC_ASSERT (sizeof(# __VA_ARGS__) != 7,"node " #x " must not be declared as static"); \
static __clib_unused vlib_node_registration_t __clib_unused_##x
#endif
```

在 VPP 启动时，通过如下函数调用链 调用 `vlib_register_all_static_nodes` 进行node 的注册：
```bash
clib_calljmp --> thread0 --> vlib_main --> vlib_register_all_static_nodes --> vlib_register_node
```

> VPP 中使用 calljmp 实现协程调用

可以 看到 `vlib_register_all_static_nodes` 函数中先注册了一个 `null-node` 作为结尾标识，通过 遍历  `vgm->node_registrations` 执行注册逻辑：
```c
void
vlib_register_all_static_nodes (vlib_main_t * vm)
{
  vlib_global_main_t *vgm = vlib_get_global_main ();
  vlib_node_registration_t *r;

  static char *null_node_error_strings[] = {
    "blackholed packets",
  };

  static vlib_node_registration_t null_node_reg = {
    .function = null_node_fn,
    .vector_size = sizeof (u32),
    .n_errors = 1,
    .error_strings = null_node_error_strings,
  };

  /* make sure that node index 0 is not used by
     real node */
  vlib_register_node (vm, &null_node_reg, "null-node");

  r = vgm->node_registrations;
  while (r)
    {
      vlib_register_node (vm, r, "%s", r->name);
      r = r->next_registration;
    }
}
```
### 2.1.3 初始化 init（function init）
	1. cmd init
	2. feature init
	3. node init

function 通过下边的宏初始化：
```c
VLIB_INIT_FUNCTION (ip4_lookup_init);
```

使用同样的方式，把所有的 init 函数以链表的方式连接：
```c
#ifndef CLIB_MARCH_VARIANT
#define VLIB_DECLARE_INIT_FUNCTION(x, tag)                                    \
  vlib_init_function_t *_VLIB_INIT_FUNCTION_SYMBOL (x, tag) = x;              \
  static void __vlib_add_##tag##_function_##x (void)                          \
    __attribute__ ((__constructor__));                                        \
  static _vlib_init_function_list_elt_t _vlib_init_function_##tag##_##x;      \
  static void __vlib_add_##tag##_function_##x (void)                          \
  {                                                                           \
    vlib_global_main_t *vgm = vlib_get_global_main ();                        \
    _vlib_init_function_##tag##_##x.next_init_function =                      \
      vgm->tag##_function_registrations;                                      \
    vgm->tag##_function_registrations = &_vlib_init_function_##tag##_##x;     \
    _vlib_init_function_##tag##_##x.f = &x;                                   \
    _vlib_init_function_##tag##_##x.name = #x;                                \
  }                                                                           \
  static void __vlib_rm_##tag##_function_##x (void)                           \
    __attribute__ ((__destructor__));                                         \
  static void __vlib_rm_##tag##_function_##x (void)                           \
  {                                                                           \
    vlib_global_main_t *vgm = vlib_get_global_main ();                        \
    _vlib_init_function_list_elt_t *this, *prev;                              \
    this = vgm->tag##_function_registrations;                                 \
    if (this == 0)                                                            \
      return;                                                                 \
    if (this->f == &x)                                                        \
      {                                                                       \
	vgm->tag##_function_registrations = this->next_init_function;         \
	return;                                                               \
      }                                                                       \
    prev = this;                                                              \
    this = this->next_init_function;                                          \
    while (this)                                                              \
      {                                                                       \
	if (this->f == &x)                                                    \
	  {                                                                   \
	    prev->next_init_function = this->next_init_function;              \
	    return;                                                           \
	  }                                                                   \
	prev = this;                                                          \
	this = this->next_init_function;                                      \
      }                                                                       \
  }                                                                           \
  static _vlib_init_function_list_elt_t _vlib_init_function_##tag##_##x
#else
/* create unused pointer to silence compiler warnings and get whole
   function optimized out */
#define VLIB_DECLARE_INIT_FUNCTION(x, tag)                      \
static __clib_unused void * __clib_unused_##tag##_##x = x
#endif

#define VLIB_INIT_FUNCTION(x) VLIB_DECLARE_INIT_FUNCTION(x,init)
```

调动 init function：`vlib_call_all_init_functions --> vlib_call_init_exit_functions --> call_init_exit_functions_internal`
```c
static inline clib_error_t *
call_init_exit_functions_internal (vlib_main_t *vm,
				   _vlib_init_function_list_elt_t **headp,
				   int call_once, int do_sort, int is_global)
{
  vlib_global_main_t *vgm = vlib_get_global_main ();
  clib_error_t *error = 0;
  _vlib_init_function_list_elt_t *i;

  if (do_sort && (error = vlib_sort_init_exit_functions (headp)))
    return (error);

  i = *headp;
  while (i)
    {
      uword *h;

      if (is_global)
	h = hash_get (vgm->init_functions_called, i->f);
      else
	h = hash_get (vm->worker_init_functions_called, i->f);

      if (call_once && !h)
	{
	  if (call_once)
	    {
	      if (is_global)
		hash_set1 (vgm->init_functions_called, i->f);
	      else
		hash_set1 (vm->worker_init_functions_called, i->f);
	    }
	  // 执行 init 函数，并获取返回值
	  error = i->f (vm);
	  if (error)
	    return error;
	}
      i = i->next_init_function;
    }
  return error;
}
```

- feature init

feature 定义为 有类似功能 的node。
```c
VNET_FEATURE_INIT (ip4_vxlan_bypass, static) =
{
  .arc_name = "ip4-unicast",
  .node_name = "ip4-vxlan-bypass",
  .runs_before = VNET_FEATURES ("ip4-lookup"),
};
```

VNET_FEATURE_INIT 宏定义如下：
```c
#define VNET_FEATURE_INIT(x,...)				\
  __VA_ARGS__ vnet_feature_registration_t vnet_feat_##x;	\
static void __vnet_add_feature_registration_##x (void)		\
  __attribute__((__constructor__)) ;				\
static void __vnet_add_feature_registration_##x (void)		\
{								\
  vnet_feature_main_t * fm = &feature_main;			\
  vnet_feat_##x.next = fm->next_feature;			\
  fm->next_feature = & vnet_feat_##x;				\
}								\
static void __vnet_rm_feature_registration_##x (void)		\
  __attribute__((__destructor__)) ;				\
static void __vnet_rm_feature_registration_##x (void)		\
{								\
  vnet_feature_main_t * fm = &feature_main;			\
  vnet_feature_registration_t *r = &vnet_feat_##x;		\
  VLIB_REMOVE_FROM_LINKED_LIST (fm->next_feature, r, next);	\
}								\
__VA_ARGS__ vnet_feature_registration_t vnet_feat_##x
```
会把所有待初始化的 feature 组成一个链表。
### 2.1.4 数据包抓取初始化


## 2.2 




# 3 VPP 性能测试

## 3.1 VPP + iperf3

## 3.2 VPP + ACL + Nginx

