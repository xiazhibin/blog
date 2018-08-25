### key空转时间
```python
int_value = redis.object('idletime', key)
```

返回的是一个`int`值，表达这个键多久没有被访问。可以考虑用`scan`删除空转时间比较大的值，剔除.

```python
host = 'vs-005.localdomain'

client = StrictRedis(host=host)
cursor, data = client.scan(cursor=0)
for key in data:
    print client.object('idletime', key)
while cursor:
    cursor, data = client.scan(cursor=cursor)
    for key in data:
        print client.object('idletime', key)
```

### redis eventLoop
```python
def main():
    inti_server()
    while server_is_not_shut_down():
        aeProcessEvents()
    clean_server()

def aeProcessEvents():
    time_event = aeSearchNearestTimer()
    
    remaind_ms = time_event.ms - now()
    
    if remain_ms <0:
        remain_ms = 0
    timeval = create_timeval_with_ms(remain_ms)
    aeApiPoll(timeval)
    processFileEvents()
    processTimeEvents()

def processTimeEvents():
    for time_event in all_time_events():
        if time_event.when <= now():
            retval = time_event.timeProc()
            if retvel == AE_NOMORE: ## 如果是定期事件
                del_time_event_from_server(time_event)
            else: ## 周期性事件
                update_when(time_event,retval)
```

正常情况下redis只有一个周期时间`serverCron`，更新服务器统计信息，清理过期的键值对，关闭清理失效的客户端，尝试进行AOF或者RDB持久化操作。
如果是主服务器，进行定期同步。如果是集群模式，进行定期同步和链接测试。


### AOF和RDB
- RDB 压缩的二进制文件
  - 可以save和bgsave
  - redis-server版本不同，数据之间不兼容

- AOF(append of file)文本文件
  - 体积较大，但是不存在版本兼容问题
  - aof会重写，来减少自己的体积，减少冗余。aof重写，不会读旧的aof文件，会遍历redis的所有key
  
### redis pipeline
[pipeline](https://github.com/xiazhibin/blog/blob/master/redis%E7%AC%94%E8%AE%B0.md)
