---
title: 深入浅出 MySQl（一）
date: 2019-09-05 11:32:14
tags:
  - mysql
  - note
  - reading
  - 深入浅出 MySQL
categories: 读书笔记
---

<!--more-->

# 1. 帮助的使用

## 1.1 按层次看帮助

```sql
mysql> ? contents
You asked for help about help category: "Contents"
For more information, type 'help <item>', where <item> is one of the following
categories:
   Account Management
   Administration
   Compound Statements
   Data Definition
   Data Manipulation
   Data Types
   Functions
   Functions and Modifiers for Use with GROUP BY
   Geographic Features
   Help Metadata
   Language Structure
   Plugins
   Procedures
   Storage Engines
   Table Maintenance
   Transactions
   User-Defined Functions
   Utility
```

对于列出的分类，可以进行看自己感兴趣的部分，比如：

```sql
mysql> ? data types
You asked for help about help category: "Data Types"
For more information, type 'help <item>', where <item> is one of the following
topics:
   AUTO_INCREMENT
   BIGINT
   BINARY
   BIT
   BLOB
   BLOB DATA TYPE
   BOOLEAN
......
```

对于列出的具体数据类型，可以进一步查看过情况：

```sql
mysql> ? int
Name: 'INT'
Description:
INT[(M)] [UNSIGNED] [ZEROFILL]

A normal-size integer. The signed range is -2147483648 to 2147483647.
The unsigned range is 0 to 4294967295.
```

## 1.2 快速查阅帮助

需要快速查阅某项语法时，可以使用关键字快速查阅：

```sql
mysql> ? show
Name: 'SHOW'
Description:
SHOW has many forms that provide information about databases, tables,
columns, or status information about the server. This section describes
those following:

SHOW AUTHORS
SHOW {BINARY | MASTER} LOGS
SHOW BINLOG EVENTS [IN 'log_name'] [FROM pos] [LIMIT [offset,] row_count]
SHOW CHARACTER SET [like_or_where]
SHOW COLLATION [like_or_where]
SHOW [FULL] COLUMNS FROM tbl_name [FROM db_name] [like_or_where]
SHOW CONTRIBUTORS
SHOW CREATE DATABASE db_name
SHOW CREATE EVENT event_name
SHOW CREATE FUNCTION func_name
SHOW CREATE PROCEDURE proc_name
SHOW CREATE TABLE tbl_name
SHOW CREATE TRIGGER trigger_name
SHOW CREATE VIEW view_name
......
```

我想知道 `create table` 的语法，可以命令如下：

```sql
mysql> ? create table
Name: 'CREATE TABLE'
Description:
Syntax:
CREATE [TEMPORARY] TABLE [IF NOT EXISTS] tbl_name
    (create_definition,...)
    [table_options]
    [partition_options]

CREATE [TEMPORARY] TABLE [IF NOT EXISTS] tbl_name
    [(create_definition,...)]
    [table_options]
    [partition_options]
    [IGNORE | REPLACE]
    [AS] query_expression

CREATE [TEMPORARY] TABLE [IF NOT EXISTS] tbl_name
    { LIKE old_tbl_name | (LIKE old_tbl_name) }

create_definition:
    col_name column_definition
  | [CONSTRAINT [symbol]] PRIMARY KEY [index_type] (index_col_name,...)
      [index_option] ...
  | {INDEX|KEY} [index_name] [index_type] (index_col_name,...)
      [index_option] ...
  | [CONSTRAINT [symbol]] UNIQUE [INDEX|KEY]
      [index_name] [index_type] (index_col_name,...)
      [index_option] ...
  | {FULLTEXT|SPATIAL} [INDEX|KEY] [index_name] (index_col_name,...)
      [index_option] ...
  | [CONSTRAINT [symbol]] FOREIGN KEY
      [index_name] (index_col_name,...) reference_definition
  | CHECK (expr)

column_definition:
    data_type [NOT NULL | NULL] [DEFAULT default_value]
      [AUTO_INCREMENT] [UNIQUE [KEY] | [PRIMARY] KEY]
      [COMMENT 'string']
      [COLUMN_FORMAT {FIXED|DYNAMIC|DEFAULT}]
      [STORAGE {DISK|MEMORY|DEFAULT}]
      [reference_definition]

data_type:
    BIT[(length)]
  | TINYINT[(length)] [UNSIGNED] [ZEROFILL]
  | SMALLINT[(length)] [UNSIGNED] [ZEROFILL]
  | MEDIUMINT[(length)] [UNSIGNED] [ZEROFILL]
  | INT[(length)] [UNSIGNED] [ZEROFILL]
  | INTEGER[(length)] [UNSIGNED] [ZEROFILL]
  | BIGINT[(length)] [UNSIGNED] [ZEROFILL]
  | REAL[(length,decimals)] [UNSIGNED] [ZEROFILL]
  | DOUBLE[(length,decimals)] [UNSIGNED] [ZEROFILL]
  | FLOAT[(length,decimals)] [UNSIGNED] [ZEROFILL]
  | DECIMAL[(length[,decimals])] [UNSIGNED] [ZEROFILL]
  | NUMERIC[(length[,decimals])] [UNSIGNED] [ZEROFILL]
  | DATE
  | TIME[(fsp)]
  | TIMESTAMP[(fsp)]
......
```

