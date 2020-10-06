#1.DBImpl::WriteImpl

```cpp
DBImpl::WriteImpl
--switch enable_piped_wirtes
----PipelinedWriteImpl//这里简单画了，很多东西和另一个分支重复
----WriteThread::Writer w(write_options, my_batch, callback, log_ref, disable_memtable)
----write_thread_.JoinBatchGroup(&w)
----switch
----case STATE_GROUP_LEADER
------WriteThread::WriteGroup wal_write_group
------EnterAsBatchGroupLeader
------WriteToWAL
------ExitAsBatchGroupLeader
--------if (LinkGroup(write_group, &newest_memtable_writer_))
----------SetState(write_group_leader, STATE_MEMTABLE_WRITER_LEADER)
--------//Line the remaining of the group to memtable writer list, 将wal_write_group连到newest_memtable_writer队列中，然后选择MEMTABLE_WRITER队列中的Leader，并唤醒它。
--------if (!newest_writer.compare_exchange_strong(newest_writer, null)){
          Writer* next_leader = newest_writer;
          while (next_leader->link_order != last_writer) {
            next_leader = next_leader->link_order;
            assert(next_leader != null);
          }
          next_leader->link_order = null;
          SetState(next_leader, STATE_GROUP_LEADER);//选择newest_writer中的leader，并唤醒它。
        }
        AwaitState(leader, STATE_MEMTABLE_WRITER_LEADER|STATE_PARALLEL_MEMTABLE_WRITER | STATE_COMPLETED, &eabgl_ctx)
        //沉睡于MEMTABLE_WRITER队列中
----case STATE_MEMTABLE_WRITER_LEADER
------EnterAsMemtableWriter
------switch  if memtable_write_group.size > 1 && mmutable_db_options_allow_concurrent_memtable_write
--------LauchParallelMemtableWrites
------switch else
--------WriteBatchInternal::InsertInto
--------ExitAsMemtableWriter
----case STATE_PARALLEL_MEMTABLE_WRITER:
------//类似非pipeline
----case STATE_COMPLETED:
------return w.FinalStatus()

--switch !enable_piped_wirtes
----WriteThread writer//建立写操作，链表的节点。
----WriteThread::JoinBatchGroup//将要写的batch的write加入队列
------bool linked_as_leader = LinkOne(w, &newest_writer_)//将当前writer原子的加入到group
------if (! linked_as_leader)
--------WriteThread::AwaitState
----switch
----case STATE_PARALLEL_MEMTABLE_WRITE
------WriteBatchInternal::InsertInto
------CompleteParallelMemTableWriter
--------AwaitState(w, STATE_COMPLETED, &cpmtw_ctx)//如果不是最后一个
------ExitAsBatchGroupFollower
----case STATE_COMPLETED
------return w_FinalStatus()
----case STATE_GROUP_LEADER
------WriteThread::EnterAsBatchGroupLeader//把leader下所有的write都连接到一个WriteGroup里
------switch
------!two_write_queues_(2PC)
--------ConcurrentWriteToWAL
------two_write_queues_
--------WriteToWAL
------switch
------！parallel
--------WriteBatchInternal::InsertInto
------parallel
--------WriteThread::LaunchParallelMemTableWrites
----------for (auto w:*write_group)
                  SetState(w, STATE_PARALLEL_MEMTABLE_WRITER);
--------WriteThread::WriteBatchInternal::InsertInto
--------WriteThread::CompleteParallelMemTableWrites
----------AwaitState(w, STATE_COMPLETED, &cpmtw_ctx)//如果不是最后一个
------ExitAsBatchGroupLeader
--------switch
--------enable_pipelined_write_//对应的是上面PipelinedWriteImpl中的内容
--------！enable_pipelined_write_
----------SetState(next_leader, STATE_GROUP_LEADER)//选择newest_writer_队列中新的leader并唤醒它。
```