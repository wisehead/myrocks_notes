#1.TTL

生存时间TTL
对于某些工作负载，在一段时间后需要删除行。通常的方法是创建一个时间列，并定期删除以删除旧行 这并不理想，因为删除会导致MyRocks中出现许多问题。我们必须消耗CPU成本去处理delete语句。 以及将删除标记写入数据库的IO成本。删除标记也可以使扫描速度变慢。
相反，我们可以利用压缩过滤器来执行删除操作。
DDL语法
TTL 是通过表注释定义的，有两个变体，隐式和显式时间戳

```sql
CREATE TABLE t1 (a INT, b INT, c INT, PRIMARY KEY (a), KEY(b)) ENGINE=ROCKSDB COMMENT "ttl_duration=3600;";

CREATE TABLE t2 (a INT, b INT, c INT, ts BIGINT UNSIGNED NOT NULL, PRIMARY KEY (a), KEY(b)) ENGINE=ROCKSDB COMMENT "ttl_duration=3600;ttl_col=ts;";
```

Read读过滤
对于可重复读隔离级别，或者如果在具有辅助键的表上定义TTL，则必须打开读取过滤。 这可以通过设置rocksdb_enable_ttl_read_filtering变量。(默认情况下已启用)来完成。


在上面的示例中，我们将ttl_duration设置为3600意味着我们希望从数据库中删除超过3600秒的行。
t1表模式使用隐式时间戳，这意味着创建的时间呗隐式记录为插入或更新记录时的当前时间。记录的时间戳存储在记录中，但对用户不可见。 t2表模式使用显式时间戳，这由表注释中的ttl_col限定符表示。这意味着用于TTL的创建时间直接来自ts列。



