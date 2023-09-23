---
title: Kettle实现循环动态表输出
date: 2023-09-23 13:02:11
categories:
- 工具
- Kettle
tags:
- Kettle
---

## 一、背景

​		系统A需要往系统B迁移，数据库不变。系统B有个针对分库分表的索引表，需要根据目前分库分表数据聚合出来。 数据库使用的Mysql，目前已经是百库千表。

## 二、整体思路

1. 获取涉及分库的全部schema

   ```sql
   SELECT table_schema AS table_schema1,table_schema AS table_schema2 FROM (SELECT DISTINCT table_schema FROM information_schema.tables ) t WHERE t.table_schema LIKE 'schema前缀%' or t.table_schema LIKE 'schema前缀%'
   ```

2. 将每个schema下的表名字组装出来

   ```sql
   SELECT CONCAT('`',s.tableSchema,'`','.','`',s.TABLE_NAME,'`' )   AS TABLE_NAME  FROM (SELECT ? AS tableSchema ,TABLE_NAME   FROM INFORMATION_SCHEMA.TABLES WHERE TABLE_SCHEMA = ? ) s WHERE s.TABLE_NAME LIKE  'schema下表前缀%'
   ```

3. 根据组装的表名字查询需要的数据

   ```sql
   SELECT  t.*,'${tableName}' AS tbname  FROM  ${tableName}  t  WHERE  DATE(t.`LAST_MODIFICATION_TIME`) < '2023-09-22'
   ```

4. 将数据全部放入目标表

   执行聚合操作等流程

## 三、实际流程

![涉及的作业与转换](https://raw.githubusercontent.com/li123sai/myPictures/main/img/q1.png)

------

![作业流程](https://raw.githubusercontent.com/li123sai/myPictures/main/img/q2.png)

------

![获取全部schema](https://raw.githubusercontent.com/li123sai/myPictures/main/img/q3.png)

------

![获取schema下表名字](https://raw.githubusercontent.com/li123sai/myPictures/main/img/q4.png)

------

![将表名字设置成变量](https://raw.githubusercontent.com/li123sai/myPictures/main/img/q5.png)

当前转换设置变量需要在子转换或者同作业调用链转换使用。（这是一个巨坑）

------

![使用动态表名字获取数据](https://raw.githubusercontent.com/li123sai/myPictures/main/img/q6.png)
