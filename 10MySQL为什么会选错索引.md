# MySQL为什么会选错索引

## 数据准备

```sql 
CREATE TABLE t(
    id int(11) NOT NULL,
    a int(11) DEFAULT NULL,
    b int(11) DEFAULT NULL,
    PRIMARY KEY (id),
    key a(a),
    key b(b) 
) ENGINE = InnoDB;
```

```sql
delimiter ;;
create procedure idata()
begin
declare i int;
set i=1;
while(i<=100000)do
    insert into t values(i,i,i);
    set i=i+1;
end while;
end ;;
delimiter ;
call idata();
```

## 选错索引一

1. sessionA
   
   ```sql
   start transaction with consistent snapshot
   ```

2. sessionB

    ```sql
    delete from t;
    call idata();
    explain select * from tt where a between 10000 and 20000;
    ```
索引`a`未使用到
+----+-------------+-------+------------+------+---------------+------+---------+------+-------+----------+-------------+
| id | select_type | table | partitions | type | possible_keys | key  | key_len | ref  | rows  | filtered | Extra       |
+----+-------------+-------+------------+------+---------------+------+---------+------+-------+----------+-------------+
|  1 | SIMPLE      | tt    | NULL       | ALL  | a             | NULL | NULL    | NULL | 99214 |    10.08 | Using where |
+----+-------------+-------+------------+------+---------------+------+---------+------+-------+----------+-------------+

3. sessionA
   
   ```sql
   commit
   ```

**解决**:重新统计信息    

```sql
analyze table t;
```

## 选错索引二

```sql
explain select * from t where (a between 1 and 1000) and (b between 50000 and 100000) order by b limit 1;
```

+----+-------------+-------+------------+-------+---------------+------+---------+------+-------+----------+------------------------------------+
| id | select_type | table | partitions | type  | possible_keys | key  | key_len | ref  | rows  | filtered | Extra                              |
+----+-------------+-------+------------+-------+---------------+------+---------+------+-------+----------+------------------------------------+
|  1 | SIMPLE      | tt    | NULL       | range | b,a           | b    | 5       | NULL | 50128 |     1.00 | Using index condition; Using where |
+----+-------------+-------+------------+-------+---------------+------+---------+------+-------+----------+------------------------------------+

**解决** 

```sql
explain select * from tt where (a between 1 and 1000) and (b between 50000 and 100000) order by b,a limit 1;
```