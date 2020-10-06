#1.Version::Get

在sstable中搜索
如果在内存中没有得到结果，那么下一步就是从sstable中找寻了。但是这一步并不是马上会产生I/O,因为rocksdb/leveldb都做了一定程度的cache（table cache/block cache），只有cache中没有的时候才回去读磁盘。


```cpp
Version::Get
--GetContext get_context()
--pinned_iters_mgr.StartPinning();
--FilePicker fp()
--FdWithKeyRange* f = fp.GetNextFile()
----while (!search_ended_) {  // Loops over different levels.
      while (curr_index_in_curr_level_ < curr_file_level_->num_files) {
------if (num_levels_ > 1 || curr_file_level_->num_files > 3)
--------fractional cascading优化寻找SST的方法 FileIndexer.
------else
--------层内顺序访问sst文件（已经排序）
--------PrepareNextLevel（Binary Search决定候选文件）

--while (f != nullptr) 
----*status = table_cache_->Get
----switch (get_context.State())
------case GetContext::kNotFound:
				break；
------case GetContext::kMerge:
				break;
------case GetContext::kFound:
				return//stop while
------case GetContext::kDeleted:
        // Use empty error message for speed
        *status = Status::NotFound();
        return;
      case GetContext::kCorrupt:
        *status = Status::Corruption("corrupted key for ", user_key);
        return;
      case GetContext::kBlobIndex:
        ROCKS_LOG_ERROR(info_log_, "Encounter unexpected blob index.");
        *status = Status::NotSupported
----f = fp.GetNextFile();

--if (GetContext::kMerge == get_context.State())//begin the merge
----*status = MergeHelper::TimedFullMerge
----if (LIKELY(value != nullptr))
------value->PinSelf()
----else
------*status = Status::NotFound(); // Use an empty error message for speed



```

这里面有两个函数值得注意，一个是GetNextFile（）明确了我们每一次去读哪个文件，按什么顺序读。另一个是TableCache::Get明确了每个文件我们怎么读里面的内容。

在GetNextFile中，RocksDB实现了一个基于FileIndexer得fractional cascading优化寻找sst算法。
常规的办法是通过两层循环先在每层内全局二分搜索，然后从0层循环到最高层。
通过维护FileIndexer，对于第1层以上的层且层文件数多于3个时，可以根据之前上一层读过的最后一个文件的key-range和要搜索的key值，对本层要读的sstable文件进行范围收紧。
这里有最多五种不同情况，每一种都可以至少确定一个right_bound或者left_bound。
