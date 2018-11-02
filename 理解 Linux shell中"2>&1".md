### 前言
我们经常可以看到这样的

`./test.sh  > log.txt 2>&1`

我们也知道他的作用是将标准输出和标准错误都重定向到`log.txt`那里

### 举个例子
假如我们有如下的`test.sh`
```bash
#!/bin/bash
date         #打印当前时间
while true   #死循环
do
    sleep 2     #每隔2秒打印一次
    whatthis    #不存在的命令
    echo -e "std output"
done
```

然后执行`./test.sh > log.txt`

我们可以看到这样的输出:

```bash
$ ./test.sh >log.txt
./test.sh: line 7: whatthis: command not found
```

如果执行`./test.sh  > log.txt 2>&1`，那么错误也会重定向到`log.txt`

### 文件描述符
我们执行`nohup ./test.sh  > log.txt 2>&1 &`,可以看到返回的pid

cd到目录`cd /proc/pid/fd`， 执行`ls -al`：

```bash
l-wx------ 1 deploy deploy 64 Nov  2 16:52 0 -> /dev/null
l-wx------ 1 deploy deploy 64 Nov  2 16:52 1 -> /tmp/log.txt
l-wx------ 1 deploy deploy 64 Nov  2 16:52 2 -> /tmp/log.txt
lr-x------ 1 deploy deploy 64 Nov  2 16:52 255 -> /tmp/test.sh
```

我们可以看到这个pid所有打开的文件描述符。PS: 255是bash的一个trick。

### 最后
执行 `./test.sh 2>&1 >log.txt ` 也是可以的
