---
title: p6spy 执行流程
date: 2023-12-10 10:53:20
tags: 框架

---
## p6spy 执行流程

1. 配置 p6spy 的 mysql 驱动：com.p6spy.engine.spy.P6SpyDriver
![p6spy执行流程](https://cdn.jsdelivr.net/gh/CheneyKwok/img-storage/blog/p6spy执行流程-1.png)

2. 通过 P6SpyDriver 创建 Connection，返回 com.p6spy.engine.wrapper.ConnectionWrapper 包装真正的 Connection 
![p6spy执行流程](https://cdn.jsdelivr.net/gh/CheneyKwok/img-storage/blog/p6spy执行流程-2.png)

![p6spy执行流程](https://cdn.jsdelivr.net/gh/CheneyKwok/img-storage/blog/p6spy执行流程-3.png)

3. ConnectionWrapper 重写 createStatement()，返回 com.p6spy.engine.wrapper.StatementWrapper 包装真正的 Statement
<!--stackedit_data:
eyJoaXN0b3J5IjpbLTE4MTc0MTQ1NTAsLTgyNjc1OTI1NSwtNT
g5ODMzOTM5LC0xMjUzMTczODU5LDE4ODc5MTM1ODYsNTk3NDgz
NDY0XX0=
-->