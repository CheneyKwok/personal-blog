---
title: MySQL - 全局锁与表锁
date: 2023-11-28 15:33:01
tags: MySQL
---

**根据加锁范围，MySQL 大致可分为全局锁、表锁和行锁。**

## 全局锁

即对整个数据库实例进行加锁，命令：

```yaml
 Flush tables with read;
```
     
执行该命令后，其他线程的以下语句会被阻塞：

 - 数据更新语句（增删改）
 - 数据定义语句（表的建立和修改、索引的建立和修改）
 >  更新类的事务的提交语句可以提交，原因待深究
 >  
**使用场景**：全库逻辑备份

缺点：

- 如果在主库上备份，则备份期间都不能执行更新；
- 如果在从库上备份，则备份期间从库无法执行从主库同步过来的 binlog，导致主从延迟。

**引申**

官方自带的逻辑备份工具是 mysqldump。

当 mysqldump 使用参数 –single-transaction 时，导出数据之前会启动一个事务，来确保拿到一致性视图，由于 m
<!--stackedit_data:
eyJoaXN0b3J5IjpbOTI3NzMzMTI1LC05OTkzNDAxMDgsLTY2MD
M3Nzk4NywtMjk0ODAyNDksLTYwODU0NzgzNywxMDQ2MTEzNjM3
LDEyNTE0Mzc0MzYsMjA2ODg0Njk3NSwtNTE0MDk2ODMxLDE5OT
EwNDM0MjcsLTE5NDM0NjU1MzYsLTEzNjk0NDYzMTAsLTUwMTAz
MDg2MF19
-->