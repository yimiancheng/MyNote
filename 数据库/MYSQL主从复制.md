



## 





mysql binlog的格式三种：statement,row,mixed**



**为什么mysql用的是repeatable而不是read committed:在 5.0之前只有statement一种格式，而主从复制存在了大量的不一致，故选用repeatable**



**在RC级用别下，主从复制用什么binlog格式：row格式，是基于行的复制！**





```
时间以插入记录的时间为准，与commit时间无关。
```

```
mysql> select version();
+------------+
| version()  |
+------------+
| 5.7.23-log |
+------------+
1 row in set (0.00 sec)

mysql> status
--------------
mysql  Ver 14.14 Distrib 5.1.73, for redhat-linux-gnu (x86_64) using readline 5.1

Connection id:          18194
Current database:
Current user:           root@11.42.51.125
SSL:                    Not in use
Current pager:          stdout
Using outfile:          ''
Using delimiter:        ;
Server version:         5.7.23-log MySQL Community Server (GPL)
Protocol version:       10
Connection:             172.20.129.98 via TCP/IP
Server characterset:    utf8mb4
Db     characterset:    utf8mb4
Client characterset:    utf8
Conn.  characterset:    utf8
TCP port:               3306
Uptime:                 74 days 23 hours 52 min 43 sec

Threads: 39  Questions: 2702078  Slow queries: 395  Opens: 10627  Flush tables: 1  Open tables: 10227  Queries per second avg: 0.417
--------------
```

