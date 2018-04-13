## TCP
- [报文格式](https://github.com/xiazhibin/blog/blob/master/pic/TCP%E6%8A%A5%E6%96%87%E6%A0%BC%E5%BC%8F.jpg)
- 提供数据的可靠递送
- RTT算法，客户端和服务器的往返时间
- 对分包的数据进行排序
- 流量控制

### TCP建立和终止

#### 建立：三路握手
- 服务器调用`socket`，`bind`，`listen` `被动打开(passive open)`接受外来的连接
- 客户端调用`connect`，`主动打开{active open)` 客户端发送一个`SYN`(不携带数据)
- 服务器发送`ACK`该客户端`SYN`，同时也发送一个`SYN`给客户端
- 客户端发送`ACK`服务的`SYN`
- [示例图](https://github.com/xiazhibin/blog/blob/master/pic/three_way_handshake.jpg)


#### 终止：四次挥手
- 调用`close(active close)`，发送一个`FIN`分节
- 接受这个`FIN`执行被动关闭(passvie close)，传递一个`EOF`给该端的应用进程
- 过一会儿，该端接收并处理这个`EOF`调用`close`关闭，导致也发送一个`FIN`
- [示例图](https://github.com/xiazhibin/blog/blob/master/pic/close_tcp.jpg)

#### 三次握手原因
为了防止已失效的连接请求报文段突然又传送到了服务端，因而产生错误。client发出的第一个连接请求报文段并没有丢失，而是在某个网络结点长时间的滞留了，以致延误到连接释放以后的某个时间才到达server。本来这是一个早已失效的报文段。但server收到此失效的连接请求报文段后，就误认为是client再次发出的一个新的连接请求。于是就向client发出确认报文段，同意建立连接。假设不采用“三次握手”，那么只要server发出确认，新的连接就建立了。由于现在client并没有发出建立连接的请求，因此不会理睬server的确认，也不会向server发送数据。但server却以为新的运输连接已经建立，并一直等待client发来数据。这样，server的很多资源就白白浪费掉了。
**这么做就会被别人攻击了，消耗server的backlog，client端发送SYN就不管了**

#### 四次挥手
三次挥手也是可行的，Server端将FIN和ACK一起发送。

#### TIME_WAIT
TCP主动关闭连接的那一方会最后进入TIME_WAIT。没有收到对端发送的FIN前，发送自己的FIN。

主动关闭方需要进入TIME_WAIT是为了能够重发丢掉的被动关闭方FIN的ACK。让被动关闭方可以回收资源
