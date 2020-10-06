#1.Memtable::Add

```cpp
Memtable::Add
--std::unique_ptr<MemtableRep> &table = (type==KTypeRangeDeletion)? range_del_table_ : table_;
--KeyHandle handle = table->Allocate(encoded_len, &buf)
--//通过判断key-value的类型来选择memtable，范围删除的kv，插入range_del_table_
--switch !allow_concurrent
----switch insert_with_hint_prefix_extractor->InDomain(key_slice)
------insert_with_hint_prefix_extractor->Transform(key_slice)
------table->InsertKeyWithHint//所谓hint，就是利用slice中的信息。
--------SkipList<Comparator>::Insert
----switch else 
------table->InserKey(handle)
--------SkipList<Comparator>::Insert
----prefix_bloom_->Add(prefix_extractor->Transform(key))
--switch allow_concurrent
----table->InserKeyConcurrently(handle)
------SkipList<Comparator>::Insert
----prefix_bloom_->AddConcurrently(prefix_extractor->Transform(key))//修改Bloom filter

```