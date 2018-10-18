- [like走不走索引](#like%E8%B5%B0%E4%B8%8D%E8%B5%B0%E7%B4%A2%E5%BC%95)
- [select for update](#行级锁select--for-)

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

## 行级锁(SELECT .. FOR ..)
`FOR UPDATE`, `FOR NO KEY UPDATE`,`FOR SHARE`和`FOR KEY SHARE`是锁定子句;他们影响`SELECT`如何从表中锁定行作为获得的行。锁定子句的一般形式：

 `FOR lock_strength [ OF table_name [, ...] ] [ NOWAIT ]`
 
```sql
select * from public.table01 where table01.id=1 FOR UPDATE OF table01;
select * from public.table01 where table01.id=1 FOR SHARE OF table01;
```
### 4个行级锁强度
- FOR UPDATE: 令那些被`SELECT`检索出来的行被锁住，就像在更新一样。这样就避免它们在当前事务结束前被**其它事务**修改或者删除；也就是说， 其它企图`UPDATE`,`DELETE`,`SELECT FOR UPDATE`,`SELECT FOR NO KEY UPDATE`,`SELECT FOR SHARE`或`SELECT FOR KEY SHARE`这些行的事务将被阻塞，直到当前事务结束。当然你只是想`select`的话是不会阻塞的，因为`select`表示你没有修改该行的打算，否则就使用`select .. for ..`。
- FOR NO KEY UPDATE: 行为类似于`FOR UPDATE`，只是获得的锁比较弱：该锁不阻塞尝试在相同的行上获得锁的`SELECT FOR KEY SHARE`命令。该锁模式也可以通过任何不争取`FOR UPDATE`锁的`UPDATE`获得。
- FOR SHARE: 行为类似于`FOR NO KEY UPDATE`，只是它在每个检索出来的行上获得一个共享锁，而不是一个排它锁。一个共享锁阻塞其它事务在这些行上执行`UPDATE`,`DELETE`,`SELECT FOR UPDATE`或`SELECT FOR NO KEY UPDATE`，却不阻止他们执行`SELECT FOR SHARE`或`SELECT FOR KEY SHARE`。
- FOR KEY SHARE: 行为类似于`FOR SHARE`，只是获得的锁比较弱： 阻塞`SELECT FOR UPDATE`但不阻塞`SELECT FOR NO KEY UPDATE`。一个共享锁阻塞其他事务执行`DELETE`或任意改变键值的`UPDATE`， 但是不阻塞其他`UPDATE`，也不阻止`SELECT FOR NO KEY UPDATE`,` SELECT FOR SHARE`或`SELECT FOR KEY SHARE`。

 为了避免操作等待其它事务提交，使用NOWAIT选项。如果被选择的行不能立即被锁住， 那么语句将会立即汇报一个错误，而不是等待。请注意，NOWAIT只适用于行级别的锁， 要求的表级锁ROW SHARE仍然以通常的方法进行
