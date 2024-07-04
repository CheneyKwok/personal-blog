---
title: p6spy 配置及执行流程
date: 2023-12-10 10:53:20
tags: 框架

---
## 配置
1. 引入 maven 依赖
```
<dependency>  
 <groupId>p6spy</groupId>  
 <artifactId>p6spy</artifactId>  
 <version>${p6spy.version}</version>  
</dependency>
```
2.  配置 spy.properties 文件
```
logMessageFormat=com.xxx.P6SpyLogger  
appender=com.p6spy.engine.spy.appender.Slf4JLogger  
dateformat=yyyy-MM-dd HH:mm:ss  
# 过滤 druid 空闲检测语句  
filter=true  
exclude=DUAL
```
3.  自定义日志格式
```java
public class P6SpyLogger implements MessageFormattingStrategy {  
    /**  
	 * @param connectionId: 连接ID  
	 * @param now: 当前时间  
	 * @param elapsed: 花费时间  
	 * @param category: 类别  
	 * @param prepared: 预编译SQL  
	 * @param sql: 最终执行的SQL  
	 * @param url: 数据库连接地址  
	 * @return 格式化日志结果  
	  */  
  @Override  
  public String formatMessage(int connectionId, String now, long elapsed, String category, String prepared, String sql, String url) {  
  if (StrUtil.isBlank(sql) || "SELECT 1 FROM DUAL".equals(sql)) {  
  return "";  
 } else {  
  return " took：" + elapsed + " ms" + " | sql：" + sql.replaceAll("[\\s]+", " ") + "\n";  
 }  
 }  
}
```

## 流程

1. 配置 p6spy 的 mysql 驱动：com.p6spy.engine.spy.P6SpyDriver
	![p6spy执行流程-1](https://cdn.jsdelivr.net/gh/CheneyKwok/img-storage/blog/p6spy执行流程-1.png)

2. 通过 P6SpyDriver 创建 Connection，返回 com.p6spy.engine.wrapper.ConnectionWrapper 包装真正的 Connection 
	![p6spy执行流程-2](https://cdn.jsdelivr.net/gh/CheneyKwok/img-storage/blog/p6spy执行流程-2.png)

	![p6spy执行流程-3](https://cdn.jsdelivr.net/gh/CheneyKwok/img-storage/blog/p6spy执行流程-3.png)

3. ConnectionWrapper 重写 createStatement()，返回 com.p6spy.engine.wrapper.StatementWrapper 包装真正的 Statement
	![p6spy执行流程-4](https://cdn.jsdelivr.net/gh/CheneyKwok/img-storage/blog/p6spy执行流程-4.png)

	![p6spy执行流程-5](https://cdn.jsdelivr.net/gh/CheneyKwok/img-storage/blog/p6spy执行流程-5.png)

4. StatementWrapper 重写 executeQuery()，在执行 sql 之后调用 JdbcEventListener 的 onAfterExecuteQuery()
	
	![p6spy执行流程-6](https://cdn.jsdelivr.net/gh/CheneyKwok/img-storage/blog/p6spy执行流程-6.png)

	![p6spy执行流程-7](https://cdn.jsdelivr.net/gh/CheneyKwok/img-storage/blog/p6spy执行流程-7.png)

5. 最终执行子类 LoggingEventListener 的 onAfterAnyExecute()，调用 P6Logger 的 logSQL()
	![p6spy执行流程-8](https://cdn.jsdelivr.net/gh/CheneyKwok/img-storage/blog/p6spy执行流程-8.png)

	![p6spy执行流程-9](https://cdn.jsdelivr.net/gh/CheneyKwok/img-storage/blog/p6spy执行流程-9.png)

	![p6spy执行流程-10](https://cdn.jsdelivr.net/gh/CheneyKwok/img-storage/blog/p6spy执行流程-10.png)
<!--stackedit_data:
eyJoaXN0b3J5IjpbMTMxNjc2NDM5LC0zMDU3OTU1NDIsNTE5MT
E3MTgwLDIxMTI3NzQ2ODEsMTk2Nzc1MzU1MywtMTE0NzAzODU2
MywtODI2NzU5MjU1LC01ODk4MzM5MzksLTEyNTMxNzM4NTksMT
g4NzkxMzU4Niw1OTc0ODM0NjRdfQ==
-->