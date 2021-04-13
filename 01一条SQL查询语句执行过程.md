# 一条 SQL 查询语句执行过程

## 逻辑架构

1. server 层
   
连接器 查询缓存 分析器 优化器 执行器

2. 存储引擎层
   
MyISAM InnoDB MEMORY

## 连接器

> 连接器负责跟客户端建立连接、获取操作权限、维持和管理连接

1. 连接数据库
  
```SQL  
   mysql -uroot -p
```

2. 查看连接
  
```SQL
   show processlist
```

| Id | User            | Host            | db   | Command | Time | State                  | Info             |
|----|-----------------|-----------------|------|---------|------|------------------------|------------------|
|  4 | event_scheduler | localhost       | NULL | Daemon  | 5635 | Waiting on empty queue | NULL             |
| 11 | root            | localhost:52891 | NULL | Query   |    0 | starting               | show processlist |

3. 用户连接一旦建立，权限就固定，后续修改用户权限必须重新建立连接后方可生效
  
4. 连接器空闲自动断开时间 默认`8`小时，由`wait_timeout`控制
  
```SQL  
show variables like 'wait_timeout'
```

| Variable_name | Value |
|---------------|-------|
| wait_timeout  | 28800 |

```SQL
   set session wait_timeout 10
```

```SQL
   show tables 
```

`连接断开错误`：ERROR 2013 (HY000): Lost connection to MySQL server during query

5. 长连接 vs 短连接
   
* 连接的建立过程比较复杂，尽量使用长链接
* 长链接有些时候会mysql占用内存涨的过快，导致mysql异常重启（因为系统会强行kill掉）
* 又想使用长链接怎么办呢？
定期断开长链接
5.7以上版本可以使用`mysql_reset_connection`命令初始化连接资源（无需重连和初始化资源）

## 查询缓存

> 不推荐使用，8.0之后已移除查询缓存

## 分析器

* 词法分析：表名 列名...
* 语法分析：select where ...

## 优化器

* 索引选择
* join顺序

## 执行器

> row_examined <= 引擎扫描行数

* 执行权限查询：select update delete query
* 调用搜索引擎接口
* 返回数据到客户端

