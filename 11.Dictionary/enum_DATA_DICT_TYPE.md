#1.enum DATA_DICT_TYPE

```cpp

enum DATA_DICT_TYPE {
    DDL_ENTRY_INDEX_START_NUMBER= 1, // 表和索引之间的映射关系
    INDEX_INFO=                   2, // 索引id和索引属性的关系
    CF_DEFINITION=                3, // CF 属性
    BINLOG_INFO_INDEX_NUMBER=     4, // binlog位点及gtid信息，binlog_commit更新此信息
    DDL_DROP_INDEX_ONGOING=       5, // 等待删除的索引信息
    INDEX_STATISTICS=             6, // 索引统计信息
    MAX_INDEX_ID=                 7, // 当前的index id，每次创建索引index id都从这个获取和更新
    DDL_CREATE_INDEX_ONGOING=     8, // 等待创建的索引信息
    END_DICT_INDEX_ID=          255
};
```