---
title: MySQL - 全局锁与表锁
date: 2023-11-28 15:33:01
tags: MySQL
---

**根据加锁范围，MySQL 大致可分为全局锁、表锁和行锁。**

## 全局锁

即对整个数据库实例进行加锁，命令：

     Flush tables with read lock;

<!--stackedit_data:
eyJoaXN0b3J5IjpbNDc3MDM0NTM2LC0xOTQzNDY1NTM2LC0xMz
Y5NDQ2MzEwLC01MDEwMzA4NjBdfQ==
-->