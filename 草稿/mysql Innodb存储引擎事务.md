#### 事务类型
- 扁平事务  
所有操作都处于同一层次中，所有操作要么都成功，要么都失败；缺点在于：不能提交或者回滚事务的一部分  
- 带有保存点的扁平事务  
所有操作都处于同一层次中，在事务当中可以设置多个保存点，当事务需要回滚时，可以回滚到当前操作之前的任一保存点  
- 链事务  
多个事务以链式方式工作，提交事务和开启下一个事务操作合并为一个原子操作（下一个事务能看到上一个事务的提交结果）  
- 嵌套事务  
嵌套事务是一个层次结构，包含父事务和子事务   
- 分布式事务  

#### 事务的实现  
事务的四个特性：  
1、原子性  
2、一致性  
3、隔离性  
4、持久性  
- 事务的隔离性通过锁机制来实现  
- 事务的一致性通过undo log来实现  
- 事务的原子性和持久性通过redo log(重做日志)来实现  
redo log和undo log的作用都可以视为是一种恢复操作。

#### redo log(重做日志) 
* redo log用来保证事务的持久性
* 在mysql中，redo log包含两部分：重做日志缓冲(基于内存，易丢失),重做日志文件(持久)
* 可以通过参数innodb_flush_log_at_trx_commit来控制重做日志刷盘的策略  
  - 0：表示事务提交时不进行写入重做日志操作，而是由主线程完成；主线程中每1秒回进行一次重做日志文件的fsync操作，数据库崩溃后，会丢失未写入到重做日志的数据
  - 1：该参数的默认值，表示事务提交时必须调用一次fsync操作，数据库崩溃和操作系统崩溃都不会丢失数据
  - 2：表示事务提交时将重做日志写入重做日志文件，但仅写入文件系统的缓存中，不进行fsync操作，数据库宕机而操作系统不发生宕机时，不会导致事务的丢失。

#### redo log(重做日志)和binlog(二进制日志)的区别  
binlog的作用：用来进行PIT(POINT-IN-TIME)的恢复及主从复制(Replication)环境的建立。
区别：
- 重做日志是在InnoDB存储引擎层产生的，而二进制日志是在Mysql数据库的上层产生的，二进制日志不仅仅针对于InnoDB存储引擎，Mysql任何数据库中的任何存储引擎对于数据库的更改都会产生二进制日志  
- 两种日志记录的内容形式不同，二进制日志是一种逻辑日志，记录的是对应的SQL语句。重做日志是物理格式日志，记录的是对于每个页的修改  
- 两种日志记录写入磁盘的时间点不同，二进制日志只在事务提交完成后进行一次写入。重做日志在事务进行中不断的被写入  

#### undo log
作用：用于事务回滚和mvcc  
存储：
InnoDB存储引擎对于undo log的存储采用segment(段)来存储，
InnoDB存储引擎有rollback segment(回滚段)，每个回滚段中记录了1024个undo log segment(undo段)，在每个undo log段中申请undo页 



  
