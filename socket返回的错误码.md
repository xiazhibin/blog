### EINTR
- 原因：当阻塞于某个慢系统调用的一个进程捕获某个信号且相应信号处理函数返回时，该系统调用可能返回一个EINTR错误。
例如：在socket服务器端，设置了信号捕获机制，有子进程，
当在父进程阻塞于慢系统调用时由父进程捕获到了一个有效信号时，内核会致使accept返回一个EINTR错误(被中断的系统调用)
- 处理方法：
  - `select`函数:`continue`就好了
  - `connect`函数:因为我们不能再次调用它，否则将立即返回一个错误。所以必须调用select来等待连接完成
 
 ### EADDRINUSE
 - 原因：`new`socket的时候传入的(ip,port)已经被占用了
 - 处理方法：报错
 
 ### EADDRNOTAVAIL
 - 原因：`new`socket的时候传入的(ip,port)非法
 - 处理方式：报错
 
 ### EAGAIN/EWOULDBLOCK
 #### 非阻塞socket
 - `send`函数：当客户通过Socket提供的send函数发送大的数据包时，就可能返回一个EAGAIN的错误。该错误产生的原因是由于send 函数中的size变量大小超过了tcp_sendspace的值。tcp_sendspace定义了应用在调用send之前能够在kernel中缓存的数据量。当应用程序在socket中设置了O_NDELAY或者O_NONBLOCK属性后，如果发送缓存被占满，send就会返回EAGAIN的错误。
   - 处理方法：raise出去
- `recv`函数：`continue`继续`recv`就好了
- `accept`函数：
### 阻塞socket
- 原因：socket设置了`SO_RCVTIMEO`和`SO_SNDTIMEO`会导致`read/write`函数返回`EAGAIN`。另外`O_NODELAY`会导致`write`接口返回E`AGAIN`.如果设置了`O_NODELAY`而当前不可写，那么`write`接口会设置`errno`为`EAGAIN`，但是`write`接口会返回0而不是-1
