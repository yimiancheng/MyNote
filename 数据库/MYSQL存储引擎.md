#### 存储引擎

> 关系数据库表是用于存储和组织信息的数据结构，可以将表理解为由行和列组成的表格，类似于Excel的电子表格的形式。

> > 有的表简单，有的表复杂，有的表根本不用来存储任何长期的数据，有的表读取时非常快，但是插入数据时去很差；而我们在实际开发过程中，就可能需要各种各样的表，不同的表，就意味着存储不同类型的数据，数据的处理上也会存在着差异。

> 对于MySQL来说，它提供了很多种类型的存储引擎，我们可以根据对数据处理的需求，选择不同的存储引擎，从而最大限度的利用MySQL强大的功能。

##### 支持的存储引擎

```sql
mysql> show engines;
+--------------------+---------+----------------------------------------------------------------+--------------+------+------------+
| Engine             | Support | Comment                                                        | Transactions | XA   | Savepoints |
+--------------------+---------+----------------------------------------------------------------+--------------+------+------------+
| CSV                | YES     | CSV storage engine                                             | NO           | NO   | NO         |
| MRG_MYISAM         | YES     | Collection of identical MyISAM tables                          | NO           | NO   | NO         |
| MyISAM             | YES     | MyISAM storage engine                                          | NO           | NO   | NO         |
| BLACKHOLE          | YES     | /dev/null storage engine (anything you write to it disappears) | NO           | NO   | NO         |
| PERFORMANCE_SCHEMA | YES     | Performance Schema                                             | NO           | NO   | NO         |
| MEMORY             | YES     | Hash based, stored in memory, useful for temporary tables      | NO           | NO   | NO         |
| ARCHIVE            | YES     | Archive storage engine                                         | NO           | NO   | NO         |
| InnoDB             | DEFAULT | Supports transactions, row-level locking, and foreign keys     | YES          | YES  | YES        |
| FEDERATED          | NO      | Federated MySQL storage engine                                 | NULL         | NULL | NULL       |
+--------------------+---------+----------------------------------------------------------------+--------------+------+------------+
9 rows in set (0.00 sec)
```

##### 默认的存储引擎

`show variables like '%storage_engine%';`

```

mysql>  show variables like '%storage_engine%';
+----------------------------------+--------+
| Variable_name                    | Value  |
+----------------------------------+--------+
| default_storage_engine           | InnoDB |  # default_storage_engine  永久表
| default_tmp_storage_engine       | InnoDB |  # default_tmp_storage_engine  临时表
| disabled_storage_engines         |        |
| internal_tmp_disk_storage_engine | InnoDB |
+----------------------------------+--------+
4 rows in set (0.01 sec)
```

##### 查看当前表的存储引擎

1. `show create table test;`
2. ` show table status from clog(数据库库名) where name='test'(表名);`

#### MyISAM

`mysql5.5之前默认的存储的引擎`

**特点：**

1. 不支持行锁(MyISAM只有表锁)，**读写操作会相互冲突**， 读取时对需要读到的所有表加锁，写入时则对表加排他锁；

   > 对同一个表进行查询和插入操作时，为了降低锁竞争的频率，根据concurrent_insert的设置，MyISAM是可以**并行处理询和插入**的：
   >
   > 当concurrent_insert=0时，不允许并发插入功能。
   >
   > 当concurrent_insert=1时，允许对没有空洞（即表的中间没有被删除的行）的表使用并发插入，新数据位于数据文件结尾。
   >
   > 当concurrent_insert=2时，不管表有没有洞洞，都允许在数据文件结尾并发插入。把concurrent_insert设置为2是很划算的，至于由此产生的文件碎片，可以定期使用OPTIMIZE TABLE语法优化。

2. 不支持事务

3. 不支持外键

4. 不支持崩溃后的安全恢复

5. 支持BLOB和TEXT的前500个字符索引，支持全文索引

6. 支持延迟更新索引，极大地提升了写入性能

7. 于不会进行修改的表，支持压缩表 ，极大地减少了磁盘空间的占用

8. 缓存有表meta-data（行数等），COUNT(*)时对于一个结构很好的查询是不需要消耗多少资源

9. MyISAM表是独立于操作系统的，这说明可以轻松地将其从Windows服务器移植到Linux服务器。

   ```
   每当我们建立一个MyISAM引擎的表时，就会在本地磁盘上建立三个文件，文件名就是表名。
   我建立了一个MyISAM引擎的tb_Demo表，那么就会生成以下三个文件：
   1.tb_demo.frm，存储表定义
   2.tb_demo.MYD，存储数据
   3.tb_demo.MYI，存储索引
   ```

**问题：**

1. max_write_lock_count 写操作的优先级要高于读操作的优先级，即便是先发送的读请求，后发送的写请求，此时也会优先处理写请求，然后再处理读请求

   一旦我发出若干个写请求，就会堵塞所有的读请求，直到写请求全都处理完，才有机会处理读请求。

   * max_write_lock_count=1  当系统处理一个写操作后，就会暂停写操作，给读操作执行的机会
   * low-priority-updates=1 直接降低写操作的优先级，给读操作更高的优先级

2. 经常会碰到会话处于表级锁等待（Waiting for table level lock）的情况，会导致连接数堆积，**终止长时间的查询解决**。

**使用场景：** `MyISAM存储引擎很适合管理邮件或Web服务器日志数据`

1. 选择密集型的表。MyISAM存储引擎在筛选大量数据时非常迅速，这是它最突出的优点。
2. 插入密集型的表。MyISAM的并发插入特性允许同时选择和插入数据。

#### InnoDB

`mysql5.6之后默认的存储的引擎`

> InnoDB是一个健壮的事务型存储引擎，这种存储引擎已经被很多互联网公司使用，为用户操作非常大的数据存储提供了一个强大的解决方案。

**特点：**

1. 支持行锁，采用MVCC来支持高并发，有可能死锁

   > 

2. 支持事务

3. 支持外键，**MySQL支持外键的存储引擎只有InnoDB**

4. 不支持全文索引

5. 支持崩溃后的安全恢复

> 

#### MEMORY

> 使用MySQL Memory存储引擎的出发点是速度。为得到最快的响应时间，采用的逻辑存储介质是系统内存。虽然在内存中存储表数据确实会提供很高的性能，但当mysqld守护进程崩溃时，所有的Memory数据都会丢失。
>
> 获得速度的同时也带来了一些缺陷。它要求存储在Memory数据表里的数据使用的是长度不变的格式，这意味着不能使用BLOB和TEXT这样的长度可变的数据类型，VARCHAR是一种长度可变的类型，但因为它在MySQL内部当做长度固定不变的CHAR类型，所以可以使用。

**使用场景：**

1. 目标数据较小，而且被非常频繁地访问。在内存中存放数据，所以会造成内存的使用，可以通过参数max_heap_table_size控制Memory表的大小，设置此参数，就可以限制Memory表的最大大小。
2. 如果数据是临时的，而且要求必须立即可用，那么就可以存放在内存表中。
3. 存储在Memory表中的数据如果突然丢失，不会对应用服务产生实质的负面影响。
4. Memory同时支持散列索引和B树索引。B树索引的优于散列索引的是，可以使用部分查询和通配查询，也可以使用<、>和>=等操作符方便数据挖掘。散列索引进行“相等比较”非常快，但是对“范围比较”的速度就慢多了，因此散列索引值适合使用在=和<>的操作符中，不适合在<或>操作符中。