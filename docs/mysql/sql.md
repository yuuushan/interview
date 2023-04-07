# SQL

## DELETE、TRUNCATE 和 DROP 的区别

- `DROP table` 会删除表的结构、被依赖的约束（constrain）、触发器（trigger）、索引（index）；依赖于该表的存储过程、函数将保留，但是变为 invalid 状态。
- `DELETE` 和 `TRUNCATE` 只删除表中的数据。

在删除表中所有数据但不删除表结构的情况下，使用 `TRUNCATE table`，其速度更快，因为：

- `DELETE` 命令会记录数据的变动。删除的数据将存储在 Rollback Segments 中，需要的时候，数据可以回滚恢复。
- `TRUNCATE` 命令删除的数据是不可恢复的。
