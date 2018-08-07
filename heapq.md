### heapq默认是最小值堆
最小堆是完全二叉树
```python
heapq.heappush(list, data)
heapq.heappop(list) 
heapq.heappushpop(list, data) # 先把item加入到堆中，然后再pop 
heapq.heapreplace(list, data) # 先pop，然后再把item加入到堆中
heapq.heapify(x)
heapq.nlargest(n, iterable, key=None) 返回最大的n个元素（Top-K问题）
heapq.nsmallest(n, iterable, key=None) 返回最小的n个元素（Top-K问题）
```

### topK堆
```python
import heapq
from collections import namedtuple

item = namedtuple('item', 'value, key')

class TopHeap:

    def __init__(self, k):
        self._k = k
        self._data = []

    def push(self, item):
        if len(self._data) < self._k:
            heapq.heappush(self._data, item)
        else:
            topk_small = self._data[0]
            if item.value > topk_small.value:
                heapq.heapreplace(self._data, item)

    def topK(self):
        return heapq.nlargest(self._k, self._data)
```

### bottomK堆
比较trick，push进去的是相反数，所以小的就变大了，根据上面的例子，最后保存的是最大的N个数，反过来就是最小的N个数

```python
class BottomHeap(object):
    def __init__(self, k):
        self.k = k
        self.data = []

    def push(self, elem):
        elem = -elem
        if len(self.data) < self.k:
            heapq.heappush(self.data, elem)
        else:
            topk_small = self.data[0]
            if elem > topk_small:
                heapq.heapreplace(self.data, elem)

    def btmK(self):
        return sorted([-x for x in self.data])
```
