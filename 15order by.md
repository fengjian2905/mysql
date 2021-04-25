# order by 是怎么工作的？

> select city,name,age from ttt where city = '上海' order by name limit 1000;

## 数据准备

``` sql
CREATE TABLE `ttt` (
  `id` int(11) NOT NULL,
  `city` varchar(16) NOT NULL,
  `name` varchar(16) NOT NULL,
  `age` int(11) NOT NULL,
  `addr` varchar(128) DEFAULT NULL,
  PRIMARY KEY (`id`),
  KEY `city` (`city`)
) ENGINE=InnoDB
```

## 全字段排序

mysql 会开辟一块内存 sort buffer，存放所有满足条件的记录的 city name age，
之后按照 name 排序 取 1000 条返回个客户端  

若排序的数据量过大，还会使用磁盘临时文件辅助排序  

## rowid排序

相比全字段，只会存放 主键 id 和要排序的字段，但是要回表

## 如何优化排序

1. 联合索引- city 和 name age创建联合索引，不用sort buffer排序了
2. 覆盖索引-避免回表 extra 中显示 using index



