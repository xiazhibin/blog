## 记一次postgresql9.6流复制

### 准备

主库：192.168.1.10

从库：192.168.1.11

postgresql版本：9.6.5

操作系统：Ubuntu 14.04

### 配置主库

- 创建一个新用户
```
postgres=# CREATE USER repuser REPLICATION LOGIN CONNECTION LIMIT 2 ENCRYPTED PASSWORD '123456';
```
修改`pg_hba.conf`,添加

```
host   replication     repuser          192.168.1.0/24         md5
```

- 创建一个archive directory

`mkdir -p /var/lib/pgsql/backup_in_progress`

- 修改postgresql.conf
```

listen_addresses = '*'
archive_mode = on
archive_command = 'test ! -f /var/lib/pgsql/backup_in_progress || (test ! -f /var/lib/pgsql/archive/%f && cp %p /var/lib/pgsql/archive/%f)'
wal_level = replica       
hot_standby = on
max_wal_senders = 3          # 流复制的最大连接数
wal_keep_segments = 16        # 流复制保留的最大xlog数
logging_collector = on
log_directory = 'pg_log'
log_filename = 'postgresql-%Y-%m-%d_%H%M%S.log'
```

- 重启数据库
```
sudo service postgresql restart
```

## 配置从库

- 关闭数据库
```
sudo service postgresql stop
```

- 把之前的$PGDATA清理掉
```
mv /var/lib/postgresql/9.6/main /var/lib/postgresql/9.6/main_old # 注意文件的owner是不是postgres
```

- 使用pg_backendup生成备库。
```
pg_basebackup -D /var/lib/postgresql/9.6/main -Fp -Xs -v -P -h 192.168.1.10 -p 5432 -U repuser
```

- 修改`postgresql.conf`
`hot_standby = on` 

- 创建recovery.conf
```
cp -avr /usr/share/postgresql/9.6/recovery.conf.sample /var/lib/postgresql/9.6/main/recovery.conf
```

- 修改recovery.conf
```
standby_mode = on
primary_conninfo = 'host=192.168.1.10 port=5432 user=repuser password=123456'
trigger_file = '/tmp/postgresql.trigger.5432'
```

- 重启从库
```
service postgresql restart
```

### 到这里就差不多结束了。
可以试试在主库进行修改，看看从库有没有对应的变化

在主库可以看到这样的进程：postgres: 9.6/main: wal sender process repuser 


