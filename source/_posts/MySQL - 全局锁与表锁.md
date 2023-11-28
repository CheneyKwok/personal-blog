---
title: MySQL - 全局锁与表锁
date: 2023-11-28 15:33:01
tags: MySQL

---

**根据加锁范围，MySQL 大致可分为全局锁、表锁和行锁。**

## 全局锁

即对整个数据库实例进行加锁，命令：

```yaml
 Flush tables with read lock;
```
     
执行该命令后，其他线程的以下语句会被阻塞：

 - 数据更新语句（增删改）
 - 数据定义语句（表的建立和修改、索引的建立和修改）
 >  更新类的事务的提交语句可以提交，原因待深究

**使用场景**：全库逻辑备份

缺点：

- 如果在主库上备份，则备份期间都不能执行更新；
- 如果在从库上备份，则备份期间从库无法执行从主库同步过来的 binlog，导致主从延迟。

**引申**

官方自带的逻辑备份工具是 mysqldump。前提是当前MySQL 的引擎要

mysqldump 加上 –single-transaction  参数可以保证备份后的库是在一个逻辑时间点，并且备份期间可以正常更新。

> –single-transaction 使用的前提是库中的所有表的存储引擎都使用了事务，否则只能使用 FTWRL。

原因是当 mysqldump 使用参数 –single-transaction 时，导出数据之前会启动一个事务，来确保拿到一致性视图。而由于 MVCC 的支持，可以执行更新。
<!--stackedit_data:
eyJoaXN0b3J5IjpbMTQ4MDk0MTUzNywtOTk5MzQwMTA4LC02Nj
AzNzc5ODcsLTI5NDgwMjQ5LC02MDg1NDc4MzcsMTA0NjExMzYz
NywxMjUxNDM3NDM2LDIwNjg4NDY5NzUsLTUxNDA5NjgzMSwxOT
kxMDQzNDI3LC0xOTQzNDY1NTM2LC0xMzY5NDQ2MzEwLC01MDEw
MzA4NjBdfQ==
-->