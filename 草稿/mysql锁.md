#### 一致性非锁定读
- 定义：INNODB存储引擎通过行多版本控制的方式读取当前执行时间数据库中行的数据；如果读取的行正在执行DELETE和UPDATE操作，这时读取操作不会等待行上的锁释放，而是去读取行的一个快照数据。  
- 读取快照数据、通过undo日志实现  
- 跟事务的隔离级别有关，并不是在每个事务隔离级别下都采用一致性非锁定读；即使都采用一致性非锁定读，对于快照数据的定义也不同  
- mvcc(多版本并发控制)：一行数据可能具有多个版本，对于多版本的数据的并发控制   
- 在事务隔离级别为READ COMMITTED 和 REPEATABLE READ下，INNODB存储引擎采用的是一致性非锁定读  

#### 一致性锁定读 
定义：用户对数据库读取操作进行加锁，用来保证数据逻辑的一致性  
INNODB存储引擎对于SELECT语句支持两种一致性锁定读的操作： 
- SELECT ... FOR UPDATE  
- SELECT ... LOCK IN SHARE MODE  
其中：SELECT ... FOR UPDATE对读取的行记录加一个X锁；SELECT ... LOCK IN SHARE MODE 对读取的行记录加一个S锁

#### MVCC 
- 是一种提供并发的技术。  
- 在只通过锁机制来做并发控制时，只有读读之间可以并发；读写、写读、写写都要阻塞；引入mvcc之后只有写写之间相互阻塞，其他三种操作都能并行。  
- MVCC只在 READ COMMITTED和REPEATABLE READ两个隔离级别下工作。其他两个隔离级别够和MVCC不兼容, 因为 READ UNCOMMITTED总是读取最新的数据行, 而不是符合当前事务版本的数据行。而 SERIALIZABLE 则会对所有读取的行都加锁。   
mvcc的实现方法     
一般有两种实现方式： 
- 写新数据时，把旧数据转移到一个单独的地方，如回滚段；其他人读数据时，从回滚段中把旧数据读出来；如oracle数据库和Mysql的InnoDB引擎。  
- 写新数据时，旧数据不删除，而是把新数据插入。postgreSQL就是使用的这种实现方式。  
事务的快照(read view)： 
通过undolog 和 事务系统实现 
关于快照（read view）的创建时机：  
- 在RR(可重复读)的隔离级别下，会在第一次执行select语句时创建read view  
- 在RC(读已提交)的隔离级别下，会在每次执行select语句时创建read view  

事务ID（Transaction ID）
InnoDB 里面每个事务有一个唯一的事务 ID，叫作 transaction id。它是在事务开始的时候向 InnoDB 的事务系统申请的，是按申请顺序严格递增的

参考文章：  
https://segmentfault.com/a/1190000012650596  
http://mysql.taobao.org/monthly/2015/04/01/  
http://mysql.taobao.org/monthly/2018/03/01/  
https://www.cnblogs.com/stevenczp/p/8018986.html  
http://mysql.taobao.org/ 阿里mysql内核月报  