#### 行锁（Record Lock）

> 锁直接加在索引记录上面，锁住的是key。
>
> 行锁锁定的是索引记录，而不是行数据，也就是说锁定的是key。
>
> 如果InnoDB存储引擎表在建立的时候没有设置任何一个索引，那么这是InnoDB存储引擎会使用隐式的主键来进行锁定。

#### 间隙锁（Gap Lock）

>  间隙锁，锁定一个范围，但不包含记录本身。
>
>  锁定索引记录间隙，确保索引记录的间隙不变。间隙锁是针对事务隔离级别为**可重复读或以上级别**而已的。

####  Next-Key Lock

> 行锁和间隙锁组合起来就叫Next-Key Lock
>
> InnoDB工作在可重复读隔离级别下，并且会以Next-Key Lock的方式对数据行进行加锁，这样可以有效防止幻读的发生。
>
> 当InnoDB扫描索引记录的时候，会首先对索引记录加上行锁（Record Lock），再对索引记录两边的间隙加上间隙锁（Gap Lock）。加上间隙锁之后，其他事务就不能在这个间隙修改或者插入记录。

>  <font color=red>**只在REPEATABLE READ或以上的隔离级别下的特定操作才会取得gap lock或Nextkey lock**</font>





































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