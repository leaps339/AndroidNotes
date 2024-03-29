# HTTP发展史

## 目录

- [HTTP发展史](#http发展史)
  - [目录](#目录)
  - [概述](#概述)
  - [万维网](#万维网)
  - [HTTP/0.9 单行协议](#http09-单行协议)
    - [总结](#总结)
  - [ HTTP/1.0 构建可扩展性](#-http10-构建可扩展性)
    - [ 弊端](#-弊端)
    - [ 总结](#-总结)
  - [HTTP/1.1 标准化的协议](#http11-标准化的协议)
    - [ 弊端](#-弊端-1)
    - [ 总结](#-总结-1)
  - [ SPDY：HTTP1.x的优化（改进版HTTP/1.1）](#-spdyhttp1x的优化改进版http11)
  - [HTTP/2 为了更优异的表现](#http2-为了更优异的表现)
    - [二进制分帧层](#二进制分帧层)
    - [弊端](#弊端)
  - [HTTP/3 基于QUIC的HTTP](#http3-基于quic的http)
  - [ 各版本对比](#-各版本对比)
  - [参考](#参考)

## <span id="head1">概述

HTTP(HyperText Transfer Protocol)是万维网（World Wide Web）的基础协议。自Tim Berners-Lee博士和他的团队在1989年-1991年创造出它以来，HTTP已经发生了太多的变化，**在保持协议简单性的同时，不断扩展其灵活性。** 如今，HTTP已经从一个只在实验室之间交换文件的早期协议进化到了可以传输图片、高分辨率视频和3D效果的现代复杂互联网协议。

## <span id="head2">万维网

1898年，Tim Berners-Lee博士写了一份关于建立一个通过网络传输超文本系统的报告。这个系统起初被命名为Mesh，在随后的1990项目实施期间被更名为万维网（World Wide Web）。他在现有的TCP和IP协议基础上建立，由四个部分组成：

* 一个用来表示超文本文档的文本格式，超文本标记语言（HTML）
* 一个用来交换超文本文档的简单协议，超文本传输协议（HTTP）
* 一个显示（以及编辑）超文本文档的客户端，即网络浏览器。第一个网络浏览器被称为WorldWideWeb
* 一个服务器用于提供可访问的文档，即httpd的前身

>httpd：Apache超文本传输协议服务器的主程序。它是一个独立的后台进程，能够处理请求的子进程和线程

这四个部分完成于 1990 年底，且第一批服务器已经在 1991 年初在 CERN 以外的地方运行了。1991 年 8 月 16 日，Tim Berners-Lee 在公开的超文本新闻组上发表的文章被视为是万维网公共项目的开始。

HTTP 在应用的早期阶段非常简单，后来被称为 HTTP/0.9，有时也叫做单行（one-line）协议。

## <span id="head3">HTTP/0.9 单行协议

最初的HTTP协议并没有版本号，后来它的版本号被定为在0.9，以区分后来的版本。HTTP/0.9极其简单：请求由单行指令构成，以唯一可用方法```GET```开头，其后跟目标资源的路径。

```http
GET /mypage.html
```

响应也非常简单，值包含相应文档本身

```html
<HTML>
这是一个非常简单的HTML界面
</HTML>
```

> 最初的HTTP响应内容不包含HTTP头，因此默认只有HTML类的文件可以传送，其他类型的文件无法传送。也没有状态码或错误码，一单出现问题，一个特殊的包含问题描述信息的HTML文件将被发回，供人们查看。

### <span id="head4">总结

* 只有一个GET请求方式
* 没有HEADER、状态码、版本号等额外配置信息
* 只能传输HTML文件
* 服务端响应之后，立刻关闭TCP连接

## <span id="head5"> HTTP/1.0 构建可扩展性

由于HTTP/0.9协议的应用十分有限，浏览器和服务器迅速扩展内容使其用途更广，希望通过HTTP来传输脚本、样式、图片、音视频等不同类型的文件：

* 丰富请求方法：增加POST、HEAD、POST等新方法
* HEADER：引入HTTP标头概念，无论是请求还是响应，允许传输元数据，使协议变得灵活，更具扩展性
* 引入协议版本号概念信息
* 增加响应状态码，使浏览器能了解请求执行成功或失败，并相应调整行为（1xx、2xx、3xx、4xx、5xx）
* 在新 HTTP 标头的帮助下，具备了传输除纯文本 HTML 文件以外其他类型文档的能力（凭借 Content-Type 标头）。

```http
#请求报文
GET /myimage.gif HTTP/1.0
User-Agent: NCSA_Mosaic/2.0 (Windows 3.1)

#响应报文
200 OK
Date: Tue, 15 Nov 1994 08:12:32 GMT
Server: CERN/3.0 libwww/2.17
Content-Type: text/gif
#(这里是图片内容)
```

### <span id="head6"> 弊端

* 连接复用性差：每个TCP连接只能发送一个请求，数据发送完毕连接就关闭，如果还需求请求其他数据，需要再次建立TCP连接。由于TCP的三次握手和四次挥手，导致建立连接成本很高，基于拥塞控制开始时发送速率较慢，所以性能并不比理想
* 又规定在前一个请求响应到达之后下个一请求才能发送，所以阻塞成本也比较大。（**队头阻塞**）

### <span id="head7"> 总结

* 丰富请求方法
* 引入header，增加扩展性
* 引入响应码与响应信息
* 扩展了传输内容的格式
* 引入协议版本号
* 弊端：TCP连接不可复用，队头阻塞问题

>在 1991-1995 年，这些新扩展并没有被引入到标准中以促进协助工作，而仅仅作为一种尝试。服务器和浏览器添加这些新扩展功能，但出现了大量的互操作问题。直到 1996 年 11 月，为了解决这些问题，一份新文档（RFC 1945）被发表出来，用以描述如何操作实践这些新扩展功能。文档 [RFC 1945](https://datatracker.ietf.org/doc/html/rfc1945) 定义了 HTTP/1.0，但它是狭义的，并不是官方标准。

## <span id="head8">HTTP/1.1 标准化的协议

>HTTP/1.0 多种不同的实现方式在实际运用中显得有些混乱。自 1995 年开始，即 HTTP/1.0 文档发布的下一年，就开始修订 HTTP 的第一个标准化版本。在 1997 年初，HTTP1.1 标准发布，就在 HTTP/1.0 发布的几个月后。

* 增加PUT、DELETE、TRACE、OPTIONS等请求方法
* 增加Host字段，用来指定服务器的域名。可以使不同的域名配置在同一个IP地址的服务器上，提高机器复用率
* 新增Connection字段，可以设置keep-alive值保持连接不断开，即TCP连接默认不关闭，可以被多个请求复用。
* 引入管道机制（pipelining），管道化可以不等第一个请求响应继续发送后面的请求，但响应的顺序还是按照请求的顺序返回。即同一个TCP连接中，客户端可以同时发送多个请求。
* 允许响应数据分块（chunked），利于传输大文件
* 引入额外的缓存控制机制，新增一些缓存的字段（If-Modified-Since，If-None-Match）
* 引入内容协商机制，包括语言、编码、类型等。并允许客户端和服务器之间约定以最合适的内容进行交换。（Accept、Accept-Language、Accept-Encoding）

一个典型的请求流程，所有请求都通过一个连接实现：

```http
GET /en-US/docs/Glossary/Simple_header HTTP/1.1
Host: developer.mozilla.org
User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10.9; rv:50.0) Gecko/20100101 Firefox/50.0
<!-- 内容协商机制 -->
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate, br
<!-- 表明请求的来源 -->
Referer: https://developer.mozilla.org/en-US/docs/Glossary/Simple_header

200 OK
Connection: Keep-Alive
Content-Encoding: gzip
Content-Type: text/html; charset=utf-8
Date: Wed, 20 Jul 2016 10:55:30 GMT
Etag: "547fa7e369ef56031dd3bff2ace9fc0832eb251a"
Keep-Alive: timeout=5, max=1000
Last-Modified: Tue, 19 Jul 2016 00:59:33 GMT
Server: Apache
<!-- 分块传输 -->
Transfer-Encoding: chunked
Vary: Cookie, Accept-Encoding

(content)


GET /static/img/header-background.png HTTP/1.1
Host: developer.mozilla.org
User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10.9; rv:50.0) Gecko/20100101 Firefox/50.0
Accept: */*
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate, br
Referer: https://developer.mozilla.org/en-US/docs/Glossary/Simple_header

200 OK
Age: 9578461
Cache-Control: public, max-age=315360000
Connection: keep-alive
Content-Length: 3077
Content-Type: image/png
Date: Thu, 31 Mar 2016 13:34:46 GMT
Last-Modified: Wed, 21 Oct 2015 18:27:50 GMT
Server: Apache

(image content of 3077 bytes)

```

HTTP/1.1在1997年1月以[RFC 2068](https://datatracker.ietf.org/doc/html/rfc2068)文件发布

### <span id="head9"> 弊端

* pipelining只部分解决了队头阻塞（HOLB）。HTTP1.1尝试使用pipelining来解决**队头阻塞**问题，即客户端可以一次性发出多个请求（同个域名，同一条TCP链接）。但pipelining要求返回是按序的，那么前一个请求如果很耗时，那么后面的请求即使服务器已经处理完，仍然会等待前面的请求处理完才会按序返回。
* 协议开销大，没有响应的传输优化方案，HTTP1.1在使用时，header里携带的内容过多，在一定程度上增加了传输的成本，并且每次请求header基本不怎么变化。
* 虽然加入了keep-alive可以复用一部分连接，但**域名分片**等情况下仍需建立多个connect，耗费资源，给服务器带来性能压力
  * 因为**队头阻塞**的存在，那么还是需要多个TCP连接进行并发请求；而为了减轻服务器的压力，限制了同一个域名下和连接数，一般为6~8个。为了解决数量限制，出现了**域名分片**的技术，其实就是资源分域，将资源放在不同域名下（比如二级子域名下），这样就可以针对不同的域名创建连接并请求，以一种讨巧的方式突破限制。
  * 但是滥用此技术也会造成很多问题，比如每个TCP连接本身需要经过DNS查询、三次握手、慢启动等，还占用额外的CPU和内存，对于服务器来说过多的连接也容易造成网络拥挤、交通阻塞等。

### <span id="head10"> 总结

* 增加PUT、DELETE、OPTIONS等请求方法
* 增加长连接，可被多个请求复用；并增加管道化技术，允许在第一个应答被完全发送前就发送第二个请求，降低通信延迟
* 允许响应数据分块传输
* 引入额外的缓存控制机制
* 引入内容协商机制，包括语言、编码、类型等。
* 新增Host标头，明确标明请求域名

## <span id="head11"> SPDY：HTTP1.x的优化（改进版HTTP/1.1）

这些年来，网页愈渐变得的复杂，甚至演变成了独有的应用，可见媒体的播放量，增进交互的脚本大小也增加了许多：更多的数据通过 HTTP 请求被传输。HTTP/1.1 链接需要请求以正确的顺序发送，理论上可以用一些并行的链接（尤其是 5 到 8 个），但是带来的成本和复杂性堪忧。比如，HTTP 管道化（pipelining）就成为了 Web 开发的负担。为此，在 2010 年早期，谷歌通过实践了一个实验性的 SPDY 协议。

>SPDY是Google开发的基于TCP的会话层协议，用以最小化网络延迟，提升网络速度，优化用户的网络使用体验。SPDY并不是一种用于替代HTTP的协议，而是对HTTP协议的增强。
新协议的功能包括数据流的多路复用、请求优先级以及HTTP报头压缩。谷歌表示引入SPDY协议后，在实验室测试中页面加载速度比原先快64%。

* **降低延迟**：针对HTTP高延迟的问题，SPDY采用了多路复用（multiplexing）。多路复用通过多个**stream**共享一个tcp连接的方式，解决了HOL blocking（队头阻塞）问题，降低了延迟同时提高了宽带的利用率
* **请求优先级**（request prioritization）：多路复用带来一个新的问题是，在连接共享的基础之上有可能会导致关键请求被阻塞。SPDY允许给每个request设置优先级，这样重要的请求就会优先得到响应。
* **header压缩**：前面提到HTTP1.x的header很多时候都是重复多余的。选择合适的压缩算法可以减小包的大小和数量。
* **基于HTTPS的加密协议传输**：大大提高了传输数据的可靠性。
* **服务端推送**：可以让服务端主动把资源文件推送给客户端，当然客户端也有权利选择是否接受。![网络分层架构](https://blog-1256965811.cos.ap-guangzhou.myqcloud.com/img/SPDY.webp)

## <span id="head12">HTTP/2 为了更优异的表现

SPDY协议得到Chrome、Firefox等大型浏览器的支持，在一些大型网站和小型网站种部署，这个高效的协议引起了HTTP工作组的注意，在此基础上制定了官方Http2.0标准。

HTTP/2 在 HTTP/1.1 有几处基本的不同：

* **双向数据流**：客户端和服务器把HTTP消息分解为互不依赖的帧，然后乱序发送，最后再在另一端把它们重新组合起来。注意，同一链接上有多个不同方向的数据流在传输。客户端可以一边乱序发送stream，也可以一边接收者服务器的响应，而服务器那端同理。
  * 可以并行交错地发送请求,请求之间互不影响
  * 可以并行交错地发送响应,响应之间互不干扰
  * 只使用一个连接即可并行发送多个请求和响应
  * 消除不必要的延迟,从而减少页面加载的时间
![二进制分帧层](https://blog-1256965811.cos.ap-guangzhou.myqcloud.com/img/%E5%8F%8C%E5%90%91%E5%AD%97%E8%8A%82%E6%B5%81.jpg.crdownload)
* **多路复用**：废弃了HTTP/1.1的管道，同一个TCP连接里面，客户端和服务器可以同时发送多个请求和多个响应，并且不用按照顺序来。由于服务器不用按顺序来处理响应，所以避免了**队头阻塞**的问题。
* **头部信息压缩**：在客户端和服务器端使用“首部表”来跟踪和存储之前发送的键-值对，对于相同的数据，不再通过每次请求和响应发送;通信期间几乎不会改变的通用键-值对(用户代理、可接受的媒体类型,等等)只需发送一次。事实上,如果请求中不包含首部(例如对同一资源的轮询请求),那么 首部开销就是零字节。此时所有首部都自动使用之前请求发送的首部。
  * 如果首部发生变化了，那么只需要发送变化了数据在Headers帧里面，新增或修改的首部帧会被追加到“首部表”。首部表在 HTTP 2.0 的连接存续期内始终存在,由客户端和服务器共同渐进地更新 。
* **服务端主动推送**：服务器可以对一个客户端请求发送多个响应。换句话说，除了对最初请求的响应外，服务器还可以额外向客户端推送资源，而无需客户端明确地请求。

### <span id="head13">二进制分帧层

HTTP 2.0最大的特点： 不会改动HTTP 的语义，HTTP 方法、状态码、URI 及首部字段，等等这些核心概念上一如往常，却能致力于突破上一代标准的性能限制，改进传输性能，实现低延迟和高吞吐量。而之所以叫2.0，是在于新增的二进制分帧层。

既然又要保证HTTP的各种动词，方法，首部都不受影响，那就需要在应用层(HTTP2.0)和传输层(TCP or UDP)之间增加一个二进制分帧层。

在二进制分帧层上，HTTP 2.0 会将所有传输的信息分割为更小的消息和帧,并对它们采用二进制格式的编码 ，其中HTTP1.x的首部信息会被封装到Headers帧，而我们的request body则封装到Data帧里面。

![二进制分帧层](https://blog-1256965811.cos.ap-guangzhou.myqcloud.com/img/binary_framing_layer.webp)

然后，HTTP 2.0 通信都在一个连接上完成，这个连接可以承载任意数量的双向数据流。相应地，每个数据流以消息的形式发送，而消息由一或多个帧组成，这些帧可以乱序发送，然后再根据每个帧首部的流标识符重新组装。

**整个二进制分帧层包含四个概念**：

* **链接Link**：指一条C/S之间的TCP链接，整个通信链路的基础
* **数据流Stream**：已建立的TCP连接内的双向字节流，TCP连接中可以承载一条或多条消息
* **消息Message**：消息属于一个数据流，消息就是逻辑请求或响应消息对应的完整的一系列帧，帧组成了消息
* **帧Frame**：帧是最小的通信单位，每个帧都包含帧头和消息体，标识当前帧所属的数据流。

![二进制分帧层格式简图](https://blog-1256965811.cos.ap-guangzhou.myqcloud.com/img/binary_framing_process.webp)

### <span id="head14">弊端

HTTP/2虽然解决了许多问题，但在TCP协议级别上仍然存在类似的队头阻塞问题，而TCP仍是Web构建的基础。当TCP数据包在传输过程丢失时，在服务器重新发送丢失的数据包之前，接受方无法确认传入的数据包。由于TCP在设计上不遵循HTTP之类的高级协议，因此单个丢失的数据包将阻塞所有进行中的HTTP请求的流，知道重新发送丢失的数据为止。

## <span id="head15">HTTP/3 基于QUIC的HTTP

HTTP/2 由于采用二进制分帧进行多路复用，通常只使用一个 TCP 连接进行传输，在 TCP 层处理的数据包丢失检测和重传可以阻止所有流。

这种问题已经不能通过优化来解决了，因为这是TCP协议本身具有的特性。包括三次握手、四次挥手，拥塞控制等，因为它是面向连接、可靠的传输层协议。

因此，谷歌选择了UDP去开发新一代的HTTP协议-QUIC（Quick UDP Internet Connections）协议。一个具备TCP协议有点的新协议。

首先看一下TCP协议的不足和UDP的一些优点：

* 基于TCP开发的设备和协议非常多，兼容困难
* TCP协议栈是Linux内部的重要部分，修改和升级成本很大
* UDP本身是无连接的、没有建链和拆链成本
* UDP的数据包无队头阻塞问题
* UDP改造成本小

幸运的是UDP是足够简单的，并且同TCP一样等到广泛支持，可以作为在其之上运行的自定义协议的基础。

与 HTTP2 在技术上允许未加密的通信不同，QUIC 严格要求加密后才能建立连接。此外，加密不仅适用于 HTTP 负载，还适用于流经连接的所有数据，从而避免了一大堆安全问题。建立持久连接、协商加密协议，甚至发送第一批数据都被合并到 QUIC 中的单个请求/响应周期中，从而大大减少了连接等待时间。如果客户端具有本地缓存的密码参数，则可以通过简化的握手重新建立与已知主机的连接。

为了解决传输级别的线头阻塞问题，通过 QUIC 连接传输的数据被分为一些流。流是持久性 QUIC 连接中短暂、独立的“子连接”。每个流都处理自己的数据包丢失检测和重传，但使用连接全局压缩和加密属性。每个客户端发起的 HTTP 请求都在单独的流上运行，因此丢失数据包不会影响其他流/请求的数据传输。

## <span id="head16"> 各版本对比

| 协议版本 | 解决的核心问题 | 解决方式|
| --- | --- | --- |
| 0.9 | HTML文件传输 | 确立了客户端请求、服务端响应的通信流程|
| 1.0 | 不同类型的文件传输| 设计头部字段，增加C/S端自定义扩展性|
| 1.1 | TCP连接建链、拆链开销大| 建立长连接进行复用|
| 2  |统一域名下TCP链接数量有限，队头阻塞又不得不启用多条TCP通道| 二进制分帧，请求和响应彻底无序进行| 
| 3 | TCP丢包重传策略，依然会造成队头阻塞| 在UDP的基础上进行改造，继承TCP协议的优点 |

## 参考

* [HTTP的发展](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Basics_of_HTTP/Evolution_of_HTTP)
* [终于搞懂了，HTTP协议发展简史](https://zhuanlan.zhihu.com/p/472698466)
* [HTTP发展史，HTTP1.1与HTTP2.0的区别](https://juejin.cn/post/7079936383925616653#heading-2)
* [HTTP2.0性能增强的核心：二进制分帧](https://blog.csdn.net/u011904605/article/details/53012844)