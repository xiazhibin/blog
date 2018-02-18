### 在做商品秒杀的时候很容易产生“超发”，“多发”的现象

假设某个抢购场景中，我们一共只有100个商品，在最后一刻，我们已经消耗了99个商品，仅剩最后一个。
这个时候，系统发来多个并发请求，这批请求读取到的商品余量都是1个，然后都通过了这一个余量判断，最终导致超发
[如图](https://github.com/xiazhibin/blog/blob/master/pic/redis_%E8%B6%85%E5%8F%91.jpg)

#### 主要原因是某一些操作并不是原子操作，取余量，判断，更新商品余量

#### 解决思路
- 悲观锁：进行抢购逻辑（取余量，判断，更新余量）前加锁
- FIFO: 将每个请求放进队列,慢慢一个进行处理，将多线程变成单线程那样
- 乐观锁：采用带版本号 （Version）更新。实现就是，这个数据所有请求都有资格去修改，但会获得一个该数据的版本号，只有版本号符合的才能更新成功
（同一个版本号，只有一个能提交成功，并且更新版本号），
其他的返回抢购失败。这样的话，我们就不需要考虑队列的问题，不过，它会增大CPU的计算开销。但是，综合来说，这是一个比较好的解决方案。
- 改变业务模型： 不采用检查余量的方法发放奖品。

#### 采用`watch` 使得取余量，判断，更新余量是一个原子操作
```python
with redis_client.pipeline() as pipe:
    try:
        pipe.watch(key)
        count = int(pipe.get(key))
        if count > 0: 
            pipe.multi()
        pipe.set(key, count - 1)
        pipe.execute()
        do_business()
        return 'success'
    except Exception:
        return 'fail'
```

##### 改变业务模型
预先分配好所有奖品，用redis的`spop` `sadd`处理
```python

#!/usr/bin/env python
# -*- coding: utf-8 -*-
#
# Copyright 2016
# @author: liukelin
#
# 一个固定商品数量的抢购 的伪代码
# 1.避免抢购超出 预定数量
# 2.避免同用户多次抢购
# 3.redis的spop和sadd可以扛得住10w的QPS，毫无压力

import redis

'''
假设该商品有1000份
goods_num = 1000
开抢之前设定好一个1000个元素的 redis set集合 goods_set_key(每个元素为商品id（goods_id）)
 sadd goods_set [1 ，2 ，3 ，4 .. 1000]
uid = 1
'''

# 限制频繁请求
check_act = redis.get('check_act'+uid)
if check_act>1:
    redis.expire('check_act'+uid, 1)
    return '请求过于频繁，请稍后再试'

# 查询该用户是否抢购成功过
check = redis.get('has_buyed_'+uid)
if check and check>=1:
    return '你已经抢购过了'

redis.set('check_act'+uid, 1)
goods_id= redis.spop(goods_set_key)
if goods_id: # 抢购成功
    rv = do_business()
    if rv: # 业务处理成功：
        # 标记用户已经购买过
        redis.set('has_buyed_'+uid,1)
    else: # 将抢购的商品归还
        redis.sadd(goods_set_key, goods_id)
else:
    return '商品不足'
```
