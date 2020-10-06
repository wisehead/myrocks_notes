#1.WriteToWAL 

```cpp
DBImpl::WriteToWAL(const WriteThread::WriteGroup& write_group,
--DBImpl::MergeBatch
--DBImpl::WriteToWAL(const WriteBatch& merged_batch,
----WriteBatchInternal::Contents
----Writer::AddRecord
------WritableFileWriter::Append//dest_->Append
--------Writer::EmitPhysicalRecord
----------WritableFileWriter::Flush// write out the cached data to the OS cache or storage if direct I/O
------------WriteBuffered(buf_.BufferStart(), buf_.CurrentSize());
------------writable_file_->Flush
----alive_log_files_.back().AddSize(log_entry.size());

--log.writer->file()->Sync(immutable_db_options_.use_fsync);
----WritableFileWriter::Sync
------WritableFileWriter::Flush
------WritableFileWriter::SyncInternal
--------if (use_fsync)
----------PosixWritableFile::Fsync//writable_file_->Fsync();
------------fsync
--------else
----------PosixWritableFile::Sync//ritable_file_->Sync();
------------fdatasync

```

写日志可以简单分为三个阶段，第一阶段往WritableFileWriter的buf_里面写，第二阶段write到系统缓存，第三阶段是将系统缓存刷盘。

刷盘有三种策略，

一种是每次提交都刷盘（WriteOptions::sync = true  -> need_log_sync=true），WriteToWAL会调用fsync，安全性比较高，但会占用较多IO。

一种是配置了参数wal_bytes_per_sync，每写入wal_bytes_per_sync大小的WAL，就将之前的WAL刷磁盘。

最后一种，也是默认的是将刷盘交由系统处理，仅在memtable切换、flush等操作触发时再去主动调用刷盘函数。






