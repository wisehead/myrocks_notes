#1.Config

如果你想在相同的实例中同时使用InnoDB存储引擎和MyRocks存储引擎,需要在my.cnf中设置allow-multiple-engines参数 以及 删除skip-innodb参数。 使用混合存储引擎不建议在生产环境下使用。因为它不是真正的事务型，但是它可以用户实验的目的。 复制模式下，Slave节点的binlog记录是Statement模式是被允许的，但不能是Master。这是因为MyRocks 不支持next-key锁。

MyRocks 的数据存储在RocksDB中，按指数计算，RocksDB 在内部分配一个 列族 来存储索引数据。 默认情况下，所有的数据存储在 default 列族中。你可以通过为给定索引设置 COMMENT 来更改列族。 在上面的例子中，主键存储在cf_link_pk列族中，id1_type索引数据存储在rev:cf_linke_id1_type列族中。

MyRocks还有一个叫作 反向列族 的功能。如果索引主要用于降序扫描(ORDER BY … DESC),你可以通过在列族名称之前设置 rev: 来配置反向列族，在此示例中,id1_type属于反向列族。
对于给定的table表，MyRocks支持基于每个分区存储的列族，有关如何指定列族的更多信息，请参考 (四、性能调优/7.分区表列族.md)



