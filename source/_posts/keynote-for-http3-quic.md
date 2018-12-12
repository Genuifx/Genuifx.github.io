---
title: QUIC 简明教程
date: 2018-11-27 18:51:33
tags: [HTTP3, QUIC, PROTOCOL]
---

**QUIC** (Quick UDP Internet Connections) (发音：quick) 由google开发的新一代网络传输协议。QUIC设计的初衷就是利用工程师几十年的经验来改进网络传输延迟。

> 说出来你可能不信，HTTP的发展被G家握在手里了，HTTP-over-SPDY被改名为HTTP/2，HTTP-over-QUIC被改名为HTTP/3。

QUIC基于UDP协议实现了类似TCP+TLS+HTTP2的功能组合。所以可以把QUIC和现有的协议理解成以下结构：

![](/images/quic/quic-fit.png)
<center>(https://www.nanog.org/sites/default/files//meetings/NANOG64/1051/20150603_Rogan_Quic_Next_Generation_v1.pdf)</center>

## 为什么需要 QUIC？
这个问题也可以理解为，HTTP/2 现在有什么硬伤是不能解决的吗？其实HTTP/2的硬伤就是TCP。

TCP 协议在70年代设计敲定，当时的网络丢包率高，网速差，链接的稳定和可靠对当时来说是最需要解决的事情。对比今天的网络现状，毫无疑问在网络的可靠性和速度上我们都得到了巨大的进步。

经过几十年的经验累积，网络工程师对怎么优化网络访问，降低延迟也有了新的认知，于是就想着更新 TCP 协议，但是 TCP 的更新非常困难，因为网络协议栈的实现本来就依赖系统内核更新，而不管是终端设备，中间设备的系统更新都极其缓慢，一个更新迭代可能需要5-15年的时间去普及，对于现在的互联网发展来说太慢了。 

所以 TCP 的硬伤总结一下就是：**TCP的更新优化需要依赖系统内核更新**。

这是一个日益高速高速发展的应用层和缓慢进化的网络传输层的矛盾，于是[JIM](https://twitter.com/JimRoskind)（QUIC协议设计师）决定在传输层抛弃TCP，拥抱UDP协议。UDP相对TCP协议来说，是一个不稳定的乐观协议，他不需要握手建立连接，至于什么防止IP攻击，怎么保证数据一致性，这些都是UDP之上的工作，这一点刚好契合我们的需求，把很多工作从传输层移动到了应用层（4层网络模型而言~），如此一来，以后QUIC协议的升级完全不依赖于底层操作系统，只需终端和服务器升级到指定版本即可。

这就是**QUIC 的意义**。

## 对比 HTTP/2 的优势所在？

HTTP/2 是个优秀的协议，其最大的特色就是多路复用，那么 QUIC 带了什么新的东西，对比 HTTP/2 有什么优势呢。

主要在以下几点有着巨大的优势：

- 建立连接的延迟
- 改进的拥塞控制
- 多路复用——无队头阻塞版
- 错误自动纠正
- 连接迁移

接下来我们就简洁说明一下为什么会有这些优势😜

### 建立连接的延迟

首先明确一个概念是 RTT（round-trip time），顾名思义，就是服务器和终端一次交互需要的时间。RTT一般用于衡量网络延迟。

传统的TCP协议，我们需要进行3次握手，也就是1.5 RTT，才开始传输数据。

HTTP/2来说，虽然协议上支持不开启TLS，但是目前大家的实现都是绑定了TLS和HTTP，也就是我们不止要建立连接，还要确定好加密版本，加密密钥等信息，TCP+TLS需要3 RTT。

![](/images/quic/tcp-tls.png)
<center>(https://www.nanog.org/sites/default/files//meetings/NANOG64/1051/20150603_Rogan_Quic_Next_Generation_v1.pdf)</center>

对于现阶段的QUIC来说，其设计了自己的加密协议和过程，实现了最好情况下0 RTT的效果。

![](https://res.cloudinary.com/practicaldev/image/fetch/s--0hlAeTAi--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_66%2Cw_880/https://cdn-images-1.medium.com/max/800/1%2Ar6NNOhOGncUfvHXKHUM39w.gif)
<center>(https://dev.to/grigorkh/what-is-http3--4pib)</center>

0 RTT 的效果是因为QUIC的客户端会缓存服务器端发的令牌和证书，当有数据需要再次发送的时候，客户端可以直接使用旧的令牌和证书，这样子就实现了 0 RTT 了。对于没有缓存的情况，服务器端会直接拒绝请求，并且返回新生产的令牌和证书。 所以当令牌失效或者没有缓存的情况下，QUIC还是需要一次握手才能开始传输数据。

当然这么玩还是有代价的，对于重放攻击（replay attack）的防范就需要从应用层解决了。

需要注意的是：**被接纳为HTTP/3之后，QUIC未来还是会采用 TLS 1.3，放弃现有加密算法~**

TLS 1.3 是对于1.2、1.1版本的大跃迁，目前还没有正式版。TLS 1.3 只需要 2 RTT 就完成一次连接建立了。

### 改进的拥塞控制

目前的 QUIC 的拥塞控制主要实现了 TCP 的慢启动，拥塞避免，快重传，快恢复。在这些拥塞控制算法的基础上，再进行改进。

比如单调递增的 Packet Number。TCP 使用了基于字节序号 Sequence Number 和 ACK 来保证消息的有序到达。但是 Sequence Number 在重传的时候有二义性。你不知道下一个 ACK 是上一次请求的响应还是这次重传的响应。而单调递增的 Packet Number 可以避免这个问题，保证采样 RTT 的准确。

> 具体改进可以参考[stgw的文章](https://zhuanlan.zhihu.com/p/32553477)

QUIC 拥塞控制算法主要重新实现了一遍 TCP 的算法，毕竟 TCP 的算法是经过几十年的生产验证的。

### 多路复用——无队头阻塞版

SPDY 和 HTTP/2 已经实现了**多路复用**。多路复用的指的是我们不需要在为每个资源重新建立一次 TCP 连接，多个资源的传输可以共用一个连接。

![](/images/quic/http2-multiplex.png)
<center>(https://www.nanog.org/sites/default/files//meetings/NANOG64/1051/20150603_Rogan_Quic_Next_Generation_v1.pdf)</center>

如此一来，在启用了 HTTP/2 的网站，我们再也不需要对资源进行合并了（：）压缩还是可以做一下的），毕竟一次性发出去多个资源和建立多个连接一个一个下载资源相比还是会快一下的。

然后 HTTP/2 的多路复用会有个很大的问题，那就是**队头阻塞**。原因还是因为 TCP 的 Sequence Number 机制，为了保证资源的有序到达，如果传输队列的队头某个资源丢失了，TCP 必须等到这个资源重传成功之后才会通知应用层处理后续资源。

![](/images/quic/http2-multiplex-head-line-block.png)
<center>(https://www.nanog.org/sites/default/files//meetings/NANOG64/1051/20150603_Rogan_Quic_Next_Generation_v1.pdf)</center>

由于 QUIC 避开了 TCP， 他设计 connection 和 stream 的概念，一个 connection 可以复用传输多个 stream，每个 stream 之间都是独立的，单一一个 stream 丢包并不会影响到其他资源处理。

![](/images/quic/quic-multiplex.png)
<center>(https://www.nanog.org/sites/default/files//meetings/NANOG64/1051/20150603_Rogan_Quic_Next_Generation_v1.pdf)</center>

### 错误自动纠正

这里的错误指的是某个包丢了。当某个 packet 丢失的时候，QUIC 能够通过已经接收到的其他包对资源进行修复。

这意味着，实际上每个 packet 都携带着多余的信息，通过这些信息，QUIC 能够重组对应资源，而无需进行重传。

目前大概每 10 个包能修复一个 packet。

### 连接迁移

TCP 是按照 4-要素（客户端IP、端口, 服务器IP、端口） 要确定一个连接的，当这4个要素其中一个发生变化的时候，连接就需要重新建立。而在移动端，我们经常会切换 4G/wifi 使用，每一次切换，我们只能重新建立连接。

在 QUIC 中，连接是由其维护的。 于是 QUIC 通过生成客户端生成一个 Connection ID （64位）的东西来区别不同连接，只要生成的 UUID 不变， 连接就不需要重新建立，即便是客户端的网络发生变化。

![](/images/quic/quic-header.png)
<center>(https://medium.com/@nirosh/understanding-quic-wire-protocol-d0ff97644de7)</center>

## QUIC 现状

HTTP-over-QUIC 将被吸收改名为 HTTP/3。未来的 web 传输不再依赖 TCP 协议，升级更新也不再需要依赖系统内核升级了，未来的 HTTP 可以跟其他产品一样月更、甚至周更。

目前，如果想体验 QUIC 可以使用 [candy](https://github.com/mholt/caddy/wiki/QUIC) 服务器。candy 在 0.9 版本之后就支持 QUIC 了。

在 github 上面找到了 [C++ 版本](https://github.com/devsisters/libquic)的实现，利用 Nodejs 的 C++ 模块，我们可以快速实现一个 node-quic 的样子。😎

## 参考资料

[ chromium quic blog ](https://docs.google.com/document/d/1gY9-YNDNAB1eip-RTPbqphgySwSNSDHLq9D5Bty4FSU/edit)

[ medium ](https://medium.com/@nirosh/understanding-quic-wire-protocol-d0ff97644de7)

[ Rogan_Quic_Next_Generation ](https://www.nanog.org/sites/default/files//meetings/NANOG64/1051/20150603_Rogan_Quic_Next_Generation_v1.pdf)

[ 知乎专栏 ](https://zhuanlan.zhihu.com/p/32553477)

[ Mattias ](https://ma.ttias.be/googles-quic-protocol-moving-web-tcp-udp/)