---
title: p6spy 执行流程
date: 2023-12-10 10:53:20
tags: 框架

---
## p6spy 执行流程

1. 配置 p6spy 的 mysql 驱动：com.p6spy.engine.spy.P6SpyDriver
![session 状态](https://cdn.jsdelivr.net/gh/CheneyKwok/img-storage/blog/p6spy执行流程-1.png)

2. 通过 P6SpyDriver 创建 connection，返回 com.p6spy.engine.wrapper.ConnectionWrapper 对真正的 connection 包装
<!--stackedit_data:
eyJoaXN0b3J5IjpbLTU4OTgzMzkzOSwtMTI1MzE3Mzg1OSwxOD
g3OTEzNTg2LDU5NzQ4MzQ2NF19
-->