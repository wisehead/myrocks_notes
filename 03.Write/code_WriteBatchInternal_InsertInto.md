#1.WriteBatchInternal::InsertInto

```cpp
WriteBatchInternal::InsertInto
--MemtableInserter inserter
--//Iterate这个方法将WriteBatch中的内容除了头的12字节外，一条条取记录，然后根据类型调用handler（MemtableInserter）中的方法处理。
--for (auto w:write_group) {
  	w->status = w->batch->Iterate(&inserter)}
----ReadRecordFromWriteBatch
------switch(tag)
------MemTableInserter::PutCF
--------PutCFImpl
----------SeekToColumnFamily
----------switch !inplcate_update_support
------------Memtable::Add
----------switch inplcate_update_support
------------switch inpace_callback==null
--------------Memtable::Update
----------------std::unique_ptr<MemTableRep::Iterator> iter(table->GetDynamicPrefixIterator())
----------------iter->Seek(lkey.internal_key(), mem_key.data())
------------------typename SkipList<Key, Comparator>::Node* SkipList<Key, Comparator>::FindGreaterOrEqual(const Key &key)
----------------switch iter->Valid() && new_size <= pre_size//找到了，且新的value在原来的地方写的下
------------------EncodeVarint32(const_cast<char*>(key_ptr)+key_len, new_size)
----------------switch 找不到或者写不下
------------------Memtable::Add

------------switch inpace_callback!=null
--------------if (mem->UpdateCallback(sequence_, key, value))//memtable中找，根据callback要求处理
----------------std::unique_ptr<MemTableRep::Iterator> iter(table->GetDynamicPrefixIterator())
----------------iter->Seek(lkey.internal_key(), mem_key.data())
----------------switch iter->Valid()//找到了
------------------auto status = moptions_inplace_callback(pre_buffer, &new_pre_size, delta, &str_value)
------------------switch
--------------------EncodeVarint32(const_cast<char*>(key_ptr)+key_len, new_size)//status返回为原地修改，且写得下
--------------------Memtable::Add//写不下
----------------switch iter->Valid()//找不到
------------------DBImpl::Get
------------------auto status = moptions->inplace_callback//调用callback函数，可以比较新旧值并对获取的值进行处理
------------------Memtable::Add//按照status存不同的值
----------MaybeAdvanceSeq
----------CheckMemtableFull

```