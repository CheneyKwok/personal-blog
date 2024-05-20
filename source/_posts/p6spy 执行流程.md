---
title: p6spy 执行流程
date: 2023-12-10 10:53:20
tags: 框架

---
## 流程

1. 配置 p6spy 的 mysql 驱动：com.p6spy.engine.spy.P6SpyDriver
	![p6spy执行流程-1](https://cdn.jsdelivr.net/gh/CheneyKwok/img-storage/blog/p6spy执行流程-1.png)

2. 通过 P6SpyDriver 创建 Connection，返回 com.p6spy.engine.wrapper.ConnectionWrapper 包装真正的 Connection 
	![p6spy执行流程-2](https://cdn.jsdelivr.net/gh/CheneyKwok/img-storage/blog/p6spy执行流程-2.png)

	![p6spy执行流程-3](https://cdn.jsdelivr.net/gh/CheneyKwok/img-storage/blog/p6spy执行流程-3.png)

3. ConnectionWrapper 重写 createStatement()，返回 com.p6spy.engine.wrapper.StatementWrapper 包装真正的 Statement
	![p6spy执行流程-4](https://cdn.jsdelivr.net/gh/CheneyKwok/img-storage/blog/p6spy执行流程-4.png)

	![p6spy执行流程-5](https://cdn.jsdelivr.net/gh/CheneyKwok/img-storage/blog/p6spy执行流程-5.png)

4. StatementWrapper 重写 executeQuery()，在执行sql之后调用 JdbcEventListener 的 onAfterExecuteQuery()
	![p6spy执行流程-6](https://cdn.jsdelivr.net/gh/CheneyKwok/img-storage/blog/p6spy执行流程-6.png)

	![p6spy执行流程-7](https://cdn.jsdelivr.net/gh/CheneyKwok/img-storage/blog/p6spy执行流程-7.png)

5. 最终执行子类 LoggingEventListener 的 onAfterAnyExecute()，调用 P6Logger 的 logSQL()
	![p6spy执行流程-8](https://cdn.jsdelivr.net/gh/CheneyKwok/img-storage/blog/p6spy执行流程-8.png)

	![p6spy执行流程-9](https://cdn.jsdelivr.net/gh/CheneyKwok/img-storage/blog/p6spy执行流程-9.png)

	![p6spy执行流程-10](https://cdn.jsdelivr.net/gh/CheneyKwok/img-storage/blog/p6spy执行流程-10.png)
<!--stackedit_data:
eyJoaXN0b3J5IjpbLTE5MjQwNzYyNCwyMTEyNzc0NjgxLDE5Nj
c3NTM1NTMsLTExNDcwMzg1NjMsLTgyNjc1OTI1NSwtNTg5ODMz
OTM5LC0xMjUzMTczODU5LDE4ODc5MTM1ODYsNTk3NDgzNDY0XX
0=
-->