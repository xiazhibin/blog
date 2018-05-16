### 环境
- 已经完成步骤：[postgresql9.6流复制](https://github.com/xiazhibin/blog/blob/master/20171019_01.md)
- OS: Ubuntu 14.04
- Postgresql: PostgreSQL 9.6.9 #SELECT version();
- 主库：192.168.1.10
- 从库：192.168.1.11

### 主从库都安装`pgpool2`
使用`sudo apt-get install pgpool2`方法安装
配置文件在`/etc/pgpool2`下

### 修改主从库的`pgpool.conf`
```
listen_addresses = '*'
backend_hostname0 = '192.168.1.10' #主机ip
backend_port0 = 5432
backend_weight0 = 1 #loadbalance不开启，无效
backend_data_directory0 = '/var/lib/postgresql/9.6/main'
backend_flag0 = 'ALLOW_TO_FAILOVER'
 
backend_hostname1 = '192.168.1.11' #备机ip
backend_port1 = 5432
backend_weight1 = 1
backend_data_directory1 = '/var/lib/postgresql/9.6/main'
backend_flag1 = 'ALLOW_TO_FAILOVER'
 
enable_pool_hba = on
pool_passwd = 'pool_passwd'
 
master_slave_mode = on
master_slave_sub_mode = 'stream'
sr_check_period = 2
sr_check_user = 'repuser' #流复制账号
sr_check_password = 'password'
```
### 修改主从库的`pool_hba.conf`，增加
```
host    all         all         192.168.1.0/24          md5
host    all         all         127.0.0.1/32          md5
```
### 修改主从库的`pg_hga.conf`,增加
```
host    all             all             192.168.1.0/24            md5
```

### 重启`pgpool2`看是否成功
```
psql -h 192.168.1.10 -p 5434 -U postgres testdb
#show pool_nodes;
node_id |   hostname   | port | status | lb_weight |  role   | select_cnt | load_balance_node | replication_delay
---------+--------------+------+--------+-----------+---------+------------+-------------------+-------------------
 0       | 192.168.1.10 | 5432 | up     | 0.500000  | primary | 3          | false             | 0
 1       | 192.168.1.11 | 5432 | up     | 0.500000  | standby | 1          | true              | 0
(2 rows)
```
如果看到这个已经成功了。

### 测试load balance
- 停止`pgpool`,使用`sudo pgpool -n -d > /tmp/pgpool.log 2>&1 &` 启动`pgpool`。
- 执行`#testdb=# select 1`,从`pgpool.log`输出
```
DEBUG:  pool_virtual_master_db_node_id: virtual_master_node_id:1 load_balance_node_id:1 PRIMARY_NODE_ID:0
2018-05-16 17:41:57: pid 62605: DEBUG:  processing ReadyForQuery
```

- 执行`#testdb=# /*REPLICATION*/ SELECT 1;`从`pgpool.log`输出
```
DEBUG:  pool_virtual_master_db_node_id: virtual_master_node_id:0 load_balance_node_id:1 PRIMARY_NODE_ID:0
```
- 可以看到`select`是会去到slave，除非强制在master上执行。`select for update`就在master上执行

### 这里忽略了`pcp.conf`和`pool_passwd.conf`的配置

### 常见问题
- 从库的status是`unused`或者`down`
解决方法：先停止服务，然后删掉`pool_status`文件，文件位置在`pgpool.conf的logdir`

- `psql: ERROR:  MD5 authentication is unsupported in replication, master-slave and parallel modes. `看到这个，好好检查`pg_hba.conf`，
使用`psql`互相连接数据库，确保是可以互通的
