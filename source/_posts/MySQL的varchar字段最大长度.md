---
title: MySQL的varchar字段最大长度
date: 2023-09-26 16:46:44
categories:
- 数据库
- Mysql
tags:
- 字段长度
---

## 一、背景

系统中一个日志表报了字段超长的错误，

```sh
Data too long for colum '' at row 1
```

于是给字段增加长度,接着另一个关联字段也报了超长错误，在给关联字段增加长度时，虽然没有超过varchar的最大长度65535，但是Mysql报错了。一直说varchar字段的最大长度是65535，实际却有出入，为了弄清到底是那的问题，做了以下复现。

## 二、场景复现

1. 创建模拟表

   ```sql
   CREATE TABLE `test_log`
   (
       `id`             VARCHAR(50)  	NOT NULL COMMENT 'test',
       `ip`             VARCHAR(60)    DEFAULT NULL COMMENT 'test',
       `app_name`       VARCHAR(128)   DEFAULT NULL COMMENT 'test',
       `it_id`        	 VARCHAR(1024)  NOT NULL DEFAULT '' COMMENT 'test',
       `it_code`      	 VARCHAR(1024)  NOT NULL DEFAULT '' COMMENT 'test',
       `it_name`	     VARCHAR(1024)  NOT NULL DEFAULT '' COMMENT 'test',
       `create_time`    TIMESTAMP    	NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间',
       `update_time`    TIMESTAMP    	NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP 		     COMMENT '更新时间',
       PRIMARY KEY (`ID`)
   ) ENGINE = INNODB DEFAULT CHARSET=utf8mb4 
   ```

2. 增加字段长度

   ```sql
   ALTER TABLE test_log MODIFY  `it_name` VARCHAR(23000);
   ```

