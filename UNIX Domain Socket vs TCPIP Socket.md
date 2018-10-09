### 1.区别

> A UNIX socket is an inter-process communication mechanism that allows bidirectional data exchange between processes running on the same machine.

> IP sockets (especially TCP/IP sockets) are a mechanism allowing communication between processes over the network. 
In some cases, you can use TCP/IP sockets to talk with processes running on the same computer (by using the loopback interface).

> UNIX domain sockets know that they’re executing on the same system, so they can avoid some checks and operations (like routing); 
which makes them faster and lighter than IP sockets. 
So if you plan to communicate with processes on the same host, this is a better option than IP sockets.

 - UNIX socket是一种IPC机制，是基于本地的可靠的通信机制，基于本地文件系统（跟socketpair有点类似）
 - IP socket，是基于网络，需要经过网络协议栈，网卡什么的
 - socket API监听unix套接字，用文件代替TCP/IP socket。
