#1.rocksdb_get

###对外接口
	RocksDB 对外提供了Get(key), Put(key), Delete(key) and NewIterator()等操作，我们可以直接从官方的文档里面找到测试程序，以此为入口开始代码分析（examples/c_simple_example.c）。
	
	代码中的rocksdb_put、rocksdb_get作为全局函数调用DB::put/DB:get接口，本质是调用的数据库实现类DBImpl的操作。
	
	写入过程中key和value被打包成slice对象传入put接口，写入WriteBatch，后续将批量写入Memtable。
	
	Slice由一个size变量和一个指向外部内存区域的指针构成。使用slice可以减少之后的key、value在传值过程中的拷贝操作。
	
	而在读的过程中，引入了PinnabeSlice作为引用出参，同样是减少内存拷贝，并做到了生命周期控制，防止block_cache中的数据在返回前被释放，最终get不到数据的问题。（BlockIter退出前通过DelegateCleanUpsTo将cleanup交给PinnabeSlice）		



```cpp
main//c_simple_example
--rocksdb_options_create
--rocksdb_open
--rocksdb_readoptions_create
--rocksdb_get(db, readoptions, key, strlen(key), &len, &err)
----db->rep->Get(options->rep, Slice(key, keylen), &tmp);
------DB::Get
--------DBImpl::Get
----------DBImpl::GetImpl
------------if (!skip_memtable)
--------------MemTable::Get//sv->mem->Get
--------------sv->imm->Get
------------else
--------------sv->current->Get
--rocksdb_close(db)
```