[like走不走索引](https://github.com/xiazhibin/blog/blob/master/postgresl%E5%B0%8F%E7%AC%94%E8%AE%B0.md#like%E8%B5%B0%E4%B8%8D%E8%B5%B0%E7%B4%A2%E5%BC%95-1)
[select for update]()
-------------

## like走不走索引

### 场景 lower(name) like 'pf%'
#### 普通btree
`CREATE INDEX users_idx0 ON users (name);`
- 全字匹配查询，走索引`select * from user where name='pf'`
- 加函数全字匹配，不走索引`select * from user where lower(name)='pf'`
- 模糊匹配，不走索引`select * from user where name like 'pf%'`

#### 字段带函数的btree
`CREATE INDEX users_dex1 ON users (lower(name));`

- 全字匹配查询，不走索引`select * from user where name='pf'`
- 加函数全字匹配，走索引`select * from user where lower(name)='pf'`
- 模糊匹配，不走索引`select * from user where name like 'pf%'`

#### 声明操作符类的btree索引
`CREATE INDEX users_dex2 ON users (lower(name) varchar_pattern_ops);`

- 全字匹配查询，走索引`select * from user where name='pf'`
- 加函数全字匹配，不走索引`select * from user where lower(name)='pf'`
- 模糊匹配，走索引`select * from user where name like 'pf%'`

### 场景 lower(name) like '%pf%'

#### 声明操作符类的btree索引
`CREATE INDEX users_dex2 ON users (lower(name) varchar_pattern_ops);`

- 模糊匹配，不走索引`select * from user where name like '%pf%'`
- 这种场景下，要使用索引，通过`pg_trgm`

## select for update
pass
