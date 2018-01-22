- [Linux进程基础](http://www.cnblogs.com/vamei/archive/2012/09/20/2694466.html)
- [Linux进程关系](http://www.cnblogs.com/vamei/archive/2012/10/07/2713023.html)
- [CREATING A DAEMON THE PYTHON WAY ](http://code.activestate.com/recipes/278731/)
- [守护进程详解](https://www.cnblogs.com/mickole/p/3188321.html)
- [How do I get my program to act like a daemon](https://www.svbug.com/documentation/comp.unix.programmer-FAQ/faq_2.html#SEC16)

```python
# coding=utf-8
import os
import sys

WORKDIR = '/'
REDIRECT_TO = getattr(os, 'devnull', '/dev/null')


def create_daemon():
    if os.fork():  # 避免挂起控制终端将守护进程放入后台执行。让父进程终止， 让守护进行在子进程中后台执行。
        os._exit(0)

    os.setsid()  # 摆脱控制终端、登录会话和进程组（从父进程继承下来），使之不受它们的影响。使子进程成为新的回话组成。

    # 此时，子进程已经成为无终端的会话组长，但它可以重新申请打开一个控制终端
    if os.fork():
        os._exit(0)  # 结束子进程,孙进程不再是会话组长了

    # 关闭文件继承的文件描述符

    # os.chdir(WORKDIR) #改变当前工作目录

    # 重设文件创建掩码
    os.umask(0o22)

    # 重定向标准输入，输出，错物流
    fd_null = os.open(REDIRECT_TO, os.O_RDWR)
    if fd_null != 0:
        os.dup2(fd_null, 0)

    os.dup2(fd_null, 1)
    os.dup2(fd_null, 2)

    return 0


if __name__ == "__main__":
    retCode = create_daemon()
    procParams = """return code = %s
process ID = %s
parent process ID = %s
process group ID = %s
session ID = %s
user ID = %s
effective user ID = %s
real group ID = %s
effective group ID = %s
   """ % (retCode, os.getpid(), os.getppid(), os.getpgrp(), os.getsid(0),
          os.getuid(), os.geteuid(), os.getgid(), os.getegid())

    open("createDaemon.log", "w").write(procParams + "\n")

    sys.exit(retCode)

```
