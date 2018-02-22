### 准备测试文本
`netstat > netstat.txt`

```txt
Proto Recv-Q Send-Q  Local Address          Foreign Address        (state)
tcp4       0      0  192.168.1.170.62716    115.159.241.223.https  ESTABLISHED
tcp4       0      0  192.168.1.170.62714    220.181.76.82.http     CLOSE_WAIT
tcp4       0      0  192.168.1.170.62713    108.177.125.188.5228   SYN_SENT
tcp4       0      0  192.168.1.170.62712    108.177.125.188.5228   SYN_SENT
tcp4       0      0  192.168.1.170.62711    115.159.241.223.https  ESTABLISHED
tcp4       0      0  192.168.1.170.62710    115.159.241.223.https  ESTABLISHED
tcp4       0      0  192.168.1.170.62709    115.159.241.223.https  ESTABLISHED
tcp4       0      0  192.168.1.170.62708    stackoverflow.co.https ESTABLISHED
tcp4       0      0  localhost.socks        localhost.62653        FIN_WAIT_2
tcp4       0      0  localhost.62653        localhost.socks        CLOSE_WAIT
tcp4       0      0  localhost.socks        localhost.62651        FIN_WAIT_2
tcp4       0      0  localhost.62651        localhost.socks        CLOSE_WAIT
tcp4       0      0  192.168.1.170.62601    140.205.32.75.https    ESTABLISHED
tcp4       0      0  192.168.1.170.62581    14.215.158.102.https   ESTABLISHED
tcp4       0      0  192.168.1.170.62579    17.188.166.14.5223     ESTABLISHED
tcp4       0      0  192.168.1.170.62578    17.252.156.218.https   ESTABLISHED
tcp6       0      0  fe80::300d:230a:.black fe80::c075:b008:.45431 ESTABLISHED
tcp6       0      0  fe80::300d:230a:.1024  fe80::c075:b008:.1024  ESTABLISHED
tcp4      31      0  192.168.1.170.49171    45.55.41.223.https     CLOSE_WAIT
```

### awk命令格式
`awk [选项参数] 'script' var=value file`


### 基本使用
- 打印某些列 `awk '{print $1, $4}' netstat.txt`
- 格式化输出 `awk '{printf "%-8s %-8s %-8s %-18s %-22s %-15s\n",$1,$2,$3,$4,$5,$6}' netstat.txt`

### 过滤条件
- `awk '$3>0 {print $1}' netstat.txt`
- `awk '$3==0 && $6=="ESTABLISHED" || NR==1 ' netstat.txt` //内建变量NR表示awk开始执行程序后所读取的数据行数
- `awk '$3==0 && $6=="ESTABLISHED" || NR==1 {printf "%-20s %-20s %s\n",$4,$5,$6}' netstat.txt`

### 内建变量
| 变量||
| :---------    | :----: |
| $0	| 当前记录（这个变量中存放着整个行的内容）|
|$1~$n	|当前记录的第n个字段，字段间由FS分隔 |
|FS	|输入字段分隔符 默认是空格或Tab|
|NF	|当前记录中的字段个数，就是有多少列|
|NR	|已经读出的记录数，就是行号，从1开始，如果有多个文件话，这个值也是不断累加中。|
|FNR	|当前记录数，与NR不同的是，这个值会是各个文件自己的行号|
|RS	|输入的记录分隔符， 默认为换行符|
|OFS	|输出字段分隔符， 默认也是空格|
|ORS	|输出的记录分隔符，默认为换行符|
|FILENAME	|当前输入文件的名字|

- 输出行号 `awk '$6=="ESTABLISHED" {printf "%2s %s \n",NR,$1}' netstat.txt`

### 指定分隔符
- `awk  'BEGIN{FS=":"} {print $1,$3,$6}' /etc/passwd` 等价于 `awk -F:  {print $1,$3,$6}`
- 转换分隔符 `awk '{print $1,$2,$3,$4,$5}' OFS="," netstat.txt`

### 正则表达式
- `awk '$6 ~ /FIN/ || NR==1 {print NR,$4,$5,$6}' OFS="\t" netstat.txt` 匹配FIN状态

### 拆分文件
- `awk 'NR!=1{print > $6}' netstat.txt` 按照第六列进行拆分
- `awk 'NR!=1{if($6 ~ /TIME|ESTABLISHED/) print > "1.txt";
else if($6 ~ /LISTEN/) print > "2.txt";
else print > "3.txt" }' netstat.txt`

### 统计
- `awk 'NR!=1{a[$6]++;} END {for (i in a) print i ", " a[i];}' netstat.txt`

### 学习网站
- [AWK程序设计语言](http://awk.readthedocs.io/en/latest/chapter-one.html)
