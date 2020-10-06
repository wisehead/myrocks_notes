#1.MemTable::Get

在内存中搜索

在Memtable和Immutable memtable中搜索调用的函数是一致的。其默认实现是基于跳表完成的。
跳表迭代器SkipListRep::Iterator iter通过FindGreaterOrEqual实现搜索定位，并根据获得目标对象的key决定是继续搜索还是返回结果。

值得注意的是，rocksdb引入了范围删除和Merge等运算操作，所以在值类型判断的时候要比leveldb复杂的多。
具体流程如下：
	1.在Memtable中找到一个大于等于该值的节点
	2.根据不同情况决定是否向下执行：
			--User Key不等，直接失败。-------------->查找失败，Merge Context被传到下一级查找。。
			--User Key相等，查找成功
			--类型为Merge，将Merge内容记入Merge Context，向下查找。
			

```cpp

MemTable::Get
--std::unique_ptr<InternalIterator> range_del_iter(NewRangeTombstoneIterator(read_opts));
--range_del_agg->AddTombstones(std::move(range_del_iter));
--prefix_bloom_->MayContain(prefix_extractor_->Transform(user_key));
--MemTableRep::Get//table_->Get
----MemTableRep::GetDynamicPrefixIterator
------GetIterator
--------SkipListRep::Iterator(&skip_list_);
----SkipListRep::Iterator::Seek//iter->Seek(k.internal_key(), k.memtable_key().data())
------InlineSkipList<Comparator>::Iterator::Seek
--------InlineSkipList<Comparator>::FindGreaterOrEqual
----SaveValue//返回true代表还要next，false就是不用继续了。
------s->mem->GetInternalKeyComparator().user_comparator()->Equal//如果不相等，则返回，说明不在memtable
--------DecodeFixed64(key_ptr + key_length - 8)
--------UnPackSequenceAndType(tag, &seq, &type);
--------if ((type == kTypeValue || type == kTypeMerge || type == kTypeBlobIndex) && range_del_agg->ShouldDelete(Slice(key_ptr, key_length))) {
      		type = kTypeRangeDeletion;}  
------switch (type)
--------case kTypeBlobIndex：
--------case kTypeValue:
----------s->value->assign(v.data(), v.size());//找到，value赋值
----------*(s->found_final_value) = true;
--------case kTypeDeletion:
----------*(s->status) = Status::NotFound();
----------*(s->found_final_value) = true;
--------case kTypeMerge:
----------merge_context->PushOperand
----------if (merge_operator->ShouldMerge)
------------MergeHelper::TimedFullMerge
------------*(s->found_final_value) = true;
```

