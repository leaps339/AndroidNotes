# Socket 通信原理

## Socket是什么

Socket是应用层与TCP/IP协议族通信的中间软件抽象层，它是一组接口。在设计模式中，Socket其实就是一个门面模式，把复杂的TCP/IP协议族隐藏在Socket接口后面，对用户来说，一组简单的接口就是全部，让Socket去组织数据，以符合指定的协议。

![socket抽象层](/计算机网络/img/socket抽象层.png)

>Socket起源于UNIX，在UNIX一切皆文件的思想下，进程间通信就被冠名为**文件描述符（file descriptor）**，Socket是一种“打开--读/写--关闭”模式的实现，服务器和客户端各自维护一个“文件”，在建立连接打开后，可以向文件写入内容供对方读取或者读取对方内容，通讯结束时关闭文件。

## Socket通信

Socket保证了不同计算机之间的通信，也就是网络通信。应用层使用Socket接口进行进程间的通信，一般我们将主动接收消息的一段称为服务器端，主动发送消息的一端称为客户端。服务端先初始化Socket，然后与端口绑定（bind），对端口进行监听（listen），调用accept阻塞，等待客户端连接。客户端那边也初始化一个socket，然后通过连接服务器（connect）

![socket通信过程](/计算机网络/img/socket通信过程.png)

客户端过程：创建Socket，连接服务器，将Socket与远程主机连接（只有 TCP 才有“连接”的概念，一些 Socket 比如 UDP、ICMP 和 ARP 没有“连接”的概念），发送数据，读取响应数据。至数据交换完毕，关闭连接，结束TCP对话。

```java
//创建Socket连接
Socket socket = new Socket();
//连接服务器
socket.connect(new InetSocketAddress("localhost", 8088));
//向服务端输出数据
PrintStream writer = new PrintStream(new BufferedOutputStream(socket.getOutputStream()));
BufferedReader reader = new BufferedReader(new InputStreamReader(socket.getInputStream()));
writer.println("hello world");
writer.flush();
//接受到服务端返回的信息
System.out.print("客户端接收到：" + reader.readLine());

writer.close();
reader.close();
socket.close();
```

服务端过程：初始化Socket，建立流式套接字，与本机地址与端口进行绑定，然后通知TCP，准备好接受连接，调用accept阻塞，等待客户端连接。当客户端和服务端建立了连接后，客户端发送数据请求，服务端接受数据请求并处理，然后将响应数据发送给客户端，客户端读取数据，至数据交换完毕。关闭连接，交互结束。

```java
//创建Socket连接
ServerSocket socket = new ServerSocket();
//绑定IP地址与端口号
socket.bind(new InetSocketAddress("localhost", 8088));
//阻塞，等待客户端访问
Socket conn = socket.accept();
//读取客户端请求
BufferedReader reader = new BufferedReader(new InputStreamReader(conn.getInputStream()));
System.out.println("服务端接收到：" + reader.readLine());
//响应数据返回给客户端
PrintStream writer = new PrintStream(new BufferedOutputStream(conn.getOutputStream()));
writer.println("you are so cute");
writer.flush();
//关闭连接
writer.close();
reader.close();
socket.close();
```

## 从TCP连接视角看Socket过程

Socket客户端与服务端进行连接与释放链接需要经过三次握手和四次挥手，接下来分析下基于TCP的三次握手建立连接、四次挥手断开连接。

在[网络分层](/%E8%AE%A1%E7%AE%97%E6%9C%BA%E7%BD%91%E7%BB%9C/%E7%BD%91%E7%BB%9C%E5%88%86%E5%B1%82.md)一节中，说到TCP/IP五层分层模型，其中在数据链路层的数据叫Frame，在网络层的数据叫Packet，在传输层的数据叫Segment。当我们进行数据传输时，我们程序的数据首先会打到TCP的Segment中，然后TCP的Segment回答到IP的Packet中，然后再打到以太网Ethernet的Frame中，传到对端后，各个层解析自己的协议，然后把数据交给更高层的协议处理。而TCP协议相关的字段，全部都在报文段首部。

### TCP报文段首部

TCP是面向字节流的，但传送的数据单元却是报文段（Segment），包含首部和数据部分。数据部分主要放置真正需要传输的原始数据，而报文段首部，包含的各个字段的作用代表着其全部功能。TCP报文段首部前20个字节是固定的，后面有4N字节是根据需要而增加的。如果把TCP报文首部放大来看的话：

![tcp报文段首部结构](/计算机网络/img/tcp报文段首部结构.png)

![tcp报文段首部结构（2）](/计算机网络/img/tcp报文段首部结构（2）.png)

需要注意的是，TCP的包是没有IP地址的，那是IP层上的事。但是有源端口和目标端口。

**源端口、目的端口**

各占2个字段，共四个字节。
用来告知主机该报文段是来自哪里以及传送给哪个应用程序（应用程序绑定了端口）。进行TCP通讯时，客户端通常使用系统自动选择的临时端口号，而服务器则使用指定服务器端口。

**序号 Sequence Number**

占4个字节
TCP是面向字节流的，在一个TCP连接中传输的字节流中每个字节都按照顺序编号。
比如100kb的HTML文档数据，一共102400（100*1024）个字节，那么每一个字节都有编号，整个文档的编号是0~102399.

序号字段值指的是**本报文段**所发送数据的第一个字节的序号。100kb的HTML文档分割为四个等分之后，第一个TCP报文段包含的是第一个25kb的数据，0~25599字节，该报文的序号值就是：0.第二个TCP报文段包含的是第二个25kb的数据，25600~51199字节，该报文的序号值就是：25600。后面依次类推。