3. 修改长度为23000

   ![修改字段长度报错信息](https://raw.githubusercontent.com/li123sai/myPictures/main/img/m1.png)

   为什么最大长度是16383 而不是 65535？

   VARCHAR(n) 中的 n 表示该列能够存储的最大字符数，而不是字节数。MySQL 会根据使用的字符集来计算所需的字节数。

   ```sql
    SHOW CHARSET LIKE 'utf8mb4';
   ```

   可以看到

   ![字符集信息](https://raw.githubusercontent.com/li123sai/myPictures/main/img/m3.png)

   - Charset：utf8mb4，指的是一种 UTF-8 编码格式的字符集，支持 4 字节的 Unicode 字符。(关于utf8和utf8mb4的区别：
     MySQL在 5.5.3 之后增加了 utf8mb4 字符编码，mb4即 most bytes 4。简单说 utf8mb4 是 utf8 的超集并完全兼容utf8，能够用四个字节存储更多的字符。)

   - Description：UTF-8 Unicode，表示该字符集是基于 Unicode 编码标准的 UTF-8 实现，可以支持世界上大部分语言的字符编码需求。

   - Default collation：utf8mb4_general_ci，表示该字符集默认使用的排序规则为 utf8mb4_general_ci，即不区分大小写的通用排序规则，可以满足大多数排序需求。

   - Maxlen：4，表示该字符集表示一个字符的最大长度（Byte字节数）。

     

     utf8mb4的maxlen=4，对应varchar最大长度=16383。65535/4=16383.75  ≈  16384

4. 修改长度为16383

   ```sql
   ALTER TABLE test_log MODIFY `it_name` VARCHAR(16383)
   ```

   报错了

   ```sh
   错误代码： 1118
   Row size too large. The maximum row size for the used table type, not counting BLOBs, is 65535. This includes storage overhead, check the manual. You have to change some columns to TEXT or BLOBs
   
   执行耗时   : 0 sec
   传送时间   : 0 sec
   总耗时    : 0.002 sec
   ```

   InnoDB 存储引擎的最大行大小是 65,535 字节（不包括 BLOB 和 TEXT 列）。需要考虑以下因素

   - 列的数据类型：数据类型决定了每列所占用的存储空间。例如，INT 类型占用 4 字节，VARCHAR 类型占用实际存储的字节数加上额外的长度前缀等。

   - 行格式：不同的行格式对行的存储方式和限制有所不同。例如，InnoDB 存储引擎可以使用 COMPACT、REDUNDANT 和 DYNAMIC 等不同的行格式。

   - 行的存储属性：有些存储引擎允许为表或列设置存储属性，如压缩或加密，这些属性可能会增加行的存储开销。

   - 存储引擎特定的限制：每个存储引擎可能还有其他特定的限制，如 InnoDB 的索引大小限制等。

     

     所以mysql表里单行中的所有列加起来（不考虑其他隐藏列和记录头信息） ，占用的最大长度是65535个字节。
     比如还有int的列，那它占用4个字节，字段越多，留给单个varchar列的空间就越少。

     根据表里面现有字段，能扩的最大长度是：
     16383-50-60-128-1024-1024=14097

5. 修改长度为14097

   ```sql
   ALTER TABLE test_log MODIFY  `it_name` VARCHAR(14097);
   ```

   报错了

   ```sh
   错误代码： 1118
   Row size too large. The maximum row size for the used table type, not counting BLOBs, is 65535. This includes storage overhead, check the manual. You have to change some columns to TEXT or BLOBs
   
   执行耗时   : 0 sec
   传送时间   : 0 sec
   总耗时      : 0.003 sec
   ```

   为什么已经减去其他列的长度还是有问题？

   ```sql
   SHOW TABLE STATUS LIKE 'test_log';
   ```

   ![查询表基础信息](https://raw.githubusercontent.com/li123sai/myPictures/main/img/m2.png)

   通过上面的 Row_format 字段可以看到这个表用的是 Dynamic 行格式。
   Dynamic格式将行记录分为两部分，分为是行记录的额外信息和行记录的真实数据。

   在行记录的额外信息中，对于 VARCHAR 列，其占用的字节数主要包括长度信息和实际数据。具体占用的字节数取决于编码方式和存储引擎的实现。
   对于 InnoDB 存储引擎，VARCHAR 列的长度信息一般占用 1-2 个字节。这些字节用于存储字段的长度值，在读取时用于解析字段的实际数据。
   实际数据部分会占用 VARCHAR 字段定义的长度。例如，如果 VARCHAR 定义的最大长度是 50 字符，那么实际存储的数据将占用实际字符长度 + 长度信息所占用的字节数，即 50 + 1-2 个字节。
   需要注意的是，对于多字节字符集（如 UTF-8），单个字符可能占用多个字节。因此，如果使用多字节字符集存储 VARCHAR 数据，实际占用的字节数将根据字符集的编码方式而变化。
   同时
   对于多个 VARCHAR 字段，每个字段的 NULL 和非 NULL 值的额外信息都会分别占用一定字节数。
   NULL 值：对于每个 NULL 值的 VARCHAR 字段，额外信息会占用 1 个字节来表示该列的值为 NULL。没有实际存储数据，只有 1 个字节的 NULL 标记。
   非 NULL 值：对于每个非 NULL 值的 VARCHAR 字段，在额外信息中会分别占用一定的字节数。
   长度信息：长度信息用于记录每个 VARCHAR 字段的实际字符长度，一般占用 1-2 个字节。每个字段都需要记录长度信息。
   实际数据：每个字段的实际数据部分将占用各自字段定义的长度。具体占用的字节数取决于编码方式和字段定义的长度。

   

   综上所述，行记录的额外信息中，VARCHAR 列的字节数大致等于字段长度信息的字节数（1-2 字节）加上实际存储数据的字节数（根据编码方式和字段定义的长度确定）

6. 修改长度为14093

   ```sql
   ALTER TABLE test_log MODIFY  `it_name` VARCHAR(14093);
   ```

   成功了

   ```sh
   1 queries executed, 1 success, 0 errors, 0 warnings
   
   查询：ALTER TABLE test_log MODIFY `it_name` VARCHAR(14093)
   
   共 0 行受到影响
   
   执行耗时   : 0.020 sec
   传送时间   : 1.014 sec
   总耗时      : 1.034 sec
   ```

## 三、总结

- 现在的mysql数据表一般采用Dynamic行记录格式。它由行记录的额外信息和行记录的真实数据组成。
- mysql表里单行中的所有列加起来（不考虑其他隐藏列和记录头信息） ，占用的最大长度是65535个字节。
- 如果数据表里有多列null 和 not null的varchar字段，新增或修改varchar字段最大长度：综合全部字段长度 + 行记录的额外信息字节数。
