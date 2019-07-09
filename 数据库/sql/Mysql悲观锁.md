## 锁（locking）

> 数据库为了维护一致性和隔离性，一般使用加锁这种方式
>
> 数据库是个高并发的应用，同一时间会有大量的并发访问，如果**加锁过度，会极大的降低并发处理能力**
>

## 悲观锁 (  Pessimistic Locking )

> 对数据被外界（包括本系统当前的其他事务，以及来自外部系统的事务处理）修改持保守态度
> 
> 悲观锁的实现，往往依靠数据库提供的锁机制（也只有数据库层提供的锁机制才能真正保证数据访问的排他性，否则，即使在本系统中实现了加锁机制，也无法保证外部系统不会修改数据）

```
select * from account where name="Erica" for update　
这条 sql 语句锁定了 account 表中所有符合检索条件（ name="Erica" ）的记录。
本次事务提交之前（事务提交时会释放事务过程中的锁），外界无法修改这些记录。 
```

##  SELECT ...FOR UPDATE(排他锁)

```sql
START TRANSACTION;
BEGIN;

select * from xxl_job_lock where lock_name = 'schedule_lock' for update;

SELECT SLEEP(10);
COMMIT;
```

> 当执行`select status from t_goods where id=1 for update;`后，在另外的事务中如果再次执行`select status from t_goods where id=1 for update;`则第二个事务会一直等待第一个事务的提交，此时**第二个查询处于阻塞的状态**，但是如果我是在第二个事务中执行`select status from t_goods where id=1;`则能正常查询出数据，不会受第一个事务的影响

```
for update的字段为索引或者主键的时候，只会锁住索引或者主键对应的行。
for update的字段为普通字段的时候，Innodb会锁住整张表。
```

> MySQL InnoDB默认Row-Level Lock，所以只有「明确」地指定主键，MySQL 才会执行Row lock (只锁住被选取的数据) ，否则(**无主键或者主键不明确**)MySQL 将会执行Table Lock (将整个数据表单给锁住)。

```
SELECT * from t_goods where status=1 for update; # 无主键
SELECT * from t_goods where id>1 for update; # 主键不明确
```

## 应用场景

1. 使用数据库悲观锁实现不可重入的分布式锁

       ```java
conn.setAutoCommit(false);
preparedStatement = conn.prepareStatement(  "select * from xxl_job_lock where lock_name = 'schedule_lock' for update" );
preparedStatement.execute();
       ```

