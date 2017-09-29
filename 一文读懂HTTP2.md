# 一文读懂 HTTP/2

今天，HTTP 1.1 已经变成互联网中主要的协议，但是在 HTTP 协议诞生初期却是一种简单直接的协议。1996 年在 RFC 1945 中定义了 HTTP 1.0 规范，仅 60 页，到 1999 年在 RFC 2616 定义了 HTTP 1.1，增长到了 176 页。但是，随着 web 技术的飞速发展。 HTTP 1.1 已经无法满足用户对性能的要求，此后 Google 推出协议 SPDY，意在解决 HTTP 1.1 中广为人知的性能问题。SPDY 得到了 Chrome、Firefox 和 Opera 的支持，很多大型网站（如谷歌、Twitter、Facebook）都对兼容客户端使用 SPDY。SPDY 在被行业采用并证明能够大幅提升性能之后，已经具备了成为一个标准的条件。

HTTP/2 是 HTTP 协议自 1999 年 HTTP 1.1 发布后的首个更新，主要基于 SPDY 协议。它由互联网工程任务组（IETF）的Hypertext Transfer Protocol Bis（httpbis）工作小组进行开发。该组织于 2014 年 12 月将 HTTP/2 标准提议递交至 IESG 进行讨论，并于 2015 年 2 月 17 日被批准。HTTP/2 标准于 2015 年 5 月以 RFC 7540 正式发表。

那 HTTP /2 到底有哪些具体变化呢？

### **二进制分帧**

先来理解几个概念：

- 帧：HTTP/2 数据通信的最小单位。
- 消息：指 HTTP/2 中逻辑上的 HTTP 消息。例如请求和响应等，消息由一个或多个帧组成。
- 流：存在于连接中的一个虚拟通道。流可以承载双向消息，每个流都有一个唯一的整数 ID。

HTTP/2 采用二进制格式传输数据，而非 HTTP 1.x 的文本格式，二进制协议解析起来更高效。             

HTTP/1 的请求和响应报文，都是由起始行，首部和实体正文（可选）组成，各部分之间以文本换行符分隔。HTTP/2 将请求和响应数据分割为更小的帧，并且它们采用二进制编码。                                  

HTTP/2 中，同域名下所有通信都在单个连接上完成（在下面“多路复用”中介绍），这个连接可以承载任意数量的双向数据流。每个数据流都以消息的形式发送，而消息又由一个或多个帧组成。多个帧之间可以乱序发送，因为根据帧首部的流标识可以重新组装。

### **多路复用**

多路复用，代替原来的序列和阻塞机制。所有请求都是通过一个 TCP 连接并发完成。

HTTP 1.x 中，如果想并发多个请求，必须使用多个 TCP 链接，且浏览器为了控制资源，还会对单个域名有 6-8 的个数限制，如下图，红色圈出来的请求就因域名链接数已超过限制，而被挂起等待了一段时间： 

