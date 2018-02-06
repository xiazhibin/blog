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

#### 终止
- 调用`close(active close)`，发送一个`FIN`分节
- 接受这个`FIN`执行被动关闭(passvie close)，传递一个`EOF`给该端的应用进程
- 过一会儿，该端接收并处理这个`EOF`调用`close`关闭，导致也发送一个`FIN`
- [示例图](https://github.com/xiazhibin/blog/blob/master/pic/close_tcp.jpg)
