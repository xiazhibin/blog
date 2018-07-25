### varint/zigzag是一种关于整数的编码方式，压缩整数
#### 1.varint编码
- 编码规则：最高位(most significant bit)表示编码是否继续，如果该位为1，表示接下来的字节仍然是该数字的一部分，
如果该位为0，表示编码结束。字节里的其余7位用原码补齐，采用低位字节补齐到高位的办法。

举几个数值具体说明下编码规则：

对数字`1`进行`varint`编码后的结果为`0000 0001`，占用1个字节。相比`int3`2的数字`1`存储占用4个字节，节省了3个字节的空间（主要是高位的0），
而Varint的想法就是 以标志位替换掉高字节的若干个0。对数字300进行varint编码，原码形式为：`00000000 00000000 00000001 00101100`
编码后的数据：第一个字节为`10101100`，第二个字节为`00000010`（不够7位高位补0）。

```python2
def encodeVInt(n):
    bytes = ''
    while n > 0x7F:
        bytes += chr((n & 0xf7)|0x80) #或0x80让最高位为1，表示接下来的字节仍然是该数字的一部分
        n >> 7
    bytes += chr(n)
    return bytes

def decodeVInt(data):
    shift = 0
    result = 0
    for c in data:
        b = ord(c)
        result |= ((b & 0x7f)<<shift)
        if not (b & 0x80):
            break
        shift += 7
    return result
```

- `varint`编码后二进制数据是不定长的，数值越小的数字使用的字节数越少。
例如对于`int32`，采用`varint`编码后需要1~5个bytes，小的数字使用1个byte，大的数字使用5个bytes。
基于实际场景中小数字的使用远远多于大数字，因此通过Varint编码对于大部分场景都可以起到一个压缩的效果。

### 2.zigzag编码
上面的`varint`编码有个问题是对于负数是不起作用的，因为负数最高位都是1,不能作为一个标记位。而`zigzag`就是一种将有符号数
统一映射成无符号数的一种编码方式。

- 编码规则:将符号位移到低位，剩下的位左移1位。

```python2
def encodeZigZag32(n): return (n << 1) ^ (n >> 31)
def decodeZigZag32(n): return (n >> 1) ^ -(n & 1)
```
