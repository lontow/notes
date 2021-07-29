# innoDB 事务

## 事务的隔离级别

### ISO/ANIS SQL 制定的隔离级别

- **READ UNCOMMITTED**

  浏览访问。事务A可以读取到事务B修改过但未提交的数据。

  可能发生脏读、不可重复读和幻读问题，一般很少使用此隔离级别。

- **READ COMMITTED**

  游标稳定。事务B只能在事务A修改过并且已提交后才能读取到事务B修改的数据。

  读已提交隔离级别解决了脏读的问题，但可能发生不可重复读和幻读问题，一般很少使用此隔离级别。

- **REPEATABLE READ**

  无幻读保护。事务B只能在事务A修改过数据并提交后，自己也提交事务后，才能读取到事务B修改的数据。

  可重复读隔离级别解决了脏读和不可重复读的问题，但可能发生幻读问题。

- **SERIALIZABLE**

  隔离。各种问题（脏读、不可重复读、幻读）都不会发生，通过加锁实现（读锁和写锁）。

**注意：** innoDB 默认支持的隔离级别是，**REPEATABLE READ**. 但同时使用 **Next-Key** Lock 锁避免幻读。

**幻读问题；** 两次 SQL 返回的结果不同，第二次返回的结果更多。

# innoDB 锁

## 共享锁和排他锁

行级锁

## 意向锁

多粒度锁（同时存在行锁和表锁）。意向锁是表级锁。持有意向锁，意味着接下来该事务可能要获取该表中行的锁。

在获取行锁之前，必须先获取所在表的相同类型的意向锁。

## 记录锁

索引记录上的锁。

### Gap Locks

范围锁。但不包含记录本身。在持有时，保证别的事务不能向该范围的插入值。没有读写类型之分。



### Next-Key Locks

当前索引记录上的记录锁和该索引之前的Gap Lock。

Suppose that an index contains the values 10, 11, 13, and 20. The possible next-key locks for this index cover the following intervals, where a round bracket denotes exclusion of the interval endpoint and a square bracket denotes inclusion of the endpoint:

```none
(negative infinity, 10]
(10, 11]
(11, 13]
(13, 20]
(20, positive infinity)
```

For the last interval, the next-key lock locks the gap above the largest value in the index and the “supremum” pseudo-record having a value higher than any value actually in the index. The supremum is not a real index record, so, in effect, this next-key lock locks only the gap following the largest index value.

By default, `InnoDB` operates in [`REPEATABLE READ`](https://dev.mysql.com/doc/refman/8.0/en/innodb-transaction-isolation-levels.html#isolevel_repeatable-read) transaction isolation level. In this case, `InnoDB` uses next-key locks for searches and index scans, which prevents phantom rows (see [Section 15.7.4, “Phantom Rows”](https://dev.mysql.com/doc/refman/8.0/en/innodb-next-key-locking.html)).

