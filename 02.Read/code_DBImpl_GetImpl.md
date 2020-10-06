#1.DBImpl::GetImpl

###数据库实现层逻辑
	Rocksdb::DB是个抽象基类，其中定义些纯虚函数，包括Get，Put操作等，具体实现时调用它的子类rocksdb::DBImpl实现这些接口功能。Get的功能就是油DBImpl::GetImpl完成。
	其中通过SuperVersion可以找到当前的Memtable、Immutable Memtable队列、以及当前Version（由此可以找到当前版本的sstable结构中的所有文件），它是由columnfamily提供维护的。
	而Version时由一个时间点所有有效SST文件组成。每次campact完成，将会创建新的version。
	旧版本的super version和version在引用为零时会被清除。
	

```cpp
	//SuperVersion数据结构定义
	// holds references to memtable, all immutable memtables and version
	struct SuperVersion {
	  // Accessing members of this class is not thread-safe and requires external
	  // synchronization (ie db mutex held or on write thread).
	  MemTable* mem;                                                                                                                                                                                                                                                
	  MemTableListVersion* imm;
	  Version* current;
  
  
DBImpl::GetImpl
--GetAndRefSuperVersion
--if (!skip_memtable)
----MemTable::Get//sv->mem->Get
----MemTableListVersion::Get//sv->imm->Get
--else
----Version::Get//sv->current->Get
--ReturnAndCleanupSuperVersion
```