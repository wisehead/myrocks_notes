#1.insert

```cpp
mysql_execute_command
--mysql_insert
----write_record
------handler::ha_write_row
--------myrocks::ha_rocksdb::write_row
----------myrocks::ha_rocksdb::skip_unique_check
----------myrocks::ha_rocksdb::update_write_row//!!!!!!!!!!!!!!!!!!!
------------myrocks::get_or_create_tx
--------------get_tx_from_thd
----------------thd_ha_data
------------------thd->ha_data[hton->slot].ha_ptr
--------------new Rdb_writebatch_impl(thd);//slave
--------------new Rdb_transaction_impl(thd);
--------------start_tx
----------------rdb->BeginTransaction(write_opts, tx_opts, m_rocksdb_reuse_tx);
------------myrocks::ha_rocksdb::get_pk_for_update
--------------rocksdb::update_hidden_pk_val
------------myrocks::ha_rocksdb::check_uniqueness_and_lock
--------------myrocks::ha_rocksdb::check_and_lock_unique_pk
----------------get_for_update
------------------TransactionBaseImpl::GetForUpdate
--------------------PessimisticTransaction::TryLock
------------myrocks::ha_rocksdb::update_indexes
--------------myrocks::ha_rocksdb::update_pk
----------------myrocks::ha_rocksdb::convert_record_to_storage_format
----------------myrocks::Rdb_transaction_impl::put
------------------rocksdb::TransactionBaseImpl::Put
--------------------rocksdb::PessimisticTransaction::TryLock
----------------------rocksdb::TransactionBaseImpl::SetSnapshotIfNeeded
----------------------rocksdb::TransactionBaseImpl::TrackKey//把所有的锁，保存到map里
------------------------rocksdb::TransactionBaseImpl::TrackKey
--------------------rocksdb::TransactionBaseImpl::GetBatchForWrite

------------myrocks::ha_rocksdb::do_bulk_commit//myrocks::ha_rocksdb::update_write_row end !!!!!!!!!!!!!!!!
--------------myrocks::ha_rocksdb::commit_in_the_middle
----------binlog_log_row

```