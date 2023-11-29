---
title: MySQL - 全局锁与表锁
date: 2023-11-28 15:33:01
tags: MySQL

---

**根据加锁范围，MySQL 大致可分为全局锁、表锁和行锁。**

## 全局锁

即对整个数据库实例进行加锁。
全局读锁命令：

```yaml
 # 加锁
 FLUSH TABLES WITH READ LOCK;
 # 解锁
 UNLOCK TABLES;
```
     
执行该命令后，**其他线程**的以下语句会被阻塞：

 - 数据更新语句（增删改）
 - 数据定义语句（表的建立和修改、索引的建立和修改）
 - 更新类事务的提交语句

**使用场景**：全库逻辑备份

缺点：

- 如果在主库上备份，则备份期间都不能执行更新；
- 如果在从库上备份，则备份期间从库无法执行从主库同步过来的 binlog，导致主从延迟。

**引申**

官方自带的逻辑备份工具是 mysqldump。
mysqldump 加上 –single-transaction  参数可以保证备份后的库是在一个逻辑时间点，并且备份期间可以正常更新。

> –single-transaction 使用的前提是库中的所有表的存储引擎都使用了事务，否则只能使用 FTWRL。

原因是当 mysqldump 使用参数 –single-transaction 时，导出数据之前会启动一个事务，来确保拿到一致性视图。而由于 MVCC 的支持，可以执行更新。

实现全库只读，`set global readonly = true` 是否可以？
可以实现只读，但会有其他影响：

- readonly 的值可能会用于其他逻辑的判断，比如用来判断一个库是主库还是从库。
- 异常处理机制上的差异。FTWRL 命令在客户端异常断开时，MySQL 会自动释放该全局锁，而设置 readonly 之后，如果客户端发生异常，整个库会一直保持 readonly 状态。

## 表级锁
表级锁分为两种：表锁、元数据锁（metadata lock，MDL）。

###  表锁
表锁命令：
```yaml
# 加锁
LOCK TABLES t1 READ/WRITE;
# 解锁
UNLOCK TABLES；
```
与 FTWRL 相同的是：客户端发生异常时，MySQL 会自动释放该表锁；
与 FTWRL 不同的是：lock table 时不仅会限制其他线程，也会限制当前线程。

对于不支持更细粒度的锁的引擎，表锁是最常用的处理并发的方式。而对于 InnoDB 引擎，其支持行锁，一般不使用表锁。

### 元数据锁（metadata lock， MDL）
MDL b'v
<!--stackedit_data:
eyJoaXN0b3J5IjpbMTU2NjYxOTA4NCw2Nzc5MDI2NjYsMjEyMz
I0OTIxNywxMTA2MzEzMDU2LC03MzkxMjM4NTksLTQ1OTY5OTE4
MiwxOTYzMjk0ODY1LDE0NDYwMTE4NywxOTYzMjk0ODY1LC0xMz
EyMjk0MywtOTk5MzQwMTA4LC02NjAzNzc5ODcsLTI5NDgwMjQ5
LC02MDg1NDc4MzcsMTA0NjExMzYzNywxMjUxNDM3NDM2LDIwNj
g4NDY5NzUsLTUxNDA5NjgzMSwxOTkxMDQzNDI3LC0xOTQzNDY1
NTM2XX0=
-->