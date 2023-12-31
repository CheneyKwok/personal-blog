---
title: MySQL - 全局锁与表级锁
date: 2023-11-28 15:33:01
tags: MySQL

---

**根据加锁范围，MySQL 大致可分为全局锁、表级锁和行锁。**

## 全局锁

即对整个数据库实例进行加锁。
全局读锁命令：

```sql
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
```sql
# 加锁
LOCK TABLES t1 READ/WRITE;

# 解锁
UNLOCK TABLES；
```
与 FTWRL 相同的是：客户端发生异常时，MySQL 会自动释放该表锁；
与 FTWRL 不同的是：lock table 时不仅会限制其他线程，也会限制当前线程。

对于不支持更细粒度的锁的引擎，表锁是最常用的处理并发的方式。而对于 InnoDB 引擎，其支持行锁，一般不使用表锁。

### 元数据锁（metadata lock， MDL）
MDL  无法手动干预，在访问一个表时 MySQL 会自动加上，直到事务提交才会释放。

MDL 的作用是保证 DDL 操作 与 DML 操作之间的一致性，当不能保证一致性时，会出现：

- 事务隔离问题：RR 的隔离级别下，在 session A 对 t 的两次查询期间，session B 对 t 的表结构做了修改，则会导致 session A 的两次查询结果不一致，无法满足可重复读。
- 数据同步问题：session A 对 t 执行更新操作并且未提交，session B 对 t 的表结构做了修改，此时 binlog 中会先记录 alter 操作，再记录更新操作，那么从库在同步时，也会先重做 alter，再重做 update。

执行 DML 操作时，会加 MDL 读锁；
执行 DDL  操作时，会加 MDL 写锁。

- 读锁之间不互斥， 因此可以多个线程可以对同一张表进行 DML。
- 读写锁之间、写锁之间是互斥的， 即 DML 、DDL 之间互相阻塞，DDL 之间互相阻塞。

MDL 可能导致的问题：对一张表执行了 DDL 操作，导致后续该表的所有DML 操作全部阻塞。

过程如下：

```sql
# session A
BEGIN;
SELECT * FROM `user`; # 注意事务未提交

# session B
ALTER TABLE `user` ADD INDEX index_name(name);

# session C
SELECT * FROM `user`;

```

此时 session B 和 session C 都会处于阻塞等待状态，执行  `SHOW PROCESSLIST;` 查看 session 状态：

![session 状态](https://cdn.jsdelivr.net/gh/CheneyKwok/img-storage/blog/MySQL-%E5%85%A8%E5%B1%80%E9%94%81%E4%B8%8E%E8%A1%A8%E7%BA%A7%E9%94%81-1.png)

发现 session B 和 session A 的 state 的是 `Waiting for table metadata lock`。

原因：session C 本身不会被 session A 阻塞，因为读锁之间不互斥，但是申请 metadata 锁的操作会形成一个队列，session B 写锁的获取优先级是高于 session C 读锁的，所以一旦写锁等待，不仅 session B 会被阻塞，session C 也会阻塞等待，后续所有访问该表的所有操作都会阻塞。如果该表中的某个查询语句频繁，并且客户端有重试机制，即超时后会再起一个 session 去请求，那么 MySQL 的连接很快就会消耗完，线程很快就会爆满。

#### 如何定位阻塞源头

performance_schema.metadata_locks 表会记录 metadata 锁的信息，包括持有对象、类型、状态等。5.7 默认设置是关闭的，8.0 默认打开，通过以下命令开启：

```sql
UPDATE performance_schema.setup_instruments SET ENABLED = 'YES', TIMED = 'YES' WHERE NAME = 'wait/lock/metadata/sql/mdl';
```
如果要永久生效，在配置文件中配置：

```yaml
[mysqld]performance-schema-instrument='wait/lock/metadata/sql/mdl=ON'
```

同时需要关联另外两张表  performance_schema.thread，performance_schema.events_statements_history，`thread`表记录了线程信息，`events_statements_history` 表记录了历史事件 sql。

关联后完整 sql ：

```sql
SELECT locked_schema,
locked_table,
locked_type,
waiting_processlist_id,
waiting_age,
waiting_query,
waiting_state,
blocking_processlist_id,
blocking_age,
substring_index(sql_text,"transaction_begin;" ,-1) AS blocking_query,
sql_kill_blocking_connection
FROM 
( 
SELECT 
b.OWNER_THREAD_ID AS granted_thread_id,
a.OBJECT_SCHEMA AS locked_schema,
a.OBJECT_NAME AS locked_table,
"Metadata Lock" AS locked_type,
c.PROCESSLIST_ID AS waiting_processlist_id,
c.PROCESSLIST_TIME AS waiting_age,
c.PROCESSLIST_INFO AS waiting_query,
c.PROCESSLIST_STATE AS waiting_state,
d.PROCESSLIST_ID AS blocking_processlist_id,
d.PROCESSLIST_TIME AS blocking_age,
d.PROCESSLIST_INFO AS blocking_query,
concat('KILL ', d.PROCESSLIST_ID) AS sql_kill_blocking_connection
FROM performance_schema.metadata_locks a JOIN performance_schema.metadata_locks b ON a.OBJECT_SCHEMA = b.OBJECT_SCHEMA AND a.OBJECT_NAME = b.OBJECT_NAME
AND a.lock_status = 'PENDING'
AND b.lock_status = 'GRANTED'
AND a.OWNER_THREAD_ID <> b.OWNER_THREAD_ID
AND a.lock_type = 'EXCLUSIVE'
JOIN performance_schema.threads c ON a.OWNER_THREAD_ID = c.THREAD_ID JOIN performance_schema.threads d ON b.OWNER_THREAD_ID = d.THREAD_ID
) t1,
(
SELECT thread_id, group_concat( CASE WHEN EVENT_NAME = 'statement/sql/begin' THEN "transaction_begin" ELSE sql_text END ORDER BY event_id SEPARATOR ";" ) AS sql_text
FROM
performance_schema.events_statements_history
GROUP BY thread_id
) t2
WHERE t1.granted_thread_id = t2.thread_id; \G

*************************** 1. row ***************************
               locked_schema: report-vision
                locked_table: students
                 locked_type: Metadata Lock
      waiting_processlist_id: 13
                 waiting_age: 938
               waiting_query: alter table students add column col1 int
               waiting_state: Waiting for table metadata lock
     blocking_processlist_id: 12
                blocking_age: 947
              blocking_query: SELECT * from students where id=1
	 sql_kill_blocking_connecti
on: KILL 12
1 row in set, 2 warnings (0.01 sec)
```
显示结果中，`processlist_id` 为 12 的线程阻塞了线程 13，所以 `kill 12` 即可解锁。

#### 如何避免 metadata 锁阻塞业务问题

- 开启 metadata_locks 表记录 MDL 锁
- 规范使用事务，及时提交事务，避免使用大事务
- DDL 操作级备份操作放在业务低峰期执行
- 设置 lock_wait_timeout 为较小值，让被阻塞端主动停止
<!--stackedit_data:
eyJoaXN0b3J5IjpbLTEyNzYwNTc4MTIsLTk0NTIwOTgwOSwtND
k5ODMxOTA1LDE4ODIwNDgwMDYsMTYzNTM2NzcxMiwtMTI4NzE3
MjA2NywxNzMzNjE3NzgyXX0=
-->