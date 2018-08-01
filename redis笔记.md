## redis pipeline
- 优势:是为了提高效率，减少RTT，网络IO消耗
- 原理分析:

```python
client = StrictRedis()

client.set(1, 2)
client.set(2, 3)
##-------##
pipe = client.pipeline()
pipe.set(1, 2)
pipe.set(2, 3)
pipe.execute()
```

以这个为例子，如果直接使用`client`发送两个`set`命令，在`tcp报文`中发会送两次。如果使用`pipeline`之后，`tcp报文`中只会发送一次（将两条命令集合在一起）

- 实现原理

以上两个`set`命令可以重写成这样

```python
with client.pipeline() as pipe:
    pipe.multi()
    pipe.set(1,2)
    pipe.set(2,3)
    pipe.execute()
```

`pipeline`将`StrictRedis`的`execute_command`返回`pipeline_execute_command`,`pipeline_execute_command`中会将所有命令`append`到`command_stack`
中，当`execute`的时候才一起执行。(PS:默认是transaction为true)

意外发现的是，`transaction`其实也是通过这种方式实现，只是增加了一个`pipe.watch(key)`

- 注意事项

  - pipeline命令不能有逻辑先后，即系写命令前需要读命令返回的结果，这个时候不能使用pipeline
  - 每次pipeline的执行命令是有上限的，超过这个值不会有太大的优化，因为pipeline中所有命令和执行结果会被缓存到Redis内存，同时也会造成网络通信变慢，得不偿失


- 参考

[pipelining](http://www.redis.cn/topics/pipelining.html)

[pipelining](https://redis.io/topics/pipelining)
