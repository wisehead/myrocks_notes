#1.TableCache::Get


```cpp
TableCache::Get
--//可能有优化
--// Check row cache if enabled. Since row cache does not currently store
  // sequence numbers, we cannot use it if we need to fetch the sequence.
  if (ioptions_.row_cache && !get_context->NeedToReadSequence()) {      
--TableCache::FindTable
----number = fd.GetNumber();
----key = GetSliceForFileNumber(&number);
----*handle = cache_->Lookup(key);//根据fd查找内存中是否有sst对应的table
----if (*handle == nullptr)
------GetTableReader
--------//ioptions_.table_factory->NewTableReader
--------BlockBasedTableFactory::NewTableReader
----------BlockBasedTable::Open
----cache_->Insert//将新建的table插入cache，以备下次读。
----return s;//返回table对象，table在cache中，通过handle拿

--t = GetTableReaderFromHandle(handle)
--std::unique_ptr<InternalIterator> range_del_iter(t->NewRangeTombstoneIterator(options));
--get_context->range_del_agg()->AddTombstones(std::move(range_del_iter))
--BlockBasedTable::Get//t->Get(options, k, get_context, skip_filters)
----filter_entry = GetFilter
----if (!FullFilterKeyMayMatch)
				return
----if (FullFilterKeyMayMatch)
------iiter = NewIndexIterator
------for (iiter->Seek(key); iiter->Valid() && !done; iiter->Next()) 
--------handle_value = iiter->value();
--------bool not_exist_in_filter =
          filter != nullptr && filter->IsBlockBased() == true &&
          handle.DecodeFrom(&handle_value).ok() &&
          !filter->KeyMayMatch(ExtractUserKey(key), handle.offset(), no_io);
--------if (not_exist_in_filter)
					break;
--------else exist
----------BlockBasedTable::NewDataBlockIterator//注意在这里把数据load到
------------MaybeLoadDataBlockToCache
------------ReadBlockFromFile
----------for (biter.Seek(key); biter.Valid(); biter.Next())
------------get_context->SaveValue(parsed_key, biter.value(), &biter))

```

其中的FindTable实现从缓存中去拿选出的sstable文件的基本元信息，如果缓存里面没有就需要先创建（BlockBasedTable::Open）一个，同时还要插入缓存（cache_->Insert）。
Open涉及的内容很多，主要是通过读footer在TableReader对象里面缓存meta block和index block里面的信息。
然后就是BlockBasedTable::Get去根据table拿到key-value了。首先考虑拿过滤器判断想搜索的内容是否可能存在。
接着用NewIndexIterator去seek值被包括的块，优先尝试从cache中读这个快，然后再考虑用过滤器判断值是否在块内，
最后用 NewDataBlockIterator去seek快内的值。最后SaveValue按照key-value的类型决定返回还是继续搜索。


读流程逻辑步骤：
	-- 找到一个可能含有Key的文件（sst fraction cascading）
	-- 从TableCache中查找Table Reader
	-- 使用FullFilter对文件进行过滤
	-- 如果包含，使用Index定位block位置
	-- 使用BlockFilter对Block进行过滤
	-- 在DataBlock中搜索记录。