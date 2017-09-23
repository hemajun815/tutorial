# 删除数据库数据，清理自增字段
## 具体方法
1. delete from
2. truncate table
## 方法比较
* truncate table在功能上与不带 where 子句的 delete 语句相同：二者均删除表中的全部行。但 truncate table 比 delete 速度快，且使用的系统和事务日志资源少。
* delete 语句每次删除一行，并在事务日志中为所删除的每行记录一项。truncatetable 通过释放存储表数据所用的数据页来删除数据，并且只在事务日志中记录页的释放。
* truncate table 删除表中的所有行，但表结构及其列、约束、索引等保持不变。新行标识所用的计数值重置为该列的种子。如果想保留标识计数值，请改用 delete。如果要删除表定义及其数据，请使用 drop table 语句。
* 对于由 foreign key 约束引用的表，不能使用 truncate table，而应使用不带 where 子句的 delete 语句。由于 truncate table 不记录在日志中，所以它不能激活触发器。
* truncate table 不能用于参与了索引视图的表。
