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
que'dian
<!--stackedit_data:
eyJoaXN0b3J5IjpbMTM0OTEyODY4MiwtOTk5MzQwMTA4LC02Nj
AzNzc5ODcsLTI5NDgwMjQ5LC02MDg1NDc4MzcsMTA0NjExMzYz
NywxMjUxNDM3NDM2LDIwNjg4NDY5NzUsLTUxNDA5NjgzMSwxOT
kxMDQzNDI3LC0xOTQzNDY1NTM2LC0xMzY5NDQ2MzEwLC01MDEw
MzA4NjBdfQ==
-->