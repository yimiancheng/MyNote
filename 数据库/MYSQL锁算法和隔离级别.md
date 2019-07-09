二阶段锁





- **快照读：**简单的select操作，属于快照读，不加锁。(当然，也有例外，下面会分析)

   - select * from table where ?;

    

- **当前读：**特殊的读操作，插入/更新/删除操作，属于当前读，需要加锁。

   - select * from table where ? lock in share mode;
   - select * from table where ? for update;
   - insert into table values (…);
   - update table set ? where ?;
   - delete from table where ?;

   所有以上的语句，都属于当前读，读取记录的最新版本。并且，读取之后，还需要保证其他并发事务不能修改当前记录，对读取记录加锁。其中，除了第一条语句，对读取记录加S锁 (共享锁)外，其他的操作，都加的是X锁 (排它锁)。

    

MySQL/InnoDB定义的4种隔离级别：

- **Read Uncommited**

   可以读取未提交记录。此隔离级别，不会使用，忽略。

- **Read Committed (RC)**

   快照读忽略，本文不考虑。

   针对当前读，**RC隔离级别保证对读取到的记录加锁 (记录锁)**，存在幻读现象。

- **Repeatable Read (RR)**

   快照读忽略，本文不考虑。

   针对当前读，**RR隔离级别保证对读取到的记录加锁 (记录锁)，同时保证对读取的范围加锁，新的满足查询条件的记录不能够插入 (间隙锁)**，不存在幻读现象。

- **Serializable**

   从MVCC并发控制退化为基于锁的并发控制。不区别快照读与当前读，所有的读操作均为当前读，读加读锁 (S锁)，写加写锁 (X锁)。

   Serializable隔离级别下，读写冲突，因此并发度急剧下降，在MySQL/InnoDB下不建议使用。