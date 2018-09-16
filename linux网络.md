### 常用命令
- route -n show / manipulate the IP routing table
- traceroute 

### iptalbes


格式：iptables [-t table] COMMAND chain CRETIRIA -j ACTION

-t table ：filter nat mangle

COMMAND：定义如何对规则进行管理 -A -I

chain：指定你接下来的规则到底是在哪个链上操作的，当定义策略的时候，是可以省略的 [OUTPUT, INPUT, FORWARD,POSTROUTING,PREROUTING

CRETIRIA:指定匹配标准 -s source_ip -p tcp

-j ACTION :指定如何进行处理 ACCEPT DROP MASQUERADE(伪装原地址，SNAT)

[iptables全攻略](https://www.kancloud.cn/hx78/linux/323876)
