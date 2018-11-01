## mysql锁

##### innodb锁类型
    
行级锁：

    1. 共享锁(S):允许事务读取一行数据
    2. 排他锁(X):允许事务删除或者更新一行数据

意向锁：允许事务行级锁与表级锁同时存在

    1. 意向共享锁(IS):事务想要获得一张表中某几行的共享锁
    2. 意向排他锁(IX)：事务想要获得一张表中某几行的排他锁

在sql执行过程中，innodb锁信息记录在infomation_schema.innodb_trx, information_schema.innodb_locks, information_schema.innodb_lock_wairs这几个表中。

##### 读锁

一致性非锁定读（多版本并发控制 Multi Version Concurrentcy Control ,MVCC）：innodb通过行多版本控制(multi versioning) 的方式读取当前执行时间数据库中的行的数据。这是为了提高数据库的并发性涉设计的。当一个读请求进来，正有其他的事物正在执行delete || update的时候，这时该读事物并不会被锁定，会读取当前时间之前的一个快照版本中的数据（通过undo段实现）。当时不同的事物隔离级别读取的快照版本是不同的。 <br>
一致性非锁定读是在隔离级别为 REPEATABLE READ(读取事物开始时的快照) || READ COMMITTED(读取最新时间的快照)级别下 <br>

一致性锁定读：需要保证逻辑一致性的情况下

##### innodb实现锁算法

1. Record Lock:单个记录上的锁
2. Gap Lock:间隙锁，锁定一个范围，但是不包含记录本身
3. Next-Key Lock:Gap Lock + Record Lock,解决Phantom Problem

对于唯一键的锁定，会将Next-Key Lock 降级为Record Lock提高并发性

Phantom Problem 是指在同一事务下，连续执行两次同样的SQL语句可能导致不同的结果，第二次的SQL语句可能会返回之前不存在的行。