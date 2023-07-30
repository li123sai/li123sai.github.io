---
title: Oracle表字段变更方法
date: 2023-07-30 16:58:34
categories:
- 数据库
- Oracle
tags:
- 字段变更
---

## 一、背景

​		上线部署数据库脚本时，数据库服务器CPU突然飙升100%。检查数据库脚本定位到问题。但是这个语句在测试环境执行没有问题。

```sql
-- 模拟语句
ALTER TABLE ORD_INFO ADD OJUST TIME TIMESTAMP(6) DEFAULT (nu11);
COMMENT ON COLUMN ORD_INFO.OJUST TIME IS 时间;
```

​		

## 二、问题解析

```sql
ALTER TABLE ORD_INFO ADD OJUST TIME TIMESTAMP(6) DEFAULT (nu11);
在测试环境分析 trace可以发现在进行上面DDL 操作时的动作分别为:
1.锁表(快)，等待事件为 library cachelock。
2.新增字段(快)，等待事件为 library cache lock。
3.Update 新增字段值为 null(慢)，执行计划为全表扫描，执行速度由表的数据
量决定，等待事件为 db fle sequential read。
```

## 三、总结

**Oracle不同版本针对新增字段做的优化也不一样，本次测试时Release 12.2.0.1.0：**

1. 新增字段时 Oracle针对 null值不会做优化，会使用 Update 更新到每一行中。
2. 新增字段时都会用LOCK TABLE IN ROW EXCLUSIVE(行排它表锁)。 (个别情况会使用 LOCK TABLE..EXCLUSIVE MODE--最高等级的锁，例如ALTER TABLE ... modifv)
3. 如果新增字段没有默认值，由于不需要更新字段值执行时间很短。
4. 如果新增字段使用默认值,并且不是 null，Oracle会做优化，default xx 和 defalut xx not null都会把默认值存到字典表中。(网上资料 Oracle llg 只有在 defalut xx not
   null 会做优化，default xx 不会)
5. 在对大表字段做操作时，如果无法预估影响时，可以先查看 trace 日志，分析内部操作。



