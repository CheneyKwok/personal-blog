---
title: MySQL - 全局锁与表锁
date: 2023-11-28 15:33:01
tags: MySQL
---

**根据加锁范围，MySQL 大致可分为全局锁、表锁和行锁。**

## 全局锁

即对整个数据库实例进行加锁，命令：

     Flush tables with read lock;
执行该命令后，其他线程的以下语句会被阻塞：

 - 数据更新语句（增删改）

<!--stackedit_data:
eyJoaXN0b3J5IjpbLTE1ODU1MDQ2NjAsLTE5NDM0NjU1MzYsLT
EzNjk0NDYzMTAsLTUwMTAzMDg2MF19
-->