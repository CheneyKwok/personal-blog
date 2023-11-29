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

`set global readonly = true` 命令也可以实现全库只读，但会有其他影响：

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
MDL  无法手动干预，在访问一个表时 MySQL 会自动加上。

MDL 的作用是保证 DDL 操作 与 DML 操作之间的一致性，当不能保证一致性时，会出现：

- 事务隔离问题：RR 的隔离级别下，在 session A 对 t 的两次查询期间，session B 对 t 的表结构做了修改，则会导致 session A 的两次查询结果不一致，无法满足可重复读。
- 数据同步问题：session A 对 t 执行更新操作并且未提交，session B 对 t 的表结构做了修改，此时 binlog 中会先记录 alter 操作，再记录更新操作，那么从库在同步时，也会先重做 alter，再重做 update。

执行 DML 


<!--stackedit_data:
eyJoaXN0b3J5IjpbLTE5NDM3MTYwNjUsLTU1MDUzMDcyMiwtOD
Y5NTE3Mjk3LDY3NzkwMjY2NiwyMTIzMjQ5MjE3LDExMDYzMTMw
NTYsLTczOTEyMzg1OSwtNDU5Njk5MTgyLDE5NjMyOTQ4NjUsMT
Q0NjAxMTg3LDE5NjMyOTQ4NjUsLTEzMTIyOTQzLC05OTkzNDAx
MDgsLTY2MDM3Nzk4NywtMjk0ODAyNDksLTYwODU0NzgzNywxMD
Q2MTEzNjM3LDEyNTE0Mzc0MzYsMjA2ODg0Njk3NSwtNTE0MDk2
ODMxXX0=
-->