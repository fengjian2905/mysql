# 为什么mysql会抖一下

> 症状：查询突然变得特变慢，持续时间短，无法复现

## 脏页

1. 内存中数据页与磁盘中数据页不一致的的时候，这个内存页称为**脏页**  

2. 脏页是由于InnoDB处理更新语句的时候造成的，只更新内存写redo log  

## 刷脏页时机

1. 内存满
   
   查询响应变长

2. redo log满
   
   这个时候，所有的更新都会阻塞

3. 空闲时间
   
4. mysql正常关闭

## 刷脏页策略（InnoDB）

> 一个查询要淘汰的脏页过多，会导致查询的响应时间明显变长 
> 日志写满，更新全部堵住，写性能跌为0

1. 脏页最大百分比默认75%，innodb_max_dirty_pages_pct
   
2. innodb_io_capaci1ty：通知 innodb 你的磁盘能力，推荐设置成磁盘的IOPS

测试磁盘IO工具：fio
``` sql
    fio -filename=$filename -direct=1 -iodepth 1 -thread -rw=randrw -ioengine=psync -bs=16k -size=500M -numjobs=10 -runtime=10 -group_reporting -name=mytest
```

没能正确设置 `innodb_io_capacity` 的会导致： 
MySQL的写入速度很慢，TPS很低，但是数据库主机的IO压力并不大

3. 控制你的脏页比例不要让他经常接近75%
   
    脏页比例 = innodb_buffer_pool_pages_dirty/innodb_buffer_pool_pages_total

## 刷脏页的另一个策略-把"邻居"顺带刷掉

``` sql
innodb_flush_neighbors
```

机械硬盘时代这种做法会显著提升系统性能  

SSD没必要，建议关闭 设置为 0 ，mysql 8.0已经默认关闭了
