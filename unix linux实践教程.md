### /etc/passwd是root用户拥有的，但是普通user执行`passwd`的时候该文件会进行修改，是如何做到的呢
- `$ ls - 1 /usr/bin/passwd` `r-sr-xr-x 1 root bin 15725 Oct 31 1997 /usr/bin/passwd` 有个特殊的属性的`set-user-ID（SUID）`。
为某些程序提供额外的权限。

### os.umask(0o22)
- 0o22是八进制，因为一个权限是3位，2的3次方就是8嘛
- `umask`就是拿掉某些位置。一个文件默认权限是`666`, “新建文件掩码”，它指定那些为需要被关掉。
例如防止程序创建能被同组用户和其他用户修改，可以关掉`----w--w-`。然后就变成`644`。

### /etc/passwd
- sshd: x:104:65534::/var/run/sshd:/usr/sbin/nologin
- deploy: x:1000:1000:deploy,,,:/home/deploy:/bin/zsh
- 注册名：口令：用户标识号：组标识号：用户名：用户主目录：命令解释程序

### 文件系统
- `--superblock-----|---inode----|------data block-------` [参考图](https://github.com/xiazhibin/blog/blob/master/pic/pic4_5.jpg)
- superblock: 存放文件系统本身的结构信息：例如每个区域的大小，未被使用的磁盘块信息。
- inode 记录 文件的一些属性，例如size，修改时间，使用了哪些数据块, 链接数等
- data 记录数量，会被划分成为大小一样的块， 每个块有自己的编号
- directory: 一些 `inode | name` pair的目录entry集合
- 例如新建一个文件`a.txt`, `inode`编号为47，使用了33，657，992三个数据块。目录增加`47|a.txt`这样一对的目录entry
- [理解inode](http://www.ruanyifeng.com/blog/2011/12/inode.html)
- `目录包含子目录`意思是父目录有一个`inode`为`277|a` 其中`a`的里面存在以下`inode`:`277|.` `865|..`
- 一个空的目录有两个link，一个是爸爸对儿子的引用，一个是儿子对自己的引用

### fcntl
- 改变已打开的文件性质

### stdin, stdout,stderr3中标准文件描述符
- 大多数进程将这3个文件描述符自动打开

##Shell
### Shell运行流程
- get input -> get command -> exe command -> waitfor command to finish
- fork 子进程，子进程exec，父进程等待子进程退出

### 僵尸进程
- 子进程先于父进程退出，父进程可能会

### execvp(progname,arglist)
- 会替换原来的进程.把当前进程中的机器指令清除，然后在这个进程中载入调用时指定的程序代码

### shell编程
- var=`command` 获得command输出
- $? 获得上一个command退出状态（exit code）
- var=1 赋值
- $var 引用
- unset var 删除
- read var 读入
- set 列出变量
- export var 变成环境变量

### 管道/IO重定向
- 标准输入，输出以及错误输出分别为文件描述符0，1，2
- 内核总是使用最低可用文件描述符
- 文件描述符集合通过exec调用传递，且不会被改变
- 管道沟通两个进程：`ls|head`
- 管道不是文件

### socket
- accept会被信号打断
- `unix:/tmp/test.sock` 其实是一个path`/tmp/test.sock`,`socket.bind(path)`
- unix socket 需要手动删除`os.unlink(path)` [UNIX(7)](http://man7.org/linux/man-pages/man7/unix.7.html)

### Thread有点鸡肋，看任务是选择thread or process还是使用任务队列

### IPC(进程通信)
- 文件
- named pipe
- share memory
