---
title: HTTP2 vs HTTP3详解
date: 2024-02-05
categories:
  - 网络协议
tags:
  - HTTP2
  - HTTP3
published: true
---
# 1 HTTP2 特性概览

## 1.1 头部压缩
在HTTP/1中使用头部字段“Content-Encoding”指定Body的编码方式，比如用gzip压缩来节约带宽，但是报文的头部却被无视，没有针对优化手段。有时候，比如GET请求，Body很小，就几十字节，但头部字段却多达上百字节。

在HTTP/2中使用头部压缩手段对头部进行优化，使用的是“HPACK”算法，在客户端和服务器两端建立“字典”，用索引号表示重复的字符串，还釆用哈夫曼编码来压缩整数和字符串，可以达到50%~90%的高压缩率。

“HPACK”算法是专门为压缩HTTP头部定制的算法，与gzip、zlib等压缩算法不同，它是一个“有状态”的算法，需要客户端和服务器各自维护一份“索引表”，也可以说是“字典”（这有点类似brotli），压缩和解压缩就是查表和更新表的操作。
为了方便管理和压缩，HTTP/2废除了原有的起始行概念，把起始行里面的请求方法、URI、状态码等统一转换成了头字段的形式，并且给这些“不是头字段的头字段”起了个特别的名字——“伪头字段”（pseudo-header fields）。
为了与“真头字段”区分开来，这些“伪头字段”会在名字前加一个“:”，比如“:authority” “:method” “:status”，分别表示的是域名、请求方法和状态码。
HTTP报文头全都是“Key-Value”形式的字段，于是HTTP/2就为一些最常用的头字段定义了一个只读的“静态表”（Static Table）。
- **下列表格列出“静态表“的一部分：**
![](https://raw.githubusercontent.com/BaihlUp/Figurebed/master/2024/20240205111609.png)

- **但如果表里只有Key没有Value，或者是自定义字段根本找不到该怎么办呢？**
这就要用到“动态表”（Dynamic Table），它添加在静态表后面，结构相同，但会在编码解码的时候随时更新。
比如说，第一次发送请求时的“user-agent”字段长是一百多个字节，用哈夫曼压缩编码发送之后，客户端和服务器都更新自己的动态表，添加一个新的索引号“65”。那么下一次发送的时候就不用再重复发那么多字节了，只要用一个字节发送编号就好。
![](https://raw.githubusercontent.com/BaihlUp/Figurebed/master/2024/20240205111703.png)
随着在HTTP/2连接上发送的报文越来越多，两边的“字典”也会越来越丰富，最终每次的头部字段都会变成一两个字节的代码，原来上千字节的头用几十个字节就可以表示了，压缩效果比gzip要好得多。

## 1.2 二进制格式
在HTTP/1中使用的纯文本形式，容易出现多义性，比如大小写、空白字符等，程序处理时使用复杂的状态机，效率低，还麻烦。
而二进制里只有“0”和“1”，可以严格规定字段大小、顺序、标志位等格式，“对就是对，错就是错”，解析起来没有歧义，实现简单，而且体积小、速度快，做到“内部提效”。
HTTP/2的二进制格式参考了TCP协议的部分特性，把原来的“Header+Body”的消息“打散”为数个小片的二进制“帧”（Frame），用“HEADERS”帧存放头数据、“DATA”帧存放实体数据。HTTP/2数据分帧后“Header+Body”的报文结构就完全消失了，协议看到的只是一个个的“碎片”。
![](https://raw.githubusercontent.com/BaihlUp/Figurebed/master/2024/20240205110823.png)
头部数据压缩之后，HTTP/2就要把报文拆成二进制的帧准备发送。HTTP/2的帧结构有点类似TCP的段或者TLS里的记录，但报头很小，只有9字节。二进制的格式也保证了不会有歧义，而且使用位运算能够非常简单高效地解析。
![](https://raw.githubusercontent.com/BaihlUp/Figurebed/master/2024/20240205111845.png)
1. 帧开头是3个字节的长度（但不包括头的9个字节），默认上限是2^14，最大是2^24，也就是说HTTP/2的帧通常不超过16K，最大是16M。
2. **帧类型**：大致可以分成数据帧和控制帧两类，HEADERS帧和DATA帧属于数据帧，存放的是HTTP报文，而SETTINGS、PING、PRIORITY等则是用来管理流的控制帧。
3. **帧标志**：可以保存8个标志位，携带简单的控制信息。常用的标志位有END_HEADERS表示头数据结束，相当于HTTP/1里头后的空行（“\r\n”），END_STREAM表示单方向数据发送结束（即EOS，End of Stream），相当于HTTP/1里Chunked分块结束标志（“0\r\n\r\n”）。
4. **流标识符**：也就是帧所属的“流”，接收方使用它就可以从乱序的帧里识别出具有相同流ID的帧序列，按顺序组装起来就实现了虚拟的“流”。流标识符虽然有4个字节，但最高位被保留不用，所以只有31位可以使用，也就是说，流标识符的上限是2^31，大约是21亿。

**下面给一个帧的实例：**
![](https://raw.githubusercontent.com/BaihlUp/Figurebed/master/2024/20240205111926.png)
在这个帧里，开头的三个字节是“00010a”，表示数据长度是266字节。
帧类型是1，表示HEADERS帧，负载（payload）里面存放的是被HPACK算法压缩的头部信息。
标志位是0x25，转换成二进制有3个位被置1。PRIORITY表示设置了流的优先级，END_HEADERS表示这一个帧就是完整的头数据，END_STREAM表示单方向数据发送结束，后续再不会有数据帧（即请求报文完毕，不会再有DATA帧/Body数据）。
最后4个字节的流标识符是整数1，表示这是客户端发起的第一个流，后面的响应数据帧也会是这个ID，也就是说在stream[1]里完成这个请求响应。

## 1.3 虚拟的“流”

为了把二进制的消息在目的地更好的组装起来，HTTP2 定义了“流”（stream）的概念，它是二进制帧的双向传输序列，同一个消息往返的帧会被分配一个唯一的流 ID，可以想象它是一个虚拟的“数据流”，在里面流动的是一个串有先后顺序的数据帧，这些帧按照次序组装起来就是HTTP/1里的请求报文和响应报文。
因为“流”是虚拟的，实际上并不存在，所以HTTP/2就可以在一个TCP连接上用“流”同时发送多个“碎片化”的消息，这就是常说的“多路复用”（ Multiplexing）——多个往返通信都复用一个连接来处理。
在“流”的层面上看，消息是一些有序的“帧”序列，而在“连接”的层面上看，消息却是乱序收发的“帧”。多个请求/响应之间没有了顺序关系，不需要排队等待，也就不会再出现“队头阻塞”问题，降低了延迟，大幅度提高了连接的利用率。
![](https://raw.githubusercontent.com/BaihlUp/Figurebed/master/2024/20240205111235.png)

为了更好地利用连接，加大吞吐量，HTTP/2还添加了一些控制帧来管理虚拟的“流”，实现了优先级和流量控制，这些特性也和TCP协议非常相似。
HTTP/2还在一定程度上改变了传统的“请求-应答”工作模式，服务器不再是完全被动地响应请求，也可以新建“流”主动向客户端发送消息。比如，在浏览器刚请求HTML的时候就提前把可能会用到的JS、CSS文件发给客户端，减少等待的延迟，这被称为“服务器推送”（Server Push，也叫Cache Push）。
HTTP/2使用虚拟的“流”传输消息，解决了困扰多年的“队头阻塞”问题，同时实现了“多路复用”，提高连接的利用率。
下边示例，在Wireshark抓包里，就有“0、1、3”一共三个流，实际上就是分配了三个流ID号，把这些帧按编号分组，再排一下队，就成了流。
![](https://raw.githubusercontent.com/BaihlUp/Figurebed/master/2024/20240205112212.png)
在概念上，一个HTTP/2的流就等同于一个HTTP/1里的“请求-应答”。在HTTP/1里一个“请求-响应”报文来回是一次HTTP通信，在HTTP/2里一个流也承载了相同的功能。
**下边整理下 HTTP/2 的流的特点，如下：**
1. 流是可并发的，一个HTTP/2连接上可以同时发出多个流传输数据，也就是并发多请求，实现“多路复用”；
2. 客户端和服务器都可以创建流，双方互不干扰；
3. 流是双向的，一个流里面客户端和服务器都可以发送或接收数据帧，也就是一个“请求-应答”来回；
4. 流之间没有固定关系，彼此独立，但流内部的帧是有严格顺序的；
5. 流可以设置优先级，让服务器优先处理，比如先传HTML/CSS，后传图片，优化用户体验；
6. 流ID不能重用，只能顺序递增，客户端发起的ID是奇数，服务器端发起的ID是偶数；
7. 在流上发送“RST_STREAM”帧可以随时终止流，取消接收或发送；
8. 第0号流比较特殊，不能关闭，也不能发送数据帧，只能发送控制帧，用于流量控制。

**下边示例展示了无序的帧是如何依据流ID重组成流的：**
![](https://raw.githubusercontent.com/BaihlUp/Figurebed/master/2024/20240205112516.png)

**根据以上 HTTP/2的 特性，可以整理出以下结论：**
1. 最开始的时候流都是“空闲”（idle）状态，也就是“不存在”，可以理解成是待分配的“号段资源”。
2. 当客户端发送HEADERS帧后，有了流ID，流就进入了“打开”状态，两端都可以收发数据，然后客户端发送一个带“END_STREAM”标志位的帧，流就进入了“半关闭”状态。这个半关闭状态，意味着客户端的请求数据已经发送完了，需要接受响应数据，而服务器端也知道请求数据接收完毕，之后就要内部处理，再发送响应数据。
3. 响应数据发送完了，也带上一个“END_STREAM”标志位，表示数据发送完毕，这样流两端就都进入了“关闭”状态，流就结束了。
4. 流id不能重用，所以流的生命周期就是HTTP/1里的一次完整的“请求-应答”，流关闭就是一次通信结束。下次再发送数据需要开一个新流。

**流的状态切换：**
HTTP/2 流状态转换图如下：
![](https://raw.githubusercontent.com/BaihlUp/Figurebed/master/2024/20240205154257.png)
1. 最开始的时候流都是“空闲”（idle）状态，也就是“不存在”，可以理解是待分配的“号段资源”。
2. 当客户端发送HEADERS帧后，有了流ID，流就进入了“打开”状态，两端都可以收发数据，然后客户端发送一个带“END_STREAM”标志位的帧，流就进入了“半关闭”状态。这个半关闭状态，意味着客户端的请求数据已经发送完了，需要接受响应数据，而服务器端也知道请求数据接收完毕，之后就要内部处理，再发送响应数据。
3. 响应数据发送完了，也带上一个“END_STREAM”标志位，表示数据发送完毕，这样流两端就都进入了“关闭”状态，流就结束了。
4. 流ID不能重用，所以流的生命周期就是HTTP/1里的一次完整的“请求-应答”，流关闭就是一次通信结束。下次再发送数据需要开一个新流。

## 1.4 协议栈
- 协议栈对比

下边的图对比了 HTTP/1、HTTPS和HTTP/2 的协议栈，可以清晰地看到，HTTP/2是建立在“HPack”“Stream”“TLS1.2”基础之上的，比HTTP/1、HTTPS复杂了一些。
![](https://raw.githubusercontent.com/BaihlUp/Figurebed/master/2024/20240205112933.png)

虽然上图中HTTP/2是建立在TLS上的，但HTTP/2其实也支持使用明文传输，不过格式还是二进制只是没有加密。为了区分“加密”和“明文”这两个不同的版本，HTTP/2协议定义了两个字符串标识符：“h2”表示加密的HTTP/2，“h2c”表示明文的HTTP/2，多出的那个字母“c”的意思是“clear text”。

- 连接前言

由于HTTP/2“事实上”是基于TLS，所以在正式收发数据之前，会有TCP握手和TLS握手，TLS握手成功之后，客户端必须要发送一个“连接前言”（connection preface），用来确认建立HTTP/2连接。
这个“连接前言”是标准的HTTP/1请求报文，使用纯文本的ASCII码格式，请求方法是特别注册的一个关键字“PRI”，全文只有24个字节：
```bash
PRI * HTTP/2.0\r\n\r\nSM\r\n\r\n
```
只要服务器收到这个“有魔力的字符串”，就知道客户端在TLS上想要的是HTTP/2协议，而不是其他别的协议，后面就会都使用HTTP/2的数据格式。
抓包看的话，如下：
![](https://raw.githubusercontent.com/BaihlUp/Figurebed/master/2024/20240205113253.png)

# 2 HTTP3 特性概览

## 2.1 HTTP/2 vs HTTP/3

HTTP2协议虽然大幅提升了HTTP/1.1的性能，然而，基于TCP实现的HTTP2遗留下3个问题：

- 有序字节流引出的 队头阻塞（[Head-of-line blocking](https://en.wikipedia.org/wiki/Head-of-line_blocking)），使得HTTP2的多路复用能力大打折扣；
- TCP与TLS叠加了握手时延，建连时长还有1倍的下降空间；
- 基于TCP四元组确定一个连接，这种诞生于有线网络的设计，并不适合移动状态下的无线网络，这意味着IP地址的频繁变动会导致TCP连接、TLS会话反复握手，成本高昂。

HTTP3协议解决了这些问题：

- HTTP3基于UDP协议重新定义了连接，在QUIC层实现了无序、并发字节流的传输，解决了队头阻塞问题（包括基于QPACK解决了动态表的队头阻塞）；
- HTTP3重新定义了TLS协议加密QUIC头部的方式，既提高了网络攻击成本，又降低了建立连接的速度（仅需1个RTT就可以同时完成建连与密钥协商）；
- HTTP3 将Packet、QUIC Frame、HTTP3 Frame分离，实现了连接迁移功能，降低了5G环境下高速移动设备的连接维护成本。

## 2.2 与 HTTP/2 的异同点
HTTP2与HTTP3采用二进制、静态表、动态表与Huffman算法对HTTP Header编码，不只提供了高压缩率，还加快了发送端编码、接收端解码的速度。
HTTP2协议基于TCP有序字节流实现，因此应用层的多路复用并不能做到无序地并发，在丢包场景下会出现队头阻塞问题。如下面的动态图片所示，服务器返回的绿色响应由5个TCP报文组成，而黄色响应由4个TCP报文组成，当第2个黄色报文丢失后，即使客户端接收到完整的5个绿色报文，但TCP层不会允许应用进程的read函数读取到最后5个报文，并发成了一纸空谈。  
当网络繁忙时，丢包概率会很高，多路复用受到了很大限制。因此， HTTP3采用UDP作为传输层协议，重新实现了无序连接，并在此基础上通过有序的QUIC Stream提供了多路复用。

## 2.3 连接迁移
对于当下的HTTP1和HTTP2协议，传输请求前需要先完成耗时1个RTT的TCP三次握手、耗时1个RTT的TLS握手（TLS1.3），由于它们分属内核实现的传输层、openssl库实现的表示层，所以难以合并在一起，如下图所示：
![](https://raw.githubusercontent.com/BaihlUp/Figurebed/master/2024/20240205155517.png)

在IoT时代，移动设备接入的网络会频繁变动，从而导致设备IP地址改变。对于通过四元组（源IP、源端口、目的IP、目的端口）定位连接的TCP协议来说，这意味着连接需要断开重连，所以上述2个RTT的建链时延、TCP慢启动都需要重新来过。而HTTP3的QUIC层实现了连接迁移功能，允许移动设备更换IP地址后，只要仍保有上下文信息（比如连接ID、TLS密钥等），就可以复用原连接。  
在UDP报文头部与HTTP消息之间，共有3层头部，定义连接且实现了Connection Migration主要是在Packet Header中完成的，如下图所示：
![](https://raw.githubusercontent.com/BaihlUp/Figurebed/master/2024/20240205155552.png)

**这3层Header实现的功能各不相同：**
- Packet Header实现了可靠的连接。当UDP报文丢失后，通过Packet Header中的Packet Number实现报文重传。连接也是通过其中的Connection ID字段定义的；
- QUIC Frame Header在无序的Packet报文中，基于QUIC Stream概念实现了有序的字节流，这允许HTTP消息可以像在TCP连接上一样传输；
- HTTP3 Frame Header定义了HTTP Header、Body的格式，以及服务器推送、QPACK编解码流等功能。

为了进一步提升网络传输效率，Packet Header又可以细分为两种：
- Long Packet Header用于首次建立连接；
- Short Packet Header用于日常传输数据。

其中，Long Packet Header的格式如下图所示：
![](https://raw.githubusercontent.com/BaihlUp/Figurebed/master/2024/20240205155706.png)

建立连接时，连接是由服务器通过Source Connection ID字段分配的，这样，后续传输时，双方只需要固定住Destination Connection ID，就可以在客户端IP地址、端口变化后，绕过UDP四元组（与TCP四元组相同），实现连接迁移功能。下图是Short Packet Header头部的格式，这里就不再需要传输Source Connection ID字段了：
![](https://raw.githubusercontent.com/BaihlUp/Figurebed/master/2024/20240205155805.png)

上图中的Packet Number是每个报文独一无二的序号，基于它可以实现丢失报文的精准重发。如果通过抓包观察Packet Header，会发现Packet Number被TLS层加密保护了，这是为了防范各类网络攻击的一种设计。下图给出了Packet Header中被加密保护的字段：
![](https://raw.githubusercontent.com/BaihlUp/Figurebed/master/2024/20240205155845.png)

其中，显示为E（Encrypt）的字段表示被TLS加密过。当然，Packet Header只是描述了最基本的连接信息，其上的Stream层、HTTP消息也是被加密保护的。

## 2.4 Stream 解决队头阻塞问题
解决队头阻塞的方案，就是允许微观上有序发出的Packet报文，在接收端无序到达后也可以应用于并发请求中。
在Packet Header之上的QUIC Frame Header，定义了有序字节流Stream，而且Stream之间可以实现真正的并发。HTTP3的Stream，借鉴了HTTP2中的部分概念，所以在讨论QUIC Frame Header格式之前，可以先看看HTTP2中的Stream长成什么样子：
![](https://raw.githubusercontent.com/BaihlUp/Figurebed/master/2024/20240205160104.png)

每个Stream就像HTTP/1中的TCP连接，它保证了承载的HEADERS frame（存放HTTP Header）、DATA frame（存放HTTP Body）是有序到达的，多个Stream之间可以并行传输。在HTTP3中，上图中的HTTP2 frame会被拆解为两层，先来看底层的QUIC Frame。  
一个Packet报文中可以存放多个QUIC Frame，当然所有Frame的长度之和不能大于PMTUD（Path Maximum Transmission Unit Discovery，这是大于1200字节的值），可以把它与IP路由中的MTU概念对照理解：
![](https://raw.githubusercontent.com/BaihlUp/Figurebed/master/2024/20240205160440.png)

每个Frame都有明确的类型：
![](https://raw.githubusercontent.com/BaihlUp/Figurebed/master/2024/20240205160505.png)
前4个字节的Frame Type字段描述的类型不同，接下来的编码也不相同，下表是各类Frame的16进制Type值：
![](https://raw.githubusercontent.com/BaihlUp/Figurebed/master/2024/20240205160520.png)

只要分析0x08-0x0f这8种STREAM类型的Frame，就能弄明白Stream流的实现原理，自然也就清楚队头阻塞是怎样解决的了。Stream Frame用于传递HTTP消息，它的格式如下所示：
![](https://raw.githubusercontent.com/BaihlUp/Figurebed/master/2024/20240205160823.png)

Stream Frame头部的3个字段，完成了多路复用、有序字节流以及报文段层面的二进制分隔功能，包括：
- Stream ID标识了一个有序字节流。当HTTP Body非常大，需要跨越多个Packet时，只要在每个Stream Frame中含有同样的Stream ID，就可以传输任意长度的消息。多个并发传输的HTTP消息，通过不同的Stream ID加以区别；
- 消息序列化后的“有序”特性，是通过Offset字段完成的，它类似于TCP协议中的Sequence序号，用于实现Stream内多个Frame间的累计确认功能；
- Length指明了Frame数据的长度。

你可能会奇怪，为什么会有8种Stream Frame呢？这是因为0x08-0x0f 这8种类型其实是由3个二进制位组成，它们实现了以下3种 标志位的组合：

- 第1位表示是否含有Offset，当它为0时，表示这是Stream中的起始Frame，这也是上图中Offset是可选字段的原因；
- 第2位表示是否含有Length字段；
- 第3位Fin，表示这是Stream中最后1个Frame，与HTTP2协议Frame帧中的FIN标志位相同。

Stream数据中并不会直接存放HTTP消息，因为HTTP3还需要实现服务器推送、权重优先级设定、流量控制等功能，所以Stream Data中首先存放了HTTP3 Frame：
![](https://raw.githubusercontent.com/BaihlUp/Figurebed/master/2024/20240205161037.png)

其中，Length指明了HTTP消息的长度，而Type字段（请注意，低2位有特殊用途，在下边的QPACK章节中会详细介绍）包含了以下类型：
- 0x00：DATA帧，用于传输HTTP Body包体；
- 0x01：HEADERS帧，通过QPACK 编码，传输HTTP Header头部；
- 0x03：CANCEL_PUSH控制帧，用于取消1次服务器推送消息，通常客户端在收到PUSH_PROMISE帧后，通过它告知服务器不需要这次推送；
- 0x04：SETTINGS控制帧，设置各类通讯参数；
- 0x05：PUSH_PROMISE帧，用于服务器推送HTTP Body前，先将HTTP Header头部发给客户端，流程与HTTP2相似；
- 0x07：GOAWAY控制帧，用于关闭连接（注意，不是关闭Stream）；
- 0x0d：MAX_PUSH_ID，客户端用来限制服务器推送消息数量的控制帧。

总结一下，QUIC Stream Frame定义了有序字节流，且多个Stream间的传输没有时序性要求，这样，HTTP消息基于QUIC Stream就实现了真正的多路复用，队头阻塞问题自然就被解决掉了。

## 2.5 QPACK 编码解决队头阻塞问题
与HTTP2中的HPACK编码方式相似，HTTP3中的QPACK也采用了静态表、动态表及Huffman编码：
![](https://raw.githubusercontent.com/BaihlUp/Figurebed/master/2024/20240205161620.png)

在上图中，GET方法映射为数字2，这是通过客户端、服务器协议实现层的硬编码完成的。在HTTP2中，共有61个静态表项：
![](https://raw.githubusercontent.com/BaihlUp/Figurebed/master/2024/20240205161647.png)

而在QPACK中，则上升为90多个静态表项，比如Nginx上的ngx_http_v3_static_table数组所示：
```c
static ngx_http_v3_field_t  ngx_http_v3_static_table[] = {

    { ngx_string(":authority"),            ngx_string("") },
    { ngx_string(":path"),                 ngx_string("/") },
    { ngx_string("age"),                   ngx_string("0") },
    { ngx_string("content-disposition"),   ngx_string("") },
    { ngx_string("content-length"),        ngx_string("0") },
    { ngx_string("cookie"),                ngx_string("") },
    { ngx_string("date"),                  ngx_string("") },
    { ngx_string("etag"),                  ngx_string("") },
    { ngx_string("if-modified-since"),     ngx_string("") },
    { ngx_string("if-none-match"),         ngx_string("") },
    { ngx_string("last-modified"),         ngx_string("") },
    { ngx_string("link"),                  ngx_string("") },
    { ngx_string("location"),              ngx_string("") },
    { ngx_string("referer"),               ngx_string("") },
    { ngx_string("set-cookie"),            ngx_string("") },
    { ngx_string(":method"),               ngx_string("CONNECT") },
    { ngx_string(":method"),               ngx_string("DELETE") },
    { ngx_string(":method"),               ngx_string("GET") },
    { ngx_string(":method"),               ngx_string("HEAD") },
    { ngx_string(":method"),               ngx_string("OPTIONS") },
    { ngx_string(":method"),               ngx_string("POST") },
    { ngx_string(":method"),               ngx_string("PUT") },
    { ngx_string(":scheme"),               ngx_string("http") },
    { ngx_string(":scheme"),               ngx_string("https") },
    { ngx_string(":status"),               ngx_string("103") },
    { ngx_string(":status"),               ngx_string("200") },
    { ngx_string(":status"),               ngx_string("304") },
    { ngx_string(":status"),               ngx_string("404") },
    { ngx_string(":status"),               ngx_string("503") },
    { ngx_string("accept"),                ngx_string("*/*") },
    { ngx_string("accept"),                ngx_string("application/dns-message") },
    { ngx_string("accept-encoding"),       ngx_string("gzip, deflate, br") },
    { ngx_string("accept-ranges"),         ngx_string("bytes") },
    { ngx_string("access-control-allow-headers"), ngx_string("cache-control") },
    { ngx_string("access-control-allow-headers"), ngx_string("content-type") },
    { ngx_string("access-control-allow-origin"), ngx_string("*") },
    { ngx_string("cache-control"),         ngx_string("max-age=0") },
    { ngx_string("cache-control"),         ngx_string("max-age=2592000") },
    { ngx_string("cache-control"),         ngx_string("max-age=604800") },
    { ngx_string("cache-control"),         ngx_string("no-cache") },
    { ngx_string("cache-control"),         ngx_string("no-store") },
    { ngx_string("cache-control"),         ngx_string("public, max-age=31536000") },
    { ngx_string("content-encoding"),      ngx_string("br") },
    { ngx_string("content-encoding"),      ngx_string("gzip") },
    { ngx_string("content-type"),          ngx_string("application/dns-message") },
    { ngx_string("content-type"),          ngx_string("application/javascript") },
    { ngx_string("content-type"),          ngx_string("application/json") },
    { ngx_string("content-type"),          ngx_string("application/x-www-form-urlencoded") },
    { ngx_string("content-type"),          ngx_string("image/gif") },
    { ngx_string("content-type"),          ngx_string("image/jpeg") },
    { ngx_string("content-type"),          ngx_string("image/png") },
    { ngx_string("content-type"),          ngx_string("text/css") },
    { ngx_string("content-type"),
          ngx_string("text/html;charset=utf-8") },
    { ngx_string("content-type"),          ngx_string("text/plain") },
    { ngx_string("content-type"),
          ngx_string("text/plain;charset=utf-8") },
    { ngx_string("range"),                 ngx_string("bytes=0-") },
    { ngx_string("strict-transport-security"),
                                           ngx_string("max-age=31536000") },
    { ngx_string("strict-transport-security"),
          ngx_string("max-age=31536000;includesubdomains") },
    { ngx_string("strict-transport-security"),
          ngx_string("max-age=31536000;includesubdomains;preload") },
    { ngx_string("vary"),                  ngx_string("accept-encoding") },
    { ngx_string("vary"),                  ngx_string("origin") },
    { ngx_string("x-content-type-options"),
                                           ngx_string("nosniff") },
    { ngx_string("x-xss-protection"),      ngx_string("1;mode=block") },
    { ngx_string(":status"),               ngx_string("100") },
    { ngx_string(":status"),               ngx_string("204") },
    { ngx_string(":status"),               ngx_string("206") },
    { ngx_string(":status"),               ngx_string("302") },
    { ngx_string(":status"),               ngx_string("400") },
    { ngx_string(":status"),               ngx_string("403") },
    { ngx_string(":status"),               ngx_string("421") },
    { ngx_string(":status"),               ngx_string("425") },
    { ngx_string(":status"),               ngx_string("500") },
    { ngx_string("accept-language"),       ngx_string("") },
    { ngx_string("access-control-allow-credentials"),
                                           ngx_string("FALSE") },
    { ngx_string("access-control-allow-credentials"),
                                           ngx_string("TRUE") },
    { ngx_string("access-control-allow-headers"),
                                           ngx_string("*") },
    { ngx_string("access-control-allow-methods"),
                                           ngx_string("get") },
    { ngx_string("access-control-allow-methods"),
                                           ngx_string("get, post, options") },
    { ngx_string("access-control-allow-methods"),
                                           ngx_string("options") },
    { ngx_string("access-control-expose-headers"),
                                           ngx_string("content-length") },
    { ngx_string("access-control-request-headers"),
                                           ngx_string("content-type") },
    { ngx_string("access-control-request-method"), ngx_string("get") },
    { ngx_string("access-control-request-method"), ngx_string("post") },
    { ngx_string("alt-svc"),               ngx_string("clear") },
    { ngx_string("authorization"),         ngx_string("") },
    { ngx_string("content-security-policy"), ngx_string("script-src 'none';object-src 'none';base-uri 'none'") },
    { ngx_string("early-data"),            ngx_string("1") },
    { ngx_string("expect-ct"),             ngx_string("") },
    { ngx_string("forwarded"),             ngx_string("") },
    { ngx_string("if-range"),              ngx_string("") },
    { ngx_string("origin"),                ngx_string("") },
    { ngx_string("purpose"),               ngx_string("prefetch") },
    { ngx_string("server"),                ngx_string("") },
    { ngx_string("timing-allow-origin"),   ngx_string("*") },
    { ngx_string("upgrade-insecure-requests"), ngx_string("1") },
    { ngx_string("user-agent"),            ngx_string("") },
    { ngx_string("x-forwarded-for"),       ngx_string("") },
    { ngx_string("x-frame-options"),       ngx_string("deny") },
    { ngx_string("x-frame-options"),       ngx_string("sameorigin") }
};
```
对于Huffman以及整数的编码，QPACK与HPACK并无多大不同，但动态表编解码方式差距很大。
动态表，就是将未包含在静态表中的Header项，在其首次出现时加入动态表，这样后续传输时仅用1个数字表示，大大提升了编码效率。因此，动态表是天然具备时序性的，如果首次出现的请求出现了丢包，后续请求解码HPACK头部时，一定会被阻塞！

- QPACK是如何解决队头阻塞问题的呢？

QPACK将动态表的编码、解码独立在单向Stream中传输，仅当单向Stream中的动态表编码成功后，接收端才能解码双向Stream上HTTP消息里的动态表索引。
单向指只有一端可以发送消息，双向则指两端都可以发送消息。上一小节的QUIC Stream Frame头部中，其中的Stream ID除了标识Stream外，它的低2位还可以表达以下组合：
![](https://raw.githubusercontent.com/BaihlUp/Figurebed/master/2024/20240205162245.png)

因此，当Stream ID是0、4、8、12时，这就是客户端发起的双向Stream（HTTP3不支持服务器发起双向Stream），它用于传输HTTP请求与响应。单向Stream有很多用途，所以它在数据前又多出一个Stream Type字段：
![](https://raw.githubusercontent.com/BaihlUp/Figurebed/master/2024/20240205162324.png)

Stream Type有以下取值：
- 0x00：控制Stream，传递各类Stream控制消息；
- 0x01：服务器推送消息；
- 0x02：用于编码QPACK动态表，比如面对不属于静态表的HTTP请求头部，客户端可以通过这个Stream发送动态表编码；
- 0x03：用于通知编码端QPACK动态表的更新结果。
由于HTTP3的STREAM之间是乱序传输的，因此，若先发送的编码Stream后到达，双向Stream中的QPACK头部就无法解码，此时传输HTTP消息的双向Stream就会进入Block阻塞状态（两端可以通过控制帧定义阻塞Stream的处理方式）。

## 2.6 常见问题
### 2.6.1 放大攻击
放大攻击利用攻击者和目标Web资源之间的带宽差异消耗，当差异消耗经过多次请求被放大时，产生的流量可能导致网络设施中断。通过发送小型请求来导致大规模响应，达到四四两拨千斤的效果。
最常见的放大攻击是DNS放大攻击，因为DNS使用的是UDP协议。TCP是面向连接的协议，在交互前需要进行三次握手进行建立连接，服务器可以验证客户端IP地址的合法性，UDP协议由于其无连接性，最容易被用来做放大攻击，HTTP3协议是基于UDP的，所以同样面临此问题。

- 解决放大攻击--地址验证功能

QUIC 针对放大攻击的主要防御措施是验证端点（endpoint）是否能够在其声明的传输地址接收数据包。地址验证在连接建立（connection establishment）期间和连接迁移（connection migration）期间进行。
- 连接建立时，为了验证客户端的地址是否是攻击者伪造的，服务端会生成一个令牌（token）并通过重试包（Retry packet）响应给客户端。客户端需要在后续的初始包（Initial packet）带上这个令牌，以便服务端进行地址验证。
- 服务端可以在当前连接中通过 NEW_TOKEN 帧预先发布令牌，以便客户端在后续的新连接使用，这是 QUIC 实现 0-RTT 很重要的一个功能。
- 当我们的网络路径变化时（比如从蜂窝网络切换到 WIFI），QUIC 提供了连接迁移（connection migration）的功能来避免连接中断。QUIC 通过路径验证（Path Validation）验证网络新地址的可达性（reachability），防止在连接迁移中的地址是攻击者伪造的。

### 2.6.2 连接迁移失败
通过上边可知，通过 Connection ID的使用可以在地址变化的情况下实现连接迁移能力，但是在网络中客户端服务端怎么知道是否要开始连接迁移呢，如下图所示：
![](https://raw.githubusercontent.com/BaihlUp/Figurebed/master/2024/20240205165116.png)

对于上边的问题有两种方式：
1. 客户端网络状态发生变化时，主动出发连接迁移
2. 客户端等待时间超出限制时，使用PING帧来探测连接，告知服务器连接迁移的发生

### 2.6.3 连接迁移对LB的影响
因为在现实网络中引入了负载均衡设备，会导致在连接迁移时，所有依赖五元组 Hash 做转发导致关联 Session 的机制失效。以 LVS 为例，连接迁移后， LVS 依靠五元组寻址会导致寻址的服务器存在不一致。
即便 LVS 寻址正确，当报文到达服务器时，如果是多进程的Server(如 Nginx)，内核会根据五元组关联进程，依然会寻址出错。
1. 在四层或者七层负载均衡上，需要重新设计QUIC LoadBalancer 的机制
2. 若是QUIC 服务器多进程工作模式上，也需要进行相应的改造

针对以上问题解决方案思路是：提取出 连接 ID，进行一定的处理。

1. 四层负载均衡的改造

![](https://raw.githubusercontent.com/BaihlUp/Figurebed/master/2024/20240205170500.png)

四层均衡器增加解析 QUIC 建连报文 提取其中的 连接ID，通过把 连接ID与四元组对应起来，实现根据负载均衡。为了避免维护 连接ID 与 四元组的对应关系，可以通过把 连接ID的一部分加入转发路由中，组成转发信息，等请求到达时，根据连接ID 可以直接找到需要转发出去的路由。

2. 七层负载均衡改造

![](https://raw.githubusercontent.com/BaihlUp/Figurebed/master/2024/20240205170830.png)

因为连接ID是在用户态处理，可以使用 ebpf 技术，通过设置 hook 点，解析出连接ID，找到 连接ID 对应的 套接字。
# 3 总结
## 3.1 涉及的RFC文档

基于四元组定义连接并不适用于下一代IoT网络，HTTP3创造出Connection ID概念实现了连接迁移，通过融合传输层、表示层，既缩短了握手时长，也加密了传输层中的绝大部分字段，提升了网络安全性。
HTTP3在Packet层保障了连接的可靠性，在QUIC Frame层实现了有序字节流，在HTTP3 Frame层实现了HTTP语义，这彻底解开了队头阻塞问题，真正实现了应用层的多路复用。
QPACK使用独立的单向Stream分别传输动态表编码、解码信息，这样乱序、并发传输HTTP消息的Stream既不会出现队头阻塞，也能基于时序性大幅压缩HTTP Header的体积。

- 涉及的RFC文档

![](https://raw.githubusercontent.com/BaihlUp/Figurebed/master/2024/20240205162632.png)
## 3.2 HTTP2 和 HTTP3 对比图
![](https://raw.githubusercontent.com/BaihlUp/Figurebed/master/2024/Pasted%20image%2020240205162730.png)