根据8位=1字节，那么4个字节可以表示的数值范围：[0，2^32]，一共2^32（4294967296）个序号。序号增加到最大值时，下一个序号又回到了0.也就是说TCP协议可对4GB的数据进行编号，在一般情况下可保证当序号重复使用时，旧序号的数据已经通过网络到达终点或者丢失了。

>Sequence Number主要用来解决网络包乱序（reordering）问题

**确认号 Acknowledgement Number**

占4个字节
表示**期望收到对方下一个报文段的序号值**
TCP的可靠性，是建立在【每一个数据报文都需要确认收到】的基础之上的。就是说，通讯的任何一方在收到对方的一个报文之后，都要发送一个相对应的【确认报文】，来表示确认收到。确认报文，就包含确认号。

比如，通讯一方收到第一个25kb的报文，该报文序号值=0，那么就需要回复一个**确认报文**，其中确认号=25600，代表前0~25599的数据都已经被收到了，这种也叫做**累计确认机制**。

>Acknowledgement Number--就是ACK，用于确认收到，解决不丢包的问题

**数据偏移 Offset**

占0.4个字节
这个字段实际指出了**TCP报文段的首部长度**，它指出了TCP报文段的数据起始处距离TCP报文的起始处有多远。

一个数据偏移量=4 byte，由于4位二进制数能表示的最大十进制数是15，因此数据偏移的最大值是60byte，这也侧面限制了TCP首部的最大长度。

**保留 Reserved**

占0.75个字节（6位）
保留为季候使用，但目前应置为0.

**标志位 TCP Flags**

标志位，分别占1位，共6位。每一位的值只有0和1，分别表达不同的意思，代表包的类型，主要用于操纵TCP的状态机。

- UGE：urgent紧急。当UGE=1时，表示紧急指针（Urgent Pointer）有效。它告诉系统此报文段中有紧急数据，应尽快传送，而不要按原来的排队顺序传送。URG要与首部中的**紧急指针**字段配合使用。
- ACK：acknowledgement确认。当ACK=1时，确认号（Acknowledgment Number）有效。一般称携带ACK标志的TCP报文段为【确认报文段】。TCP规定，在连接建立后所有传送的报文段都必须把ACK设置为1.
- PSH：push传送。当PSH=1时，表示该报文段高优先级，接收方TCP应尽快推送给接收应用程序，而不用等到**整个TCP缓存都填满了后再交付。**
- RST：reset重置。当RST=1时，表示TCP连接中出现严重错误，需要释放并重新建立连接。一般称携带RST标志的TCP报文段为【复位报文段】。
- SYN：synchronous同步，建立联机。当SYN=1时，表明这是一个请求连接报文段。一般称携带SYN标志的TCP报文段为【同步报文段】。在TCP三次握手的第一个报文就是同步报文段，在连接建立时用来同步序号。对方若同意建立连接，则应在响应的报文段中使SYN=1和ACK=1.
- FIN：finish结束。当FIN=1时，表示此报文段发送方的数据已经发送完毕，并要求释放TCP连接。一般称携带FIN报文段为【结束报文段】。在TCP四次挥手释放连接的时候，就会用到该标志。

**窗口大小 Window Size**

占2字节。
该字段明确指出了现在允许对方发送的数据量，它告诉对方本端的TCP接收缓冲区还能容纳多少字节的数据，这样对方就可以控制发送数据的速度。

窗口大小是指，从本报文段首部中的确认号算起，接收方目前允许对方发送的数据量。假如确认号是701，窗口字段是1000.这就表明从701号算起，发送此报文段的一方的缓存空间还能接收1000（字节序号是701~1700）个字节数据。

>又叫Advertised-Window，也就是著名的滑动窗口（Sliding Window），用于解决流量控制。

**校验和 TCP Checksum**

占2个字节
由发送端填充，接收端对TCP报文段执行CRC算法，以检验TCP报文段在传输过程中是否损坏，如果损坏则丢弃。校验范围包括首部和数据两部分，也是TCP可靠性传输的重要保障。

>循环冗余校验（Cyclic Redundancy Check， CRC）是一种根据网络数据包或计算机文件等数据产生简短固定位数校验码的一种信道编码技术，主要用来检测或校验数据传输或者保存后可能出现的错误。它是利用除法及余数的原理来作错误侦测的。

**紧急指针 Urgent Pointer**

占2个字节
仅在URG=1时才有意义，它指出本报文段中紧急数据的字节数。
当URG=1时，发送方TCP就把紧急数据插入法哦本报文段数据的最前面，而在紧急数据后面的数据仍是普通数据。因此，紧急指针指出了紧急数据的末尾在报文段中的位置。

### TCP的三次握手与四次挥手

![tcp握手、挥手流程](/计算机网络/img/tcp_open_close.jpg)

所谓的三次握手（Three-way Handshake），是指建立一个TCP连接时，需要客户端和服务器总共发送3个包即交互三次。三次握手的目的是确认双方的活动状态，并且初始化Sequence Number的初始值。通信双方要互相通知对方自己的初始化的Sequene Number（缩写为ISN：initial Sequence Number）--所以叫SYN，全称Synchronize Sequence Numbers。也就是上图中的x和y。这个号要作为以后数据通信的序号，以保证应用层接收到的数据不会因为网络上的传输问题而乱序（TCP会用这个序号拼接数据）。



## 参考

- [Socket 通信原理](https://segmentfault.com/a/1190000013712747)
- [Socket 通讯原理](https://www.cnblogs.com/zhouxiangting/p/10651599.html)
- [TCP 的那些事儿（上）](https://coolshell.cn/articles/11564.html)
- [理解 TCP（二）：报文结构](https://www.jianshu.com/p/421dd948a42a)