![img](https://dn-linuxcn.qbox.me/data/attachment/album/201708/16/105151ulilbqabuoo8soau.png)

在 HTTP/2 中，有了二进制分帧之后，HTTP/2 不再依赖 TCP 链接去实现多流并行了，在 HTTP/2中：

- 同域名下所有通信都在单个连接上完成。
- 单个连接可以承载任意数量的双向数据流。
- 数据流以消息的形式发送，而消息又由一个或多个帧组成，多个帧之间可以乱序发送，因为根据帧首部的流标识可以重新组装。

这一特性，性能会有极大的提升，因为：

- 同个域名只需要占用一个 TCP 连接，消除了因多个 TCP 连接而带来的延时和内存消耗。
- 单个连接上可以并行交错的请求和响应，之间互不干扰。
- 在 HTTP/2 中，每个请求都可以带一个 31bit 的优先值，0 表示最高优先级， 数值越大优先级越低。有了这个优先值，客户端和服务器就可以在处理不同的流时采取不同的策略，以最优的方式发送流、消息和帧。

### **服务器推送**

服务端可以在发送页面 HTML 时主动推送其它资源，而不用等到浏览器解析到相应位置，发起请求再响应。例如服务端可以主动把 JS 和 CSS 文件推送给客户端，而不需要客户端解析 HTML 再发送这些请求。服务端可以主动推送，客户端也有权利选择接收与否。如果服务端推送的资源已经被浏览器缓存过，浏览器可以通过发送 `RST_STREAM` 帧来拒收。主动推送也遵守同源策略，服务器不会随便推送第三方资源给客户端。

### **头部压缩**

HTTP 1.1 请求的大小变得越来越大，有时甚至会大于 TCP 窗口的初始大小，因此它们需要等待带着 ACK 的响应回来以后才能继续被发送。HTTP/2 对消息头采用 HPACK（专为 HTTP/2 头部设计的压缩格式）进行压缩传输，能够节省消息头占用的网络的流量。而 HTTP/1.x 每次请求，都会携带大量冗余头信息，浪费了很多带宽资源。

HTTP 每一次通信都会携带一组头部，用于描述这次通信的的资源、浏览器属性、cookie 等，例如：

![img](https://dn-linuxcn.qbox.me/data/attachment/album/201708/16/105151k8fp8iwiwhz60rrg.png)

为了减少这块的开销并提升性能， HTTP/2 会压缩这些首部：

- HTTP/2 在客户端和服务器端使用“首部表”来跟踪和存储之前发送的键－值对，对于相同的数据，不再通过每次请求和响应发送；
- 首部表在 HTTP/2 的连接存续期内始终存在，由客户端和服务器共同渐进地更新;
- 每个新的首部键－值对要么被追加到当前表的末尾，要么替换表中之前的值。

例如：下图中的两个请求， 请求一发送了所有的头部字段，第二个请求则只需要发送差异数据，这样可以减少冗余数据，降低开销。

![img](https://dn-linuxcn.qbox.me/data/attachment/album/201708/16/105152kbl1w5o05bocw4ys.png)

我们来看一个实际的例子，下面是用 WireShark 抓取的访问 google 首页的包： 

![img](https://dn-linuxcn.qbox.me/data/attachment/album/201708/16/105152jowapeogzqjwq31o.png)

上图是是访问 https://www.google.com/ 抓到的第一个请求的头部，可以看到头部的内容，总共占用了437 字节，我们选中头部的 cookie，可以看到 cookie 总共占用了 118 字节。接下来我们看看第二个请求的头部：

![img](https://dn-linuxcn.qbox.me/data/attachment/album/201708/16/105153apjacpujcvyegpea.png)

从上图可以看到，得益于头部压缩，第二个请求中 cookie 只占用了 1 个字节，我们来看看变化了的 `Accept`字段： 

![img](https://dn-linuxcn.qbox.me/data/attachment/album/201708/16/105154m1du8xf660sysf11.png)

由于 `Accept` 字段与请求一中的内容不同，需要发送给服务器，所以占用了 29 字节。

PS: 这里有个 akamai 的 HTTP /2 的演示链接，有兴趣的可以点击看看： <https://http2.akamai.com/demo> 

![img](https://dn-linuxcn.qbox.me/data/attachment/album/201708/16/105154fqoui3r31azikmkr.jpg)

浏览器和网络服务支持情况：[http2 支持清单](https://github.com/http2/http2-spec/wiki/Implementations)  

### **结语**

本文转载自又拍云，又拍云 CDN 当前已全平台支持 HTTP/2，并已默认开启。又因 HTTP/2 是在 HTTPS 协议的基础上实现的，所以只要使用又拍云 HTTPS 加速服务的域名，都可免费享受 HTTP/2 服务，无需做任何特殊配置。而开启 HTTPS，只需完成证书申请与管理，无须繁杂流程，轻松实现网站与 Web 应用的 HTTPS 加密部署。

### 参考资料

- [Jerry Qu blog](https://imququ.com/) 中的 HTTP/2 专题;
- [维基百科：HTTP/2](https://zh.wikipedia.org/wiki/HTTP/2)
- [RFC 7540 – 超文本传输协议第2版（HTTP / 2）](http://httpwg.org/specs/rfc7540.html)
- [FC 7541 – HPACK：HTTP / 2的头压缩](http://httpwg.org/specs/rfc7541.html)
- [http2 讲解](https://ye11ow.gitbooks.io/http2-explained/content/part2.html)
- <http://www.cnblogs.com/yingsmirk/>