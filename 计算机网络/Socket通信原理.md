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

## 参考

- [Socket 通信原理](https://segmentfault.com/a/1190000013712747)
- [Socket 通讯原理](https://www.cnblogs.com/zhouxiangting/p/10651599.html)