# 2 MyRocks 数据字典

## 2.1 元信息

*   **MyRocks 元信息包含以下几类**

```java
enum DATA_DICT_TYPE {
    DDL_ENTRY_INDEX_START_NUMBER= 1, // 表和索引之间的映射关系
    INDEX_INFO=                   2, // 索引id和索引属性的关系
    CF_DEFINITION=                3, // CF 属性
    BINLOG_INFO_INDEX_NUMBER=     4, // binlog位点及gtid信息，binlog_commit更新此信息
    DDL_DROP_INDEX_ONGOING=       5, // 等待删除的索引信息
    INDEX_STATISTICS=             6, // 索引统计信息
    MAX_INDEX_ID=                 7, // 当前的index id，每次创建索引index id都从这个获取和更新
    DDL_CREATE_INDEX_ONGOING=     8, // 等待创建的索引信息
    END_DICT_INDEX_ID=          255
};
```

## 2.2 创建一张表

### 2.2.1 建表语句

```java
mysql> show create table t1\G
*************************** 1. row ***************************
       Table: t1
Create Table: CREATE TABLE `t1` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `a` int(11) DEFAULT NULL,
  `b` char(8) COLLATE latin1_bin DEFAULT NULL,
  PRIMARY KEY (`id`),
  KEY `idx1` (`b`) COMMENT 'cf_b'
) ENGINE=ROCKSDB DEFAULT CHARSET=latin1 COLLATE=latin1_bin
1 row in set (0.00 sec)
```

### 2.2.2 建表流程

![](assets/1599739636-fc1c3061b56bd2f8326d0a98a39ad0c4.png)

## 2.3 元信息存储

*   元信息以 key-value 形式保存在 \_\_system\_\_ CF 中
*   各个步骤与记录的对应关系如下  
    *   CF\_DEFINITION：定义新的 CF，并分配 cf\_id
        
    *   INDEX\_INFO：创建索引信息，索引所属 CF ID、索引 ID、索引类型（主键索引、辅助索引、隐藏索引）
        
    *   DDL\_ENTRY\_INDEX\_START\_NUMBER：dbname+tblname 与 索引的映射
        

![](assets/1599739636-8ef3252da2a37e93f3359306d5368b12.png)

## 2.4 元信息查询

*   创建 CF cf\_b：用于保存辅助索引 idx1 的数据

```java
mysql> SELECT * FROM INFORMATION_SCHEMA.ROCKSDB_GLOBAL_INFO;
+--------------+--------------+--------------------------------+
| TYPE         | NAME         | VALUE                          |
+--------------+--------------+--------------------------------+
| BINLOG       | FILE         | cq01-sys-replace001-bin.000011 |
| BINLOG       | POS          | 2517                           |
| BINLOG       | GTID         |                                |
| MAX_INDEX_ID | MAX_INDEX_ID | 265                            |
| CF_FLAGS     | 0            | default [0]                    |
| CF_FLAGS     | 1            | __system__ [0]                 |
| CF_FLAGS     | 2            | pk_cf [0]                      |
| CF_FLAGS     | 3            | idx_cf [0]                     |
| CF_FLAGS     | 4            | cf_b [0]                       |
+--------------+--------------+--------------------------------+
9 rows in set (0.00 sec)
```

  

*   创建两个索引
    *   主键索引：PRIMARY，索引 ID : 264，由 CF default 管理，类型为1
    *   辅助索引：idx1，索引 ID : 265，由 CF cf\_b 管理，类型为2

```java
mysql> SELECT INDEX_NAME,INDEX_NUMBER,COLUMN_FAMILY,CF,INDEX_TYPE,KV_FORMAT_VERSION FROM INFORMATION_SCHEMA.rocksdb_ddl where TABLE_NAME='t1';
+------------+--------------+---------------+---------+------------+-------------------+
| INDEX_NAME | INDEX_NUMBER | COLUMN_FAMILY | CF      | INDEX_TYPE | KV_FORMAT_VERSION |
+------------+--------------+---------------+---------+------------+-------------------+
| PRIMARY    |          264 |             0 | default |          1 |                13 |
| idx1       |          265 |             4 | cf_b    |          2 |                13 |
+------------+--------------+---------------+---------+------------+-------------------+
2 rows in set (0.00 sec)
```

  

*   建立 dbname 与 tablename 的映射关系

```java
mysql> SELECT TABLE_SCHEMA,TABLE_NAME,INDEX_NAME,INDEX_NUMBER,COLUMN_FAMILY,CF,INDEX_TYPE,KV_FORMAT_VERSION FROM INFORMATION_SCHEMA.rocksdb_ddl where TABLE_NAME='t1';
+--------------+------------+------------+--------------+---------------+---------+------------+-------------------+
| TABLE_SCHEMA | TABLE_NAME | INDEX_NAME | INDEX_NUMBER | COLUMN_FAMILY | CF      | INDEX_TYPE | KV_FORMAT_VERSION |
+--------------+------------+------------+--------------+---------------+---------+------------+-------------------+
| test         | t1         | PRIMARY    |          264 |             0 | default |          1 |                13 |
| test         | t1         | idx1       |          265 |             4 | cf_b    |          2 |                13 |
+--------------+------------+------------+--------------+---------------+---------+------------+-------------------+
2 rows in set (0.00 sec)
```

