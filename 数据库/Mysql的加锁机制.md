##  表锁

> 表锁可以是显式也可以是隐式的。
>
> 显示的锁用Lock Table来创建，但要记得Lock Table之后进行操作，需要在操作结束后，使用UnLock来释放锁。
>
> Lock Tables有read和write两种，
>
> > Lock Tables......Read通常被称为共享锁或者读锁
> >
> > Lock Table......write通常被称为排他锁或者写锁



```
LOCK TABLES test_product READ, test_user WRITE;

show OPEN TABLES where In_use > 0; [查询被锁的表]
```

![1562654658397](E:\MyNote\assets\1562654658397.png)

### Lock Tables....Read

```
LOCK TABLES xxl_job_lock READ;

update xxl_job_lock set lock_name = 'schedule_lock'; [[Err] 1099 - Table 'xxl_job_lock' was locked with a READ lock and can't be updated]

select * from xxl_job_lock;

SELECT SLEEP(10);

select * from xxl_job_group; [[Err] 1100 - Table 'xxl_job_group' was not locked with LOCK TABLES]

UNLOCK TABLES;
```

1. Lock Tables....READ不会阻塞其他线程对表数据的读取，会阻塞其他线程对数据变更
2. Lock Tables....READ**不允许对表进行更新操作(新增、删除也不行)**，并且不允许访问未被锁住的表

### Lock Tables....Write

```
LOCK TABLES xxl_job_lock WRITE;

update xxl_job_lock set lock_name = 'schedule_lock';

select * from xxl_job_lock;

SELECT SLEEP(10);

select * from xxl_job_group; [[Err] 1100 - Table 'xxl_job_group' was not locked with LOCK TABLES]

UNLOCK TABLES;
```

1. Lock Tables....WRITE会阻塞其他线程对数据读和写

2. Lock Tables....WRITE允许对被锁住的表进行增删改查，但不允许对其他表进行访问

   

## 全局读锁

> 会阻塞一切更新操作（表的增删改和数据的增删改），主要用于数据库备份的时候获取一致性数据。
>
```
FLUSH TABLES WITH READ LOCK;
```
## 隐式加锁

> SELECT FOR UPDATE和LOCK IN SHARE 这种通过编写在mysql里面的方式对需要保护的数据进行加锁的方式称为是显式加锁。还有一种加锁方式是隐式加锁，除了把事务设置成串行时，会对SELECT到的所有数据加锁外，SELECT不会对数据加锁（依赖于MVCC）。当执行update、delete、insert的时候会对数据进行加排它锁。

##  自增长锁

>  mysql数据库在很多时候都会设置为主键自增，如果这个时候使用表锁，当事务比较大的时候，会对性能造成比较大的影响。mysql提供了inodb_atuoinc_lock_mode来处理自增长的安全问题。该参数可以设置为0(插入完成之后，即使事务没结束也立即释放锁)、1(在判断出自增长需要使用的数字后就立即释放锁，事务回滚也会造成主键不连续)、2（来一个记录就分配一个值，不使用锁，性能很好，但是可能导致主键不连续）。

## 外键锁

>  当插入和更新子表的时候，首先需要检查父表中的记录，并对父表加一条lock in share mode，而这可能会对两张表的数据检索造成阻塞。所以一般生产数据库上不建议使用外键。

## 索引和锁

>  InnoDB在给行添加锁的时候，其实是通过索引来添加锁，如果查询并没有用到索引，就会使用表锁。

#  InnoDB锁

> 行锁相比表锁有一些优点，行锁比表锁有更小锁粒度，可以更大的支持并发。
> 加锁动作也是需要额外开销的，比如获得锁、检查锁、释放锁等操作都是需要耗费系统资源。如果系统在锁操作上浪费了太多时间，系统的性能就会受到比较大的影响。
>
> MySQL InnoDB一共有四种锁：共享锁（读锁，S锁）、排他锁（写锁，X锁）、意向共享锁（IS锁）和意向排他锁（IX锁）。其中共享锁与排他锁属于行级锁，另外两个意向锁属于表级锁。

## 行锁

共享锁(S) 和 排它锁(X)

**共享锁**：允许事务去读一行，阻止其他事务对该数据进行修改

> 若事务T对数据对象A加上S锁，则事务T可以读A但不能修改A，其他事务只能再对A加S锁，而不能加X锁，直到T释放S锁。

**排它锁**：允许事务去读取更新数据，阻止其他事务对数据进行查询或者修改

> 若事务T对数据对象A加上X锁，则只允许T读取和修改A，其他事务不能再对A加作何类型的锁，直到T释放A上的X锁。

## 表级锁

