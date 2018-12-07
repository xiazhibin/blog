### 写在头

我所知道的，bash中，目前有五种方法：
1. i=`expr $i + 1`;
2. let i+=1;
3. ((i++));
4. i=$[$i+1];
5. i=$(( $i + 1 ))

### 对于自增变量
```bash
#!/bin/bash
for j in $(seq 1 5)
do
  echo $j
done
```
