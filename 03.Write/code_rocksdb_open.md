#1. rocksdb_put

###对外接口
RocksDB 对外提供了Get(key), Put(key), Delete(key) and NewIterator()等操作，我们可以直接从官方的文档里面找到测试程序，以此为入口开始代码分析（examples/c_simple_example.c）。


代码中的rocksdb_put、rocksdb_get作为全局函数调用DB::put/DB:get接口，本质是调用的数据库实现类DBImpl的操作。
其中key和value被打包成slice对象传入put接口，写入WriteBatch::rep_(要么都提交，要么都回滚)，后续将批量写入Memtable。

Slice由一个size变量和一个指向外部内存区域的指针构成。使用slice可以减少之后的key、value在传值过程中的拷贝操作。
writeoptions是写的配置信息，明确是否需要写日志以及是否需要关闭异步写等参数选项。


```cpp
c_simple_example
--rocksdb_open
----DB::Open(options->rep, std::string(name), &db)
------DBImpl::Open
--rocksdb_writeoptions_create
--rocksdb_put
----db->rep->Put(options->rep, Slice(key, keylen), Slice(val, vallen)
/*
  // Pre-allocate size of write batch conservatively.
  // 8 bytes are taken by header, 4 bytes for count, 1 byte for type,
  // and we allocate 11 extra bytes for key length, as well as value length.
*/
------DB::Put
--------WriteBatch::Put
----------WriteBatchInternal::Put
------------PutLengthPrefixedSlice(&b->rep_, key);
------------PutLengthPrefixedSlice(&b->rep_, value);
--------DBImpl::Write
----------DBImpl::WriteImpl
```