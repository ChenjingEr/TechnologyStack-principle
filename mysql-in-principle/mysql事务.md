## mysql 事务:

##### 事务特性(ACID)：
A: 原子性(redo log实现) <br>
C: 一致性，事务完成后数据库的完整性与一致性都会改变(undo log实现)  <br>
I: 隔离性,多个事务同时操作不相互影响(基于锁的实现)  <br> 
D: 持久性，事务一旦提交，结果就是持久的(redo log 实现)   <br>

##### 事务分类
1. 扁平事务：事务由 begin 开始， commit || rollback 结束。其中rollback只能回滚到事务开始处
2. 带来检查点的扁平事务：事务操作过程中可以save CheckPoint。rollback时可以回滚到任一设置的check point或者事务起始处。但那时这个check point是易失的
3. 事务链:一个事务结束后，是否不需要了的数据对象，将必要的事务处理上下文隐式传给下一个事务。
4. 嵌套事务
5. 分布式事务

##### mysql 事务实现

<font color="#CD6155">隔离性实现: 锁 </font><br>
<font color="#CD6155">持久性实现 & 原子性实现：redo log。 </font>

    redo log 恢复事务提交修改的页。是物理日志，记录的是页的物理修改操作。 
    redo log 包括内存中的重做日志缓存以及重做日志文件( 
    os写入文件的逻辑是在内存与file之间添加一个缓存，然后定期将缓存中的内容刷到文件中，以提高性能)
    redo log缓存的刷新是通过一次 fsync 操作来完成的。但是何时进行 fsync 操作？
        1. commit 一次， fsync 一次。 磁盘性能不足够优的话，导致事务操作性能差
        2. 手工设置非持久性，等待一个周期才将缓存刷入文件。 在未刷新之前宕机，这有可能会丢失部分redo log丢失的情况。
        用innodb_flush_log_at_trx_commit控制redo log 的刷新策略
            = 0:事务 commit时不进行fsync，由master thread来完成。master thread每秒1次的频率执行fsync
            = 1:sync at commit
            = 2:commit仅写入缓存，不fsync。这时候数据库宕机只要os没有宕机，就不会影响事务的redo

<font color="#CD6155">一致性实现：undo log </font>
    
    undo log是保存在数据库内部的一个undo segment中。
    undo是逻辑日志，只是将数据库逻辑地恢复到原来的样子。数据结构和页本身在回滚之后可能是不同的。
    undo log 会产生redo log
    undo log空间可以重用。所以一个undo log可能记录的不仅仅是一个事务的数据。
    undo log中保存的是insert undo log & update undo log。
    insert undo log commit之后可以直接删除，因为插入操作对于其他的事务没有影响
    update undo log commit之后并不是直接删除，而是将delete flag标记为删除。真正的删除操作由purge来完成。
    原因是MySQL的mvcc,也就是在删除这条操作执行的过程中，可能其他事务有在引用这个undo记下的版本的记录。purge确认
    没有其他事务引用才会删除。

##### 事务控制语句
mysql命令行默认情况下事务都是自动提交的。需要显示开启命令用begin , start transaction 或者 set auto_commit = 0. <br>
事务控制语句总结:
    
    begin | start transaction：显示开启一个事务
    commit : 事务提交
    rollback : 事务回滚
    savepoint identifier:设置保存点，一个事务中identifier递增
    release savepoint identifier:释放保存点
    rollbakc to identifier:回滚到保存点
    set transaction:设置隔离级别
隐式事务提交SQL：
    
    DDL操作
    用来隐式地修改Mysql架构的操作 create user, drop user, grant 等。。。
    管理语句

##### 事务操作统计 QPS(Question Per Second, QPS) & TPS(Transaction Per Second TPS)
    
    mysql 用 com_commit / com_rollback记录事务提交与回滚(显示提交与回滚操作)
    TPS = (com_commit + com_rollback) / time

##### 事务隔离级别
    
    脏页：缓存更改但是还没有刷新到文件的页，这种数据是正常的，读取的是缓存中的内容，且最终会保持一致。
    脏数据：一个事务读到了另一个事务还没有提交的数据，这种数据不正常，因为事务可能回滚
    幻读：重复读取的数据不一样

1. READ UNCOMMITTED：未提交读，事务可以读取其他没有提交的事务的数据
2. READ COMMITTED：提交读，事务只能读取其他事务提交了的数据
3. REPEATABLE READ：可重复读，同一个事物多次读不会出现不一样的结果(幻读)。mysql默认的隔离级别，用next-key-lock实现
4. SERIALIZABLE：序列化

##### 分布式事务
MySQL中用xa支持分布式事务。
在MySQL中，分布式事务存在不同的机器节点之间，也存在数据库服务器与存储引擎之间(binlog 与 redo log)。 <br>
mysql xa 用两阶段提交协议。现有一个prepare阶段，再是commit <br>
mysql支持分布式事务隔离级别必须设置成serializable <br>

