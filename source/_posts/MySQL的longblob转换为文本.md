---
title: MySQL的longblob转换为文本
date: 2023-07-28 18:53:27
categories:
- 数据库
- Mysql
tags:
- 函数的使用
---

## 一、背景

```
	MySQL的longblob中存了一些图片，在做数据迁移时想基于动态SQL将数据迁移至新库。
```

## 二、longblob转换为文本

下面是三个网上比较多的转换方法：

1. 使用[UTF-8](http://en.wikipedia.org/wiki/UTF-8)编码将longblob转换为char(1000)的file_data示例

   ```
   CAST(t.file_data AS CHAR(10000) CHARACTER SET utf8)
   ```

   不好使

2. 使用MySQL 的 `CONVERT()` 函数

   ```sql
   CONVERT(t.file_data USING utf8)
   ```

   也不好使

3. 使用UTF16

   ```sql
   SELECT
       t.file_data,
       CONVERT(t.file_data USING utf16),
       CONVERT(CONVERT(t.file_data USING utf16), BINARY),
       CAST(t.file_data  AS CHAR(10000) CHARACTER SET utf16),
       CAST(CAST(t.file_data  AS CHAR(10000) CHARACTER SET utf16) AS BINARY)
   ```

   完全不好使

​	

### 	实际使用：

```sql
SELECT
    t.file_data,
    TO_BASE64(t.file_data),
    FROM_BASE64(TO_BASE64(t.file_data))
```

很好用，显示了正常的文本值。
