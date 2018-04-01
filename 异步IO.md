### IO模型
linux IO模型一般来说分为五种。(PS:同步异步和阻塞非阻塞是两组正交的概念，一样可以异步阻塞，只是没有人会那么做，太蠢了)。然后我们一般看到的是异步框架，框架
内提供的接口是异步的。但是框架跟系统之间的调用不一定是异步有可能是同步（tornado就是这样子）
- 同步阻塞IO（调用会一直阻塞直到数据接收完毕）
- 同步非阻塞 （设置了O_NONBLOCK,轮询socket是否有数据）
- IO复用（属于同步非阻塞，为了提高效率，不需要轮询socket是否有数据）
- 信号驱动
- 异步非阻塞(IOCP，AIO)
[IO模型](http://www.rowkey.me/blog/2016/01/18/io-model/)
[深入理解 Python 异步编程](http://python.jobbole.com/88291/)
[怎样理解阻塞非阻塞与同步异步的区别？](https://www.zhihu.com/question/19732473/answer/23434554)

### 同步阻塞IO
典型的`一问一答`方式
优点：模式简单，编程方便，也方便理解
缺点：一般采用多线程/多进程来提供并发能力

### 同步非阻塞
`socket.setnoblock(false)`
优点：不会阻塞主线程，进行无数据读写的时候返回`资源暂时无法使用`
缺点：因为要轮询socket的状态，会浪费cpu。轮询的socket数量增多性能下降

### IO复用
`select` `poll` `kqueue` `epoll`等
优点：不需要轮询socket了，相当于交给一个代理去轮询，然后告诉主线程可读写的socket
缺点：编程上难度增加，会引起`回调地狱`
[IO复用回调版](https://gist.github.com/xiazhibin/a47986bebbfbce0fcd36dc32a0019df6)

### IO复用 - 异步版
利用协程的特性，将回调变成顺序执行。关键在于：对于每一个`accepted socket`使用一个处理器进行调度。
[IO复用异步编程](https://gist.github.com/xiazhibin/9b223aa0d1775d7eb1b473a72fcfdee8)

### 后记
以上都是`tornado`进行学习后，参考创作
解决非本地进程之间的通讯，`ip`标识位置,`port`标识应用。
socket是在`应用层`和`传输层`之间的一个抽象层，它把TCP/IP层复杂的操作抽象为几个简单的接口供应用层调用已实现进程在网络中通信。
socket是不知道`掉线`这么一个状态，socket是本地维护`<local_ip,local_port,remote_ip,remote_port>`的状态。直到你使用`socket`进行
操作，不然不会知道对端发生了什么事情。
