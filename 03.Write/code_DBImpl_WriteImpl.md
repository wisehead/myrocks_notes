#1.DBImpl::WriteImpl

```cpp
DBImpl::WriteImpl
--switch !enable_piped_wirtes
----WriteThread writer//建立写操作，链表的节点。
----WriteThread::JoinBatchGroup//将要写的batch的write加入队列
------bool linked_as_leader = LinkOne(w, &newest_writer_)//将当前writer原子的加入到group
------if (! linked_as_leader)
--------WriteThread::AwaitState
----switch
----STATE_PARALLEL_MEMTABLE_WRITE
------WriteBatchInternal::InsertInto
------CompleteParallelMemTableWriter
--------AwaitState(w, STATE_COMPLETED, &cpmtw_ctx)//如果不是最后一个
------ExitAsBatchGroupFollower
----STATE_COMPLETED
------return w_FinalStatus()
----STATE_GROUP_LEADER
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