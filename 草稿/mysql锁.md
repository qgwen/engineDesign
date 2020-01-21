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

#### 自增长与锁
自增长计数器    
SELECT MAX(auto_inc_col) FROM t FOR UPDATE  
AUTO-INC Locking,特殊的表锁机制，为了提高插入的性能，锁不是在一个事务完成后才释放，而是在完成对自增长值插入的SQL语句后立即释放，但还是存在一些性能上的问题：  
- 对有自增长值得列的并发插入性能较差，事务必须等待前一个插入的完成  
- 对于INSERT...SELECT的大数据量的插入会影响插入性能，因为另一个事务的插入会被阻塞  
Mysql5.1.22开始，InnoDB存储引擎提供了一种轻量级互斥量的自增长实现机制。  
#### 外键和锁 
对于外键值得插入或更新，需要使用一致性锁定读（SELECT ... LOCK IN SHARE MODE）查询父表中的记录,即主动对父表加S锁。如果此时父表已经加了X锁，子表的操作阻塞。

#### 行锁的三种算法  
InnoDB存储引擎行锁包含三种算法:  
- Record Lock(记录锁，单个行记录上的锁)：锁定的是一个记录上的索引，而不是记录本身    
- Gap Lock(间隙锁，锁定的是一个范围，锁定索引之间的间隙，但是不包含索引本身)  
- Next-Key Lock是Record Lock + Gap Lock的组合  

在默认的数据库隔离级别下（Read Repeated,可重复读），InnoDB存储引擎是采用Next-Key Lock进行行锁的添加；当查询的条件含有唯一索引时，InnoDB存储引擎会将Next-Key Lock进行优化，降级为Record Lock。  
在Read Commited(读已提交)下，InnoDB存储引擎是采用Record Lock

间隙锁(Gap Lock)及添加规则：
间隙锁锁定的是一个范围，但不包含索引本身，如(3,7):左右两边都是开区间  

Note:  
1、InnoDB存储引擎中，行级锁实际上是索引记录锁，在索引记录上设置共享或互斥锁。  


TODO:  
1、谓词锁(predict lock)  ：锁定符合查询条件的所有记录，包含待插入、待更新、待删除的
2、previous-key locking  
#### Phantom Problem（幻读问题）
Phantom Problem是指在同一个事务下，连续执行两次同样的SQL语句可能导致不同的结果，第二次的SQL语句可能会返回之前不存在的行
#### 锁问题
- 脏读，脏读指的是读取在缓冲区中未提交的数据，需要和脏页区分开，脏页指的是缓冲区的页还没有刷新到磁盘中  
- 不可重复读，再一个事务里面两次查询获取的结果不同，Mysql官方文档中讲不可重复读的问题定义为Phantom Problem（幻像问题）
- 丢失更新，一个事务的更新操作会被另一个事务的更新操作覆盖，通常，物理上不会出现这种情况，在业务逻辑上可能会出现这种情况  
#### 阻塞
在默认情况下，InnoDB存储引擎不会回滚超时引发的错误异常。其实，InnoDB存储引擎在大部分情况下都不会对异常进行回滚。这个时候需要用户自行决定是回滚还是提交（这也就能解释，为什么在spring中开启事务时，需要设置rollbackFor属性）
但有一种情况特殊，就是死锁，发生死锁后，会马上回滚一个事务   

#### 死锁
死锁问题：
两个或两个以上的事务在执行过程中，因争夺锁资源而造成的一种互相等待的现象。  
解决死锁问题的方案：  
1、锁等待超时后回滚（超时机制）
这种方案的特点是比较简单，缺点是比较粗暴，假设两个事务造成死锁，事务A所占比重比较大，事务B所占比重比较小，这个时候如果回滚事务A就不合适。
2、wait-for graph(等待图)
wait-for graph是一种更为主动的死锁检测机制，InnoDB存储引擎就是使用这种方式进行死锁检测。
InnoDB根据以下两种信息来构建图：
- 锁的信息链表  
- 事务等待链表  
如果该图中存在回路，就代表存在死锁;在wait-for graph中，事务为图中的节点。而在图中，事务T1指向T2边的定义为：  
1、事务T1等待事务T2所占用的资源  
2、事务T1最终等待T2锁占用的资源，也就是事务之间在等待相同的资源，而事务T1发生在事务T2的后面  
若存在死锁，InnoDB存储引擎选择回滚undo量最小的事务。 
wait-for graph的死锁检测采用深度优先遍历算法实现

#### innodb存储引擎加锁机制
根据页进行加锁，采用位图方式

  