```
考虑这个例子：

事务A锁住了表中的一行，让这一行只能读，不能写。
之后，事务B申请整个表的写锁。
如果事务B申请成功，那么理论上它就能修改表中的任意一行，这与A持有的行锁是冲突的。

数据库需要避免这种冲突，就是说要让B的申请被阻塞，直到A释放了行锁。

数据库要怎么判断这个冲突呢？

step1：判断表是否已被其他事务用表锁锁表
step2：判断表中的每一行是否已被行锁锁住。

注意step2，这样的判断方法效率实在不高，因为需要遍历整个表。
于是就有了意向锁。

在意向锁存在的情况下，事务A必须先申请表的意向共享锁，成功后再申请一行的行锁。

在意向锁存在的情况下，上面的判断可以改成

step1：不变
step2：发现表上有意向共享锁，说明表中有些行被共享行锁锁住了，因此，事务B申请表的写锁会被阻塞。

注意：申请意向锁的动作是数据库完成的，就是说，事务A申请一行的行锁的时候，数据库会自动先开始申请表的意向锁，不需要我们程序员使用代码来申请。
```

1. 为什么没有意向锁的话，表锁和行锁不能共存？
   事务A锁住表中的**一行（写锁）**。事务B锁住**整个表（写锁）**。事务A既然锁住了某一行，其他事务就不可能修改这一行。这与”事务B锁住整个表就能修改表中的任意一行“形成了冲突。所以，没有意向锁的时候，行锁与表锁共存就会存在问题！
2. 意向锁是如何让表锁和行锁共存的？
   事务A在申请行锁（写锁）之前，数据库会自动先给事务A申请表的意向排他锁。当事务B去申请表的写锁时就会失败，因为表上有意向排他锁之后事务B申请表的写锁时会被阻塞。

意向共享锁（IS）和 意向排他锁(IX)

**意向共享锁：**当一个事务要给一条数据加S锁的时候，会先对数据所在的表先加上IS锁，成功后才能加上S锁

> 事务T在对表中数据对象加S锁前，首先需要对该表加IS（或更强的IX）锁。
>
> ```
> SELECT ... FROM T1 LOCK IN SHARE MODE语句，首先会对表T1加IS锁，成功加上IS锁后才会对数据加S锁
> ```

**意向排它锁**：当一个事务要给一条数据加X锁的时候，会先对数据所在的表先加上IX锁，成功后才能加上X锁

> 事务T在对表中的数据对象加X锁前，首先需要对该表加IX锁。

> 意向锁之间兼容，不会阻塞。但是会跟S锁和X锁冲突，冲突的方式跟读写锁相同。例如当一张表上已经有一个排它锁（X锁），此时如果另外一个线程要对该表加意向锁，不管意向共享锁还是意向排他锁都不会成功。
>
> ```
> SELECT ... FROM T1 FOR UPDATE语句，首先会对表T1加IX锁，成功加上IX锁后才会对数据加X锁
> ```



**MySQL InnoDB 锁兼容阵列**

|        | **X** | **IX** | **S** | **IS** |
| ------ | ----- | ------ | ----- | ------ |
| **X**  | ✗     | ✗      | ✗     | ✗      |
| **IX** | ✗     | ✓      | ✗     | ✓      |
| **S**  | ✗     | ✗      | ✓     | ✓      |
| **IS** | ✗     | ✓      | ✓     | ✓      |

## 死锁例子

1. 会话S1以`SELECT * FROM xxl_job_user t WHERE id = 1 LOCK IN SHARE MODE;`查询，该语句首先会对xxl_job_user 表加IS锁，接着会对数据（id = 1）加S锁。

   ```sql
   [SQL]SELECT * FROM xxl_job_user WHERE id = 1 LOCK IN SHARE MODE;
   受影响的行: 0
   时间: 0.002s
   ```

2. 会话S2执行`UPDATE xxl_job_user set username = 'hua' where id = 1;`，该语句尝试对xxl_job_user 表加IX锁，由于IX锁与IS锁是兼容的，所以成功对t表加IX锁。接着继续对数据（id = 1）加X锁，但数据已经被会话S1事务加了S锁了，所以会话S2等待。

3. 会话S1也执行`UPDATE xxl_job_user set username = 'hua' where id = 1;`，该语句首先对xxl_job_user 表加IX锁，虽然会话S2已经对xxl_job_user 表加了IX锁，但IX锁与IX锁之间是兼容的，所以成功对xxl_job_user 表加上IX锁。接着会话S1会对数据（id = 1）加X锁，此时发现会话S2占有的IX锁与X锁不兼容，所以会话S1也等待。就这样，会话S1等S2释放IX锁，而会话S2等S1释放S锁，从而死锁发生。

   ```
   [Err] 1213 - Deadlock found when trying to get lock; try restarting transaction
   ```

   