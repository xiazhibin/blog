### sql 执行顺序
```sql
SELECT DISTINCT <select_list>
FROM <left_table>
<join_type> JOIN <right_table>
ON <join_condition>
WHERE <where_condition>
GROUP BY <group_by_list>
HAVING <having_condition>
ORDER BY <order_by_condition>
LIMIT <limit_number>
OFFSET <offset_number>
```

### group by
主要用于分组统计,一般都是使用在聚合函数中使用
select 的列必须出现在group by，不然就使用聚合函数

- `select count(*) from student group by name having age > 18;`
- `select name,max(age) from student group by name`
- `select name,age from student group by name;` #column `student.age` must appear in group by clause or be used in aggregate function

### distinct
用来去重的
只会保留指定的列的信息

- `select distinct name from student` # 根据名字去重，查找
- `select distinct name ,age from student` #根据名字和年龄一起去重

### distinct on 
postgresql特有

- `select distinct on (name) name,age from student` # 根据名字去重，搜出name,age两列
- `select distinct on (name) name,age from student order by name`
- `select distinct on (name) name,age from student order by age` # select distinct on expressions must match initial order by expressions
