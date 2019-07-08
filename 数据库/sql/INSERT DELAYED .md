#### INSERT DELAYED 

> 当一个客户端使用INSERT DELAYED时，会立刻从服务器处得到一个确定。并且行被排入队列，当表没有被其它线程使用时，此行被插入。
>
> 使用INSERT DELAYED的另一个重要的好处是，来自许多客户端的插入被集中在一起，并被编写入一个块。这比执行许多独立的插入要快很多。



> InnoDB可以使用事务，不支持insert delayed.
>
> MyISAM不能支持事务，可拼接insert语句，insert into table values ('value', 'value'), ('value1', 'value1');
>
> 5.6以上的mysql版本不支持insert delayed，insert和insert delayed在MyISAM引擎上效率一致。

