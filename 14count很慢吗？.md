# count(*) 真的很慢吗？

> 结论：count(*)≈count(1)>count(主键id)>count(字段)

## 原则

1. server 层要什么就给什么  
2. InnoDB只给必要的值
3. 优化器目前只优化了count(*)的语义：“取行数”

