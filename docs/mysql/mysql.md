# MySQL

## MySQL 主从复制的原理。

- 主：记录下所有改变了数据库数据的语句，放进 master 上的 binlog 中。主库会生成一个 log dump 线程，用来给从库 I/O 线程传 Binlog 数据。
- 从：I/O 线程会去请求主库的 Binlog，并将得到的 Binlog 写到本地的 relay log（中继日志）文件中。
- 从：SQL 线程，会读取 relay log 文件中的日志，并解析成 SQL 语句逐一执行。
