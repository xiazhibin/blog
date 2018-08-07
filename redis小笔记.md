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
