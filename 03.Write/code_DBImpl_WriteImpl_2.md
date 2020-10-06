#1.数据库实现层逻辑: DBImpl::WriteImpl


每次写请求并不是直接读写Memtable的，而是打包进Writer等待批量写入的，这样的一个重要意义是可以使得一个事务内的的操作一次性写入数据和维护版本等信息。
后续多个线程的Writer会组成一个Group，并选出第一个为leader，以组的形式完成日志提交和Memtable写入。


RocksDB的维护和更新非常及时。几个有代表的优化和新功能是：pipelinewrite、concurrent_memtable_write、2pc状态下的concurrent_WriteToWAL。随着新功能的增加，在数据库层的代码逻辑也越来越复杂，分支非常多。

总的来说rocksdb会将多个写线程组成一个group，leader负责 group内所有writer的WAL及memtable的提交，提交完后唤醒所有的follwer，向上层返回。
后续更新支持 allow_concurrent_memtable_write 选项，在之前的基础上，leader提交完WAL后，group里所有线程并发写 memtable，流程如下图。

而 enable_pipelined_write 选项，引入了流水线特性，第一个 group 的 WAL 提交后，在执行 memtable 写入前，下一个 group 同时开启。

```cpp
DBImpl::WriteImpl
--//if enable_pipelined_write
----WriteThread::Writer w(write_options, my_batch, callback, log_ref,disable_memtable, pre_release_callback);
----WriteThread::JoinBatchGroup(&w);
------linked_as_leader = LinkOne(w, &newest_writer_);
------if (linked_as_leader) {
        SetState(w, STATE_GROUP_LEADER);
      }    
------if (!linked_as_leader) {
        /**
         * Wait util:
         * 1) An existing leader pick us as the new leader when it finishes
         * 2) An existing leader pick us as its follewer and
         * 2.1) finishes the memtable writes on our behalf
         * 2.2) Or tell us to finish the memtable writes in pralallel
         * 3) (pipelined write) An existing leader pick us as its follower and
         *    finish book-keeping and WAL write for us, enqueue us as pending
         *    memtable writer, and
         * 3.1) we become memtable writer group leader, or
         * 3.2) an existing memtable writer group leader tell us to finish memtable
         *      writes in parallel.
         */
        AwaitState(w, STATE_GROUP_LEADER | STATE_MEMTABLE_WRITER_LEADER |
                          STATE_PARALLEL_MEMTABLE_WRITER | STATE_COMPLETED,
                   &jbg_ctx);
      } 
----//if (w.state == WriteThread::STATE_PARALLEL_MEMTABLE_WRITER)
------WriteBatchInternal::InsertInto
------write_thread_.CompleteParallelMemTableWriter(&w)
--------AwaitState(w, STATE_COMPLETED, &cpmtw_ctx);
------write_thread_.ExitAsBatchGroupFollower(&w); 

----//if (w.state == WriteThread::STATE_COMPLETED)
------ return w.FinalStatus();

----//if w.state == WriteThread::STATE_GROUP_LEADER
------write_thread_.EnterAsBatchGroupLeader// 把次leader下所有writer都链接到一个writegroup中
------//if (!two_write_queues_)
--------WriteToWAL
------//if (two_write_queues_)
--------ConcurrentWriteToWAL
------//if (!parallel)//非并发写
--------WriteBatchInternal::InsertInto
------// if (parallel) 并发写
--------write_thread_.LaunchParallelMemTableWriters
--------WriteBatchInternal::InsertInto
--------write_thread_.CompleteParallelMemTableWriter
--------write_thread_.ExitAsBatchGroupLeader

--//if not enable_pipelined_write
----DBImpl::PipelinedWriteImpl
------WriteThread::Writer w(write_options, my_batch, callback, log_ref,disable_memtable);
------write_thread_.JoinBatchGroup
------// if STATE_GROUP_LEADER
--------WriteThread::WriteGroup wal_write_group;
--------write_thread_.EnterAsBatchGroupLeader
--------WriteToWAL
--------ExitAsBatchGroupLeader
----------if (LinkGroup(write_group, &newest_memtable_writer_)) {
          // The leader can now be different from current writer.
          SetState(write_group.leader, STATE_MEMTABLE_WRITER_LEADER);
          }
					// Link the ramaining of the group to memtable writer list. 将wal_write_group连接到newest_memtable_writer_队列中，然后选择MEMTABLE_WRITER队列中的leader，并唤醒它。
----------// Reset newest_writer_ and wake up the next leader.
          Writer* newest_writer = last_writer;
          if (!newest_writer_.compare_exchange_strong(newest_writer, nullptr)) {
            Writer* next_leader = newest_writer;
            while (next_leader->link_older != last_writer) {
              next_leader = next_leader->link_older;
              assert(next_leader != nullptr);
            }
            next_leader->link_older = nullptr;
            SetState(next_leader, STATE_GROUP_LEADER);//选择newest_writer队列中的新leader，并唤醒它。
          }  
          AwaitState(leader, STATE_MEMTABLE_WRITER_LEADER |
                                 STATE_PARALLEL_MEMTABLE_WRITER | STATE_COMPLETED, &eabgl_ctx);

          //自己沉睡于MEMTABLE_WRITER队列中。
------// if STATE_MEMTABLE_WRITER_LEADER
--------write_thread_.EnterAsMemTableWriter
--------if (memtable_write_group.size > 1 && immutable_db_options_.allow_concurrent_memtable_write) {
				  write_thread_.LaunchParallelMemTableWriters
--------else
				  WriteBatchInternal::InsertInto
				  write_thread_.ExitAsMemTableWriter				  
```



				  
				  
				  