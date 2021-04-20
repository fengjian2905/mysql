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

