#1.do_select

```cpp
do_select
--sub_select
----join_init_read_record
------init_read_record
--------handler::ha_rnd_init
----------myrocks::ha_rocksdb::rnd_init
----rr_sequential
------handler::ha_rnd_next
--------myrocks::ha_rocksdb::rnd_next
----------myrocks::ha_rocksdb::rnd_next_with_direction
------------rocksdb::BaseDeltaIterator::Next
--------------rocksdb::BaseDeltaIterator::Advance
----------------rocksdb::BaseDeltaIterator::AdvanceBase
----------------rocksdb::ArenaWrappedDBIter::Next
------------------rocksdb::ArenaWrappedDBIter::Next
--------------------rocksdb::DBIter::Next
----------------------rocksdb::MemTableIterator::Next
------------------------rocksdb::(anonymous namespace)::SkipListRep::Iterator::Next 
--------------------------rocksdb::InlineSkipList<rocksdb::MemTableRep::KeyComparator const&>::Node::Next
----------------------rocksdb::DBIter::FindNextUserEntry
------------------------rocksdb::DBIter::FindNextUserEntryInternal
----------------rocksdb::BaseDeltaIterator::UpdateCurrent
------------myrocks::ha_rocksdb::convert_record_from_storage_format
--------------myrocks::ha_rocksdb::convert_field_from_storage_format
--------------Field::move_field
--------------myrocks::ha_rocksdb::convert_varchar_from_storage_format
----evaluate_join_record
------myrocks::ha_rocksdb::unlock_row

```