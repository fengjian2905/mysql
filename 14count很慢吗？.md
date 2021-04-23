# count(*) 真的很慢吗？

> 结论：count(*)≈count(1)>count(主键 id)>count(字段)

## 原则

1. server 层要什么就给什么  
2. InnoDB只给必要的值
3. 优化器目前只优化了count(*)的语义：“取行数”

## 一探究竟

**count(主键 id)**，innodb 遍历整张表，每行 id 都取出来，返回给server，server 判断id必不为空，累加

**count(1)**，innodb 遍历整张表，并不取值，server层判断1必不空，累加

**count(字段)**，innodb 遍历整张表，取出字段，server层根据字段是否not null 决定是否判断，累加，并且不是 null 才会累加，否则跳过

**count(*)**，mysql做了专门优化，按行累加