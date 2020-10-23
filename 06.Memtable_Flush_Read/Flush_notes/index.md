
# 3 Flush

Minor Compaction：将内存中Immutable Memtable转储到磁盘SSTable。

Major Compaction：磁盘上的SSTable文件从低层向高层转储。

![](assets/1599735083-605ef45264358719c2a6fef49fc5d479.png)

执行 flush 操作的线程数可配置（rocksdb\_max\_background\_flushes）。

入口函数为：FlushMemTable()。

level 0 存在冗余数据，level 1 ~ level n 不存在数据冗余。

## 3.1 flush 触发机制

Immutable memtable 的数量是否超过了 min\_write\_buffer\_number\_to\_merge。

【构造 flush 任务】

![](assets/1599735083-a661d3a5ad30776e7b95a01a5335d799.png)

【任务执行流程】

![](assets/1599735083-c49da31b58abf43f5f9356792b02b92e.png)

流程如下：

1.  遍历 immutable-list，如果没有其它线程正在执行 flush，则将 flush 任务加入队列
2.  通过迭代器逐一扫描Immutable memtable 中的 key-value，将 key-value 写入到 data-block 
3.  如果 data block 大小已经超过 block\_size （比如16k），或者是最后一对 key-value，则触发一次 block-flush
4.  根据压缩算法对 block 进行压缩，并生成对应的 index block 记录（begin\_key, last\_key, offset）
5.  至此若干个 block 已经写入文件，并为每个 block 生成了 index-block 记录
6.  写入 index-block，meta block，metaindex block 以及 footer 信息到文件尾
7.  将 SST 文件的元信息写入 manifest 文件

flush 的过程是将 immutable table 写入 level 0，level 0 各个 SST 内部没有冗余数据，但 SST 之间会有 Key 的交叠。

## 3.2 flush 的调用点

调用 FlushMemtable 的原因如下

```java
enum class FlushReason : int {
  kOthers = 0x00,
  kGetLiveFiles = 0x01,
  kShutDown = 0x02,
  kExternalFileIngestion = 0x03,
  kManualCompaction = 0x04,
  kWriteBufferManager = 0x05,
  kWriteBufferFull = 0x06,
  kTest = 0x07,
  kDeleteFiles = 0x08,
  kAutoCompaction = 0x09,
  kManualFlush = 0x0a,
};
```

【checkpoint】

checkpoint 时会调用 flush，但是条件无法达成

![](assets/1599735083-34cda6ae8edfa5c5a7e02414b7cd3633.png)

