#1.myrocks::ha_rocksdb::rnd_init

read流程图

```cpp

mysql_execute_command
--execute_sqlcom_select
----handle_select
------mysql_select
--------mysql_execute_select
----------JOIN::exec
------------do_select
--------------sub_select
----------------join_init_read_record
------------------init_read_record
--------------------handler::ha_rnd_init
----------------------myrocks::ha_rocksdb::rnd_init

myrocks::ha_rocksdb::rnd_init
--myrocks::get_or_create_tx
--myrocks::ha_rocksdb::setup_read_decoders
--myrocks::ha_rocksdb::setup_iterator_for_rnd_scan
----myrocks::Rdb_key_def::get_first_key
------myrocks::Rdb_key_def::get_infimum_key
--------myrocks::rdb_netbuf_store_index
----myrocks::ha_rocksdb::setup_scan_iterator
------myrocks::get_or_create_tx
------myrocks::ha_rocksdb::check_bloom_and_set_bounds
--------myrocks::ha_rocksdb::can_use_bloom_filter
--------myrocks::ha_rocksdb::setup_iterator_bounds
------myrocks::Rdb_transaction::get_iterator
--------myrocks::Rdb_transaction_impl::acquire_snapshot
----------TransactionBaseImpl::SetSnapshot
------------rocksdb::DBImpl::GetSnapshotForWriteConflictBoundary
--------------rocksdb::DBImpl::GetSnapshotImpl
----------------rocksdb::SnapshotList::New
------------rocksdb::TransactionBaseImpl::SetSnapshotInternal
----------rocksdb::TransactionBaseImpl::GetSnapshot
----------myrocks::Rdb_transaction::snapshot_created
----rocksdb::BaseDeltaIterator::Seek//setup_iterator_for_rnd_scan
------rocksdb::DBIter::Seek
--------rocksdb::MemTableIterator::Seek
----------rocksdb::(anonymous namespace)::SkipListRep::Iterator::Seek
------------rocksdb::InlineSkipList<rocksdb::MemTableRep::KeyComparator const&>::Iterator::Seek
--------------rocksdb::InlineSkipList<rocksdb::MemTableRep::KeyComparator const&>::FindGreaterOrEqual 
--------rocksdb::DBIter::FindNextUserEntry
----------rocksdb::DBIter::FindNextUserEntryInternal
------rocksdb::WBWIIteratorImpl::Seek
--------rocksdb::SkipList<rocksdb::WriteBatchIndexEntry*, rocksdb::WriteBatchEntryComparator const&>::Iterator::Seek
----------rocksdb::SkipList<rocksdb::WriteBatchIndexEntry*, rocksdb::WriteBatchEntryComparator const&>::FindGreaterOrEqual
------rocksdb::BaseDeltaIterator::UpdateCurrent

```