# 2. 表类型（存储引擎）的选择

## 2.1 Mysql 存储引擎概述

`mysql` 支持多种存储引擎，在处理不同类型的应用时，可以通过选择使用不同的存储引擎提高应用的效率，或者提供灵活的存储。

`mysql` 的存储引擎包括：`MyISAM`、`InnoDB`、`BDB`、`MEMORY`、`EXAMPLE`、`NDB Cluster`、`ARCHIVE`、`CSV`、`BLACKHOLE`、`FEDERATED` 等，其中 `InnoDB` 和 `BDB` 提供事务安全表，其他存储引擎都是非事务安全表。

## 2.2 各种存储引擎的特性

| 特点         | MyISAM | BDB  | Memory | InnoDB | Archive |
| ------------ | :----: | :--: | :----: | :----: | :-----: |
| 存储限制     |   无   |  无  |   有   |  64TB  |  没有   |
| 事务安全     |        | 支持 |        |  支持  |         |
| 锁机制       |  表锁  | 页锁 |  表锁  |  行锁  |  行锁   |
| B 树索引     |  支持  | 支持 |  支持  |  支持  |         |
| 哈希索引     |        |      |  支持  |  支持  |         |
| 全文索引     |  支持  |      |        |        |         |
| 集群索引     |        |      |        |  支持  |         |
| 数据缓存     |        |      |  支持  |  支持  |         |
| 索引缓存     |  支持  |      |  支持  |  支持  |         |
| 数据可压缩   |  支持  |      |        |        |  支持   |
| 空间使用     |   低   |  低  |  N/A   |   高   |   低    |
| 内存使用     |   低   |  低  |  中等  |   高   |   低    |
| 批量插入速度 |   高   |  高  |   高   |   低   | 非常高  |
| 支持外键     |        |      |        |  支持  |         |

**最常用的两种引擎**：

- `MyISAM` 是 `MySQL` 默认存储引擎（5.5 之前），每个 `MyISAM` 在磁盘上存储三个文件。文件名和表名相同，扩展名分别是 `.frm`（存储表定义）、`.MYD`（MyData，存储数据）、`.MYI`（MyIndex，存储索引）。数据和文件和索引文件可以放置在不同的目录，平均分布 io，获得更快的速度
- `InnoDb` 是 `MySQL` 默认存储引擎（5.5 之后），`InnoDB` 存储引擎提供了具有提交、回滚和崩溃恢复的事务安全。但是对比 `MyISAM` 的存储引擎，`InnoDB` 写的效率会差一些并且会占用更多的磁盘空间以保留数据和索引

## 2.3 选择合适的存储引擎

- `MyISAM`：在 Web、数据仓储和其他应用环境下
- `InnoDB`：用于事务处理应用程序，具有众多特性，包括 ACID 事务支持
- `Memory`：将所有数据保存在 `RAM` 中，在需要快速查找引用和其他类似数据的环境下，可以提供极快的访问
- `Merge`：允许 `MySQL DBA` 或开发人员将一系列等同的 `MyISAM` 表以逻辑方式组合在一起，并作为一个对象引用他们。对于诸如数据仓储等 `VLDB` 环境十分适合

