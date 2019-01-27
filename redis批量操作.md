### 批量插入数据

[mass insert](https://redis.io/topics/mass-insert)

原理跟`MSET`差不多，就是尽量多地执行redis命令，减少 round trip。这里利用`redis-cli`的`pipe`模式

- 利用python脚本生成redis命令(redis_commands.txt)。例如：`set name0 helloworld`
- 使用bash脚本将redis命令转化为redis协议(redis_data.txt)。例如
```
*3
$3
set
$5
name0
$10
helloworld
```

- 最后 `cat redis_data.txt | redis-cli --pipe